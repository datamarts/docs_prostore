---
layout: default
title: Маршрутизация запросов к данным
nav_order: 1
parent: Запрос данных
grand_parent: Работа с системой
has_children: false
has_toc: false
---

# Маршрутизация запросов к данным {#routing}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запросы на [чтение](../../data_reading/data_reading.md) и [выгрузку данных](../../data_download/data_download.md) 
маршрутизируются следующим образом:
1.  Если в запросе указано ключевое слово [DATASOURCE_TYPE](../../../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type) 
    с [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
    [хранилища](../../../overview/main_concepts/data_storage/data_storage.md)
    для исполнения запроса, запрос направляется в указанную СУБД.
2.  Иначе:
    1. Определяются те СУБД хранилища, в которых можно выполнить запрос, — выбираются СУБД, содержащие 
       данные всех запрашиваемых логических сущностей.
    2. Определяется категория и подкатегория запроса (или только категория — в зависимости от
       [конфигурации системы](../../../maintenance/configuration/system/system.md)).   
    3. Запрос направляется в ту из выбранных СУБД, которая имеет в конфигурации системы 
       самый высокий приоритет для исполнения таких запросов (см. [ниже](#standard_priorities)).
3. Если запрос обращается к [материализованному представлению](../../../overview/main_concepts/materialized_view/materialized_view.md), 
   он проходит [дополнительные этапы маршрутизации](#mat_view_routing).

## Определение категории и подкатегории запроса {#categories}

В зависимости от конфигурации системы система выбирает СУБД хранилища для исполнения запроса, учитывая категорию и 
подкатегорию запроса или только категорию запроса:
* если в конфигурации системы задана секция параметров `plugins.category.autoSelect`, система использует настройки 
  этой секции и при маршрутизации запроса учитывает как категорию, так и подкатегорию запроса;
* иначе, если секция `plugins.category.autoSelect` не задана, система использует настройки секции 
  `plugins.category.mapping` и учитывает только категорию запроса, без учета его подкатегории.

Категории запросов:
1. Реляционный запрос (Relational) — запрос с ключевым словом JOIN и (или) подзапросами.
2. Аналитический запрос (Analytical) — запрос с ключевым словом GROUP BY и агрегатными функциями.
3. Запрос чтения по ключу (Dictionary) — запрос с условием WHERE на значения первичного ключа.
4. Другой запрос (Undefined) — запрос, который не соответствует ни одной из предыдущих категорий.

Подкатегории запросов:
* один узел (ShardOne) — запрос предназначен для одного узла кластера;
* набор узлов (ShardSet) — запрос предназначен для определенных узлов кластера (от 1 до всех);
* все узлы (ShardAll) — запрос требует исполнения на всех узлах кластера.

Система определяет категорию запроса следующим образом: проверяет запрос на соответствие первой категории,
и, если у него есть признаки этой категории, то система относит запрос к первой категории, иначе система проверяет запрос 
на соответствие второй категории и т.д. Например, запрос с ключевым словом JOIN соответствует первой категории
(«Реляционный запрос») независимо от наличия признаков других категорий — 
агрегации, группировки и чтения по ключу. Подкатегория запроса выбирается похожим образом с той разницей, что в этом 
случае порядок проверки неважен. Примеры запросов по категориям и подкатегориям и см. [ниже](#requests_by_categories).

Если при маршрутизации запросов нужно учитывать только категорию запросов, без учета подкатегории, скорректируйте 
конфигурацию системы: удалите секцию параметров `plugins.category.autoSelect` и настройте секцию 
`plugins.category.mapping`.
{: .note-wrapper}

## Стандартный порядок выбора СУБД {#standard_priorities}

По умолчанию для запросов чтения и выгрузки данных действует порядок выбора СУБД хранилища, описанный в таблице ниже. 
Например, реляционный запрос, предназначенный для одного узла кластера, будет по возможности исполнен в ADB, иначе, 
если в ADB нет нужных таблиц, — в ADP, и далее в указанном порядке. Такой порядок эффективно 
использует возможности каждой из СУБД хранилища, однако при необходимости его можно изменить.

<a id="tab_default_routing"></a>

| Категория запроса | Порядок выбора СУБД для исполнения запроса 
|:-|:-
| 1. Реляционный запрос | Один узел: 1. ADB, 2. ADP, 3. ADQM, 4. ADG<br>Набор узлов: 1. ADB, 2. ADP, 3. ADQM, 4. ADG<br>Все узлы: 1. ADB, 2. ADP, 3. ADQM, 4. ADG
| 2. Аналитический запрос | Один узел: 1. ADQM, 2. ADB, 3. ADP, 4. ADG<br>Набор узлов: 1. ADQM, 2. ADB, 3. ADP, 4. ADG<br>Все узлы: 1. ADQM, 2. ADB, 3. ADP, 4. ADG
| 3. Запрос чтения по ключу | Один узел: 1. ADG, 2. ADB, 3. ADP, 4. ADQM<br>Набор узлов: 1. ADG, 2. ADB, 3. ADP, 4. ADQM<br>Все узлы: 1. ADG, 2. ADB, 3. ADP, 4. ADQM
| 4. Другой запрос | Один узел: 1. ADB, 2. ADP, 3. ADQM, 4. ADG<br>Набор узлов: 1. ADB, 2. ADP, 3. ADQM, 4. ADG<br>Все узлы: 1. ADB, 2. ADP, 3. ADQM, 4. ADG

Наиболее полный синтаксис запросов доступен в ADB и ADP. ADG и ADQM имеют ограничения 
на выполнение запросов, вызванные особенностями этих СУБД (см. [Поддержка SQL](../../../reference/sql_support/sql_support.md)).
{: .note-wrapper}

## Примеры запросов по категориям {#requests_by_categories}

### Реляционные запросы {#relation}

Реляционный запрос:
```sql
SELECT * FROM sales.sales AS s
JOIN sales.stores AS st ON s.store_id = st.id
```

Реляционный запрос, который включает агрегацию, группировку и чтение по ключу (`st.id`):
```sql
SELECT st.id, st.category, SUM(s.product_units) AS product_amount
FROM sales.stores AS st
JOIN sales.sales AS s ON st.id = s.store_id
WHERE st.id <> 10004
GROUP BY st.id, st.category
ORDER BY product_amount DESC
```

### Запросы агрегации и группировки {#aggregation}

Запрос агрегации и группировки:
```sql
SELECT s.product_code, SUM(s.product_units) AS product_amount
FROM sales.sales AS s
GROUP BY s.product_code
ORDER BY product_amount ASC
```

Запрос агрегации и группировки, который включает чтение по ключу (`s.id`):
```sql
SELECT s.product_code, SUM(s.product_units) AS product_amount
FROM sales.sales AS s
WHERE s.id > 20000
GROUP BY s.product_code
```

### Запрос чтения по ключу {#read_by_key}

```sql
SELECT * FROM sales.sales as s
WHERE s.id BETWEEN 1001 AND 2000
```

### Запрос неопределенной категории {#undefined}

```sql
SELECT * FROM sales.sales AS s
WHERE s.product_units > 2  
```

## Примеры запросов по подкатегориям {#requests_by_subcategories}

### Запросы для одного узла {#shard_one}

Запрос к логическим таблицам `transactions1` (шардирование по `col1`) и `accounts1` (шардирование по `colA`):
```sql
SELECT * FROM transactions1 t
JOIN accounts1 a ON a.colA = t.col1
WHERE t.col1 = 1
```

Запрос к логической таблице `transactions2` (шардирование `col1` и `col2`):
```sql
SELECT * FROM transactions2 t
WHERE t.col1 = 1 AND t.col2 = 2 AND amount > 0  
```

Запросы к логическим таблицам `transactions2` (шардирование по `col1` и `col2`) и `accounts2` (шардирование по `colA` и `colB`):
```sql
-- столбцы col1 and colA имеют один тип данных; столбцы col2 and colB также имеют один тип данных

-- запрос объединения таблиц
SELECT * FROM transactions2 t
JOIN accounts a ON a.id = t.account_id
WHERE t.col1 = 1 AND t.col2 = 2 AND a.colA = 1 AND t.colB = 2;

-- запрос с подзапросом
SELECT * FROM transactions2 t
WHERE t.account_id IN (
  SELECT id
  FROM accounts2 a
  WHERE t.col1 = 1 AND t.col2 = 2 AND a.colA = 1 AND a.colB = 2;
```

### Запросы для набора узлов {#shard_set}

Запрос к логическим таблицам `transactions1` (шардирование по `col1`) и `accounts1` (шардирование по `colA`):
```sql
SELECT * FROM transactions1
WHERE col1 IN (
  SELECT id
  FROM accounts1
  WHERE colA = 1
)
```

Запрос к логической таблице `transactions2` (шардирование по `col1` и `col2`):
```sql
SELECT * FROM transactions2 t
WHERE (t.col1 = 1 AND t.col2 = 2) OR ((t.col1 = 1 AND t.col2 = 1) AND amount > 0)
```

Запросы к логическим таблицам `transactions2` (шардирование по `col1` и `col2`) и `accounts2` (шардирование по `colA` и `colB`):
```sql
-- столбцы col1 and colA имеют РАЗНЫЕ типы данных; столбцы col2 and colB имеют один тип данных
SELECT * FROM transactions2 t
JOIN accounts2 a ON a.id = t.account_id
WHERE t.col1 = 1 AND t.col2 = 2 AND a.colA = 1 AND t.colB = 2
```

### Запросы для всех узлов {#all_shards}

Запрос к логическим таблицам `transactions1` (шардирование по `col1`) и `accounts1` (шардирование по `colA`):
```sql
SELECT * FROM transactions1 t 
JOIN accounts1 a ON a.id = t.account_id
WHERE t.col1 = 1 OR a.colA = 1
```

Запрос к логической таблице `transactions2` (шардирование по `col1` и `col2`):
```sql
SELECT * FROM transactions2 t
WHERE (t.col1 = 1 AND t.col2 = 2) OR amount > 0 
```

Запросы к логическим таблицам `transactions2` (шардирование по `col1` и `col2`) и `accounts2` (шардирование по `colA` и `colB`):
```sql
-- столбцы col1 and colA имеют один тип данных; столбцы col2 and colB также имеют один тип данных
SELECT * FROM transactions2 t
JOIN accounts2 a ON a.id = t.account_id
WHERE (t.col1 = 1 AND t.col2 = 2) OR (a.colA = 1 AND t.colB = 2)
```

## Маршрутизация запросов к данным материализованных представлений {#mat_view_routing}

Запросы к данным [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md) 
проходят все этапы маршрутизации, описанные [выше](#top), и затем — дополнительные этапы:
1. Если для материализованного представления **не** указано ключевое слово 
   [FOR SYSTEM_TIME](../../../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time), 
   запрос направляется в СУБД, где размещены данные этого представления. Из представления выбираются данные, 
   актуальные на момент его последней синхронизации.
2. Иначе, если ключевое слово `FOR SYSTEM_TIME` указано, система проверяет, присутствуют ли запрашиваемые данные 
   в материализованном представлении.
   * Если данные присутствуют в представлении, запрос направляется в СУБД, где размещены данные этого представления.
   * Иначе запрос направляется к исходным таблицам СУБД-источника, на которых построено представление.
    
В запросах к материализованным представлениям доступны не все сочетания с ключевым словом `FOR SYSTEM_TIME`. 
Подробнее см. в секции 
[Ключевое слово FOR SYSTEM_TIME](../../../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time_matview) 
раздела [SELECT](../../../reference/sql_plus_requests/SELECT/SELECT.md).
{: .note-wrapper}