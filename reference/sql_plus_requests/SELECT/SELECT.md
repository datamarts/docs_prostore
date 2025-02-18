---
layout: default
title: SELECT
nav_order: 42
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# SELECT
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет выбрать данные из следующих сущностей и их соединений:
* [логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md),
* [логических представлений](../../../overview/main_concepts/logical_view/logical_view.md),
* [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md),
* [standalone-таблиц](../../../overview/main_concepts/standalone_table/standalone_table.md).

Синтаксис чтения из standalone-таблицы подразумевает использование
[внешней readable-таблицы](../../../overview/main_concepts/external_table/external_table.md#readable_table), которая
указывает на нужную standalone-таблицу.

Запросы к логическим таблицам и представлениям по умолчанию возвращают данные, актуальные на текущий момент. 
Чтобы выбрать срез данных на определенный момент времени, укажите ключевое слово [FOR SYSTEM_TIME](#for_system_time) 
с нужным значением. Для standalone-таблиц ключевое слово недоступно: запросы к таким таблицам всегда 
возвращают текущее состояние данных.

Запрос также можно использовать для 
[получения информации о самом запросе](../../../working_with_system/other_features/query_estimation/query_estimation.md), 
а также как подзапрос в других запросах:
* на [выгрузку данных](../INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md),
* на [создание](../CREATE_VIEW/CREATE_VIEW.md) или [обновление](../ALTER_VIEW/ALTER_VIEW.md) логического представления,
* на [создание материализованного представления](../CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md),
* на [вставку данных из другой сущности](../INSERT_SELECT/INSERT_SELECT.md).

Запрос `SELECT` обрабатывается в порядке, описанном в разделе [Порядок обработки запросов на чтение данных](../../../overview/interactions/llr_processing/llr_processing.md).
В ответе возвращается:
* объект ResultSet c выбранными записями или информацией о запросе при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

## Синтаксис {#syntax}

```sql
SELECT column_list
FROM [db_name.]entity_name
[FOR SYSTEM_TIME time_expression [AS alias_name]]
[DATASOURCE_TYPE = datasource_alias]
[ESTIMATE_ONLY]
```

**Параметры:**

`column_list`

: Список выбираемых столбцов таблицы или представления. Можно указывать символ `*`
  для выбора всех столбцов.

`db_name`

: Имя логической базы данных, из которой выбираются данные. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`entity_name`

: Имя таблицы или представления, из которого выбираются данные.

`time_expression`

: Выражение, задающее момент или период времени, за который выбираются данные или изменения данных. 
  Синтаксис см. [ниже](#for_system_time).

`alias_name`

: Псевдоним таблицы или представления. Может включать латинские буквы, цифры и символы
  подчеркивания.

`datasource_alias`

: Системный псевдоним СУБД хранилища, из которой выбираются данные. Возможные значения: 
  `'adb'`, `'adqm'`, `'adg'`, `'adp'`.

В запросах можно указывать: 
* ключевые слова, перечисленные в секции [Поддерживаемые ключевые слова](#key_words), 
* функции, перечисленные в разделе [Поддержка SQL](../../sql_support/sql_support.md),
* псевдонимы для имен таблиц, представлений и столбцов.

Некоторые агрегатные функции и типы соединений недоступны для исполнения в определенных
СУБД хранилища. Список доступных возможностей см. в разделе [Поддержка SQL](../../sql_support/sql_support.md).
{: .note-wrapper}

### Поддерживаемые ключевые слова {#key_words}
    
В запросе можно использовать следующие ключевые слова, которые должны быть указаны в порядке их перечисления:
1. `FOR SYSTEM_TIME` — задает момент времени или период, за который выбираются данные или 
   изменения данных. Поддерживается для логических таблиц, логических и материализованных представлений. 
   Описание и синтаксис см. в секции [Ключевое слово FOR SYSTEM_TIME](#for_system_time);
2. `JOIN ON` — соединяет данные нескольких таблиц и (или) представлений из одной или
   нескольких [логических БД](../../../overview/main_concepts/logical_db/logical_db.md). 
   Возможные значения см. в секции [Возможные типы соединений](#join_prefixes);
3. `WHERE` — задает условия выбора данных. Условия в запросах к ADG могут включать ключевое слово 
   [COLLATE](#collate);
4. `GROUP BY` — группирует данные;
5. `HAVING` — задает условия выбора сгруппированных данных;
6. `ORDER BY` — сортирует данные;
7. `LIMIT` или `FETCH NEXT <N> ROWS ONLY`— ограничивает количество возвращаемых строк;
8. `OFFSET` — пропускает указанное количество строк в результате запроса. Описание и синтаксис см. в 
    секции [Ключевое слово OFFSET](#offset);
9. <a id="param_datasource_type"></a>`DATASOURCE_TYPE` — задает [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
   [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), из которой выбираются данные. 
   Поддерживается для логических таблиц, логических и материализованных представлений;
10. `ESTIMATE_ONLY` — включает режим получения информации о запросе, а не самих данных. Описание см. в секции 
    [Ключевое слово ESTIMATE_ONLY](#estimate).
     
### Ключевое слово FOR SYSTEM_TIME {#for_system_time}

Ключевое слово `FOR SYSTEM_TIME` задает момент, по состоянию на который запрашиваются данные,
или период (диапазон дельт), за который запрашиваются изменения. Возможные значения см. в таблице 
[ниже](#tab_for_system_time_values).

Ключевое слово относится к логической таблице,
логическому представлению или материализованному представлению, после имени которого оно следует. Если в запросе
соединяется несколько логических таблиц и представлений, для каждой логической сущности можно указать свое ключевое слово
`FOR SYSTEM_TIME`, при этом значения этих ключевых слов могут различаться (см. пример [ниже](#deltas_example)).

Ключевое слово недоступно в запросах к standalone-таблицам.
{: .note-wrapper}

Если ключевое слово не указано, из логических таблиц и логических представлений
выбираются данные, актуальные на момент обработки запроса, из материализованного представления — данные,
актуальные на момент последней синхронизации этого представления.

В таблице ниже описаны возможные значения выражения `FOR SYSTEM_TIME time_expression`.

<a id="tab_for_system_time_values"></a>

| Выражение | Описание | Доступно для материализованных представлений
|:-|:-|:-
| `FOR SYSTEM_TIME AS OF 'YYYY-MM-DD hh:mm:ss'` | Запрос данных, актуальных на указанную дату и время. Возможные форматы даты и времени см. в разделе [Форматы даты и времени в запросах](../../timestamp_formats/timestamp_formats.md) | +
| `FOR SYSTEM_TIME AS OF DELTA_NUM delta_num` | Запрос данных, актуальных на дату и время закрытия дельты с номером `delta_num` | +
| `FOR SYSTEM_TIME AS OF LATEST_UNCOMMITTED_DELTA` | Запрос данных на текущий момент, включая данные, загруженные в горячей дельте. <br>По горячей дельте возвращаются записи, загруженные в рамках непрерывного диапазона завершенных операций записи (см. параметры `cn_from` и `cn_to`в разделе [GET_DELTA_HOT](../GET_DELTA_HOT/GET_DELTA_HOT.md)) | –
| `FOR SYSTEM_TIME STARTED IN (delta_num1, delta_num2)` | Запрос данных, добавленных или измененных в период между дельтой `delta_num1` и дельтой `delta_num2` (включая граничные дельты) | +<br>Если обе дельты есть в представлении
| `FOR SYSTEM_TIME FINISHED IN (delta_num1, delta_num2)` | Запрос данных, удаленных в период между дельтой `delta_num1` и дельтой `delta_num2` (включая граничные дельты) | +<br>Если обе дельты есть в представлении

Наличие и значение ключевого слова `FOR SYSTEM_TIME` в запросах к материализованным представлениям
влияют на [порядок маршрутизации запросов](../../../working_with_system/data_reading/routing/routing.md#mat_view_routing)).
{: .note-wrapper}

### Поддерживаемые типы соединений {#join_prefixes}

Поддерживаются следующие типы соединений:
*   `[INNER] JOIN` — внутреннее соединение,
*   `NATURAL JOIN` — внутреннее соединение по всем столбцам с одинаковыми именами, ключи соединения 
    не указываются,
*   `LEFT [OUTER] JOIN` — левое внешнее соединение,
*   `RIGHT [OUTER] JOIN` — правое внешнее соединение,
*   `FULL [OUTER] JOIN` — полное внешнее соединение,
*   `CROSS JOIN` — декартово произведение таблиц или представлений, ключи соединения не указываются.

### Ключевое слово COLLATE {#collate}

Ключевое слово `COLLATE` позволяет задать правило сопоставления символьных строк, например, 
приравнять строки в верхнем и нижнем регистрах. Ключевое слово доступно в условии, заданном с помощью 
ключевого слова `WHERE` (см. пример [ниже](#collate_example)).

Ключевое слово `COLLATE` поддерживается только для запросов к ADG. Это означает, что для его корректной работы 
в запросе нужно либо указать ADG в качестве источника данных (`DATASOURCE_TYPE = 'adg'`), либо обращаться к 
логической сущности, данные которой размещены только в ADG.

Подробнее о правилах сопоставления символьных строк в ADG см. в 
[документации Tarantool](https://www.tarantool.io/ru/doc/latest/reference/reference_sql/sql_statements_and_clauses/#sql-collate-clause).

### Ключевое слово OFFSET {#offset}

Ключевое слово `OFFSET` позволяет пропустить первые несколько строк результата и выбрать только последующие строки (см. 
примеры [ниже](#offset_example)). 
В качестве значения ключевого слова можно указать любое неотрицательное целое число, начиная с нуля, или 
переменную. Если для `OFFSET` указано значение 0, то пропускается 0 строк, что равносильно запросу 
без `OFFSET`. 

Запросы с `OFFSET` без ограничения количества строк не поддерживаются. То есть, если ключевое слово 
`OFFSET` указано в запросе, то перед ним должно быть ключевое слово `LIMIT <N>` или `FETCH NEXT <N> ROWS ONLY`. 
Обратного ограничения нет: ключевые слова `LIMIT <N>` и `FETCH NEXT <N> ROWS ONLY` можно использовать без `OFFSET`.

Рекомендуется сочетать `OFFSET` с ключевым словом `ORDER BY` для получения упорядоченного набора строк. Ключевое слово 
`ORDER BY` необязательно, однако без него запрос с `OFFSET` возвращает неупорядоченный и потому 
непредсказуемый набор строк.

Таким образом, для ключевого слова `OFFSET` поддерживается следующий синтаксис:
```sql
[ ORDER BY <column_name> ]
{ LIMIT <value_1> | FETCH NEXT <value_1> ROWS ONLY } 
OFFSET <value_2> [ ROW | ROWS ]
```

### Ключевое слово ESTIMATE_ONLY {#estimate}

Ключевое слово `ESTIMATE_ONLY` позволяет запросить информацию о выполнении запроса к данным, а не сами данные (см. 
пример [ниже](#estimate_example)). 

Если ключевое слово указано, запрос возвращает следующую информацию: 
* имя СУБД хранилища, в которой предполагается выполнение запроса;
* план выполнения запроса (для ADB и ADP) — результат выполнения команды EXPLAIN в СУБД хранилища. 
  Подробнее о команде EXPLAIN в ADB см. в [документации Greenplum](https://gpdb.docs.pivotal.io/6-17/ref_guide/sql_commands/EXPLAIN.html),
  о команде в ADP — в [документации PostgreSQL](https://www.postgresql.org/docs/13/using-explain.html);
* обогащенный запрос — запрос, подготовленный системой на основе исходного запроса с учетом специфики СУБД хранилища.

В ответе возвращается объект ResultSet с одной строкой, содержащей JSON-строку в следующем формате:
```json
{
  "plugin": "<имя_СУБД>",
  "estimation": <план_выполнения_запроса>,
  "query": <обогащенный_запрос>
}
```

Ниже показан пример JSON-строки, полученной из ADB. Для наглядности пример представлен в виде дерева, 
а не плоской строки.

```json
{
  "plugin": "ADB",
  "estimation": [
    {
      "Plan": {
        "Node Type": "Gather Motion",
        "Senders": 4,
        "Receivers": 1,
        "Slice": 1,
        "Segments": 4,
        "Gang Type": "primary reader",
        "Startup Cost": 0.00,
        "Total Cost": 433.70,
        "Plan Rows": 50000,
        "Plan Width": 8,
        "Plans": [
          {
            "Node Type": "Seq Scan",
            "Parent Relationship": "Outer",
            "Slice": 1,
            "Segments": 4,
            "Gang Type": "primary reader",
            "Relation Name": "sales_actual",
            "Alias": "sales_actual",
            "Startup Cost": 0.00,
            "Total Cost": 432.18,
            "Plan Rows": 50000,
            "Plan Width": 8
          }
        ]
      },
      "Settings": {
        "Optimizer": "Pivotal Optimizer (GPORCA)"
      }
    }
  ],
  "query": "SELECT * FROM (SELECT id FROM marketing.sales_actual WHERE sys_from <= 98 AND COALESCE(sys_to, 9223372036854775807) >= 98)"
}

```

Ниже показан пример JSON-строки, полученной из ADP. Для наглядности пример представлен в виде дерева,
а не плоской строки.
```json
{
  "plugin": "ADP",
  "estimation": [
    {
      "Plan": {
        "Node Type": "Seq Scan",
        "Parallel Aware": false,
        "Relation Name": "sales_actual",
        "Alias": "sales_actual",
        "Startup Cost": 0.00,
        "Total Cost": 18.80,
        "Plan Rows": 880,
        "Plan Width": 64
      }
    }
  ],
  "query": "SELECT * FROM (SELECT id FROM marketing.sales_actual WHERE sys_from <= 98 AND COALESCE(sys_to, 9223372036854775807) >= 98)"
}
```

## Ограничения {#restrictions}

*   Запрос может обращаться либо к логической БД, либо к сервисной БД (см. [SELECT FROM INFORMATION_SCHEMA](../SELECT_FROM_INFORMATION_SCHEMA/SELECT_FROM_INFORMATION_SCHEMA.md)), 
    но не к обеим одновременно.
*   Если ключами соединения в запросе выступают поля типа Nullable, то строки, где хотя бы один из ключей 
    имеет значение NULL, не соединяются.
*   Ключевое слово `ORDER BY` не поддерживается для SELECT-подзапроса в составе запроса 
    [CREATE MATERIALIZED VIEW](../CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md). 

## Примеры {#examples}

### Звездочка и WHERE {#asterisk_and_where_example}

Запрос с неявным указанием столбцов и ключевым словом `WHERE`:
```sql
SELECT * FROM marketing.sales
WHERE store_id = 1234
```

### DATASOURCE_TYPE {#datasource_type_example}

Запрос с перечислением столбцов и выбором данных из определенной СУБД хранилища (ADQM):
```sql
SELECT sold.store_id, sold.product_amount
FROM marketing.stores_by_sold_products AS sold
DATASOURCE_TYPE = 'adqm'
```

### GROUP BY, ORDER BY и LIMIT {#group_by_example}

Запрос с агрегацией, группировкой и сортировкой данных, а также выбором первых 20 строк:
```sql
SELECT s.store_id, SUM(s.product_units) AS product_amount
FROM marketing.sales AS s
GROUP BY (s.store_id)
ORDER BY product_amount DESC
LIMIT 20
```

### ESTIMATE_ONLY {#estimate_example}

Запрос на получение информации о запросе:
```sql
SELECT s.store_id, SUM(s.product_units) AS product_amount
FROM marketing.sales AS s
GROUP BY (s.store_id)
ORDER BY product_amount DESC
ESTIMATE_ONLY
```

Описание ключевого слова см. в секции [Ключевое слово ESTIMATE_ONLY](#estimate).

### COLLATE {#collate_example}

Запрос строк с указанными значениями без учета регистра:
```sql
SELECT * from marketing.sales 
WHERE product_code = 'ABC1234' AND product_code <> 'abc4567' COLLATE 'unicode_ci'
DATASOURCE_TYPE = 'adg'
```

Описание ключевого слова см. в секции [Ключевое слово COLLATE](#collate).

### OFFSET {#offset_example}

Запрос 20 строк, начиная с десятой:
```sql
SELECT * from marketing.sales FETCH NEXT 20 ROWS ONLY OFFSET 9
```

Описание ключевого слова см. в секции [Ключевое слово OFFSET](#offset).

### ORDER BY, LIMIT и OFFSET {#offset_and_order_by_example}

Запрос 20 строк, упорядоченных по значению `id` и выбираемых начиная с десятой строки 
результата:
```sql
SELECT * from marketing.sales ORDER BY id LIMIT 20 OFFSET 9
```

Такое сочетание ключевых слов позволяет выбирать данные порциями с сохранением их порядка.

### FOR SYSTEM_TIME AS OF DELTA_NUM {#for_system_time_example}

Запрос записей, актуальных на момент закрытия дельты с номером 9, из материализованного представления:
```sql
SELECT * FROM marketing.sales_and_stores FOR SYSTEM_TIME AS OF DELTA_NUM 9
```

Описание ключевого слова см. в секции [Ключевое слово FOR SYSTEM_TIME](#for_system_time).

### Соединение таблиц из разных логических БД {#join_example}

Запрос с соединением данных логических таблиц из двух разных логических БД:
```sql
SELECT
  st.id,
  st.category,
  s.product_code
FROM marketing.stores AS st
INNER JOIN marketing_new.sales AS s
  ON st.id = s.store_id
```

О возможных типах соединений см. в секции [Поддерживаемые типы соединений](#join_prefixes).

### Соединение изменений из разных дельт {#deltas_example}

Запрос с соединением записей логической таблицы, добавленных и измененных в двух различных диапазонах 
дельт:
```sql
  SELECT st.id, st.category, s.product_code 
  FROM marketing.stores FOR SYSTEM_TIME STARTED IN(0,7) AS st 
  INNER JOIN marketing.sales FOR SYSTEM_TIME STARTED IN(0,1) AS s 
  ON st.id = s.store_id
```

О возможных типах соединений см. в секции [Поддерживаемые типы соединений](#join_prefixes).

### Запрос из standalone-таблицы {#standalone_example}

```sql
-- запрос данных из standalone-таблицы, на которую указывает внешняя readable-таблица payments_ext_read_adg
SELECT p.agreement_id, p.code, SUM(p.amount) AS amount, p.currency_code 
FROM marketing.payments_ext_read_adg AS p 
GROUP BY p.agreement_id, p.code, p.currency_code
```

### Соединение standalone-таблицы и логической таблицы {#two_type_example}

```sql
-- запрос данных из логической таблицы clients и standalone-таблицы, на которую указывает 
--   внешняя readable-таблица agreements_ext_read_adp
SELECT a.id, a.client_id, c.last_name, c.first_name, c.patronymic_name 
FROM marketing.agreements_ext_read_adp AS a
LEFT JOIN marketing.clients FOR SYSTEM_TIME AS OF delta_num 9 AS c
  ON a.client_id = c.id
```