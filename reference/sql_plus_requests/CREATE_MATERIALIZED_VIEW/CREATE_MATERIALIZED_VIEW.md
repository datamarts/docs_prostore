---
layout: default
title: CREATE MATERIALIZED VIEW
nav_order: 16
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE MATERIALIZED VIEW
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет создать [материализованное представление](../../../overview/main_concepts/materialized_view/materialized_view.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md).
Материализованное представление может содержать результаты запроса к данным одной или нескольких 
[логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md).

Данные представлений можно размещать в ADG и (или) ADQM. Источником данных для представлений служит ADB.
{: .note-wrapper}

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

При успешном выполнении запроса система создает материализованное представление, а также подготавливает
[хранилище](../../../overview/main_concepts/data_storage/data_storage.md) к размещению данных
представления — создает [физические таблицы](../../../overview/main_concepts/physical_table/physical_table.md),
связанные с материализованным представлением и предназначенные для хранения его данных. Физические таблицы создаются в 
тех СУБД хранилища, которые указаны в запросе.

В отличие от запроса на создание логической таблицы, запрос `CREATE MATERIALIZED VIEW` должен содержать ключевое 
слово [DATASOURCE_TYPE](#datasource_type) со списком СУБД для размещения данных представления. Требование связано с тем, 
что данные представлений (в отличие от данных логических таблиц) могут размещаться только в ADG и ADQM, а не во всех СУБД 
хранилища.

Созданное представление начинает синхронизироваться в первом цикле синхронизации, доступном после создания представления, 
в порядке очереди. Статус синхронизации можно узнать с помощью запроса 
[CHECK_MATERIALIZED_VIEW](../CHECK_MATERIALIZED_VIEW/CHECK_MATERIALIZED_VIEW.md). Подробнее о синхронизации см. в разделе 
[Синхронизация материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md#synchronization).

Изменение материализованного представления недоступно. Для замены материализованного 
представления необходимо удалить его и создать новое.
{: .note-wrapper}

Если при обработке запроса произошла ошибка, изменение сущностей логической базы данных становится недоступно. В этом
случае нужно выполнить запрос [ERASE_CHANGE_OPERATION](../ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md).

Каждое создание представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md). 

## Синтаксис {#syntax}

```sql
CREATE MATERIALIZED VIEW [db_name.]materialized_view_name (
  column_name_1 datatype_1 [ NULL | NOT NULL ],
  column_name_2 datatype_2 [ NULL | NOT NULL ],
  column_name_3 datatype_3 [ NULL | NOT NULL ],
  PRIMARY KEY (column_list_1)
) DISTRIBUTED BY (column_list_2)
DATASOURCE_TYPE (datasource_aliases)
AS SELECT query
DATASOURCE_TYPE = origin_datasource_alias
[LOGICAL_ONLY]
```

**Параметры:**

`db_name`

: Имя логической базы данных, в которой создается материализованное представление. 
  Опционально, если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`materialized_view_name`

: Имя создаваемого материализованного представления, уникальное среди логических 
  сущностей логической БД.

`column_name_N`

: Имя столбца представления.

`datatype_N`

: Тип данных столбца `column_name_N`. Возможные значения см. 
  в разделе [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md).

`column_list_1`

: Список столбцов, входящих в первичный ключ представления.

`column_list_2`

: Список столбцов, входящих в ключ шардирования представления. Столбцы должны быть из числа столбцов `column_list_1`.

`datasource_aliases`

: Список псевдонимов СУБД хранилища, в которых нужно разместить данные представления. 
  Элементы списка перечисляются через запятую. Возможные значения: `adqm`, `adg`. 
  Значения можно указывать без кавычек (например, `adg`) или двойных кавычках (например, `"adg"`).

`query`

: [SELECT](../SELECT/SELECT.md)-подзапрос, на основе которого строится представление.

`origin_datasource_alias`

: Псевдоним СУБД, которая служит источником данных. 
  Возможные значения: `'adb'`. Значение указывается в одинарных кавычках.

### Ключевое слово DATASOURCE_TYPE {#datasource_type}

Ключевое слово `DATASOURCE_TYPE` позволяет указать СУБД хранилища, в которых необходимо
размещать данные материализованного представления. В текущей версии данные представления могут 
размещаться в ADG и (или) в ADQM.

### Ключевое слово LOGICAL_ONLY {#logical_only}

Ключевое слово `LOGICAL_ONLY` позволяет создать материализованное представление только на логическом уровне
(в [логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md)), без
пересоздания связанных [физических таблиц](../../../overview/main_concepts/physical_table/physical_table.md)
в хранилище данных.

Если ключевое слово не указано, создается как материализованное представление, так и связанные 
с ним физические таблицы.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Имена представления и его столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md) 
  и должны начинаться с латинской буквы. После первого символа могут следовать латинские буквы, цифры и символы 
  подчеркивания в любом порядке.
* Столбцы также не могут иметь имена, зарезервированные системой для служебного использования: `sys_op`, `sys_from`, 
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
* Имена столбцов должны быть уникальны в рамках представления.
* Имена, порядок и типы данных столбцов должны совпадать в SELECT-подзапросе и представлении.
* Первичный ключ должен включать все столбцы ключа шардирования.
* Логические таблицы, указанные в SELECT-подзапросе, должны принадлежать той же логической базе данных, 
  что и материализованное представление.
* Подзапрос не может содержать: 
  * ключевое слово [FOR SYSTEM_TIME](../SELECT/SELECT.md#for_system_time),
  * ключевое слово `ORDER BY`,
  * ключевое слово `LIMIT`.

## Примеры {#examples}

### Представление на основе одной таблицы с условием {#example_with_condition}

Создание представления с размещением в ADG и ADQM:

```sql
CREATE MATERIALIZED VIEW marketing.sales_december_2020 (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id)
DATASOURCE_TYPE (adg, adqm)
AS SELECT * FROM marketing.sales
   WHERE cast(transaction_date as date) BETWEEN '2020-12-01' AND '2020-12-31'
DATASOURCE_TYPE = 'adb'
```

### Представление на основе одной таблицы с условием, агрегацией и группировкой {#example_with_group_by}

Создание представления с размещением в ADQM:

```sql
CREATE MATERIALIZED VIEW marketing.sales_by_stores (
  store_id INT NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  PRIMARY KEY (store_id, product_code)
)
DISTRIBUTED BY (store_id)
DATASOURCE_TYPE (adqm)
AS SELECT store_id, product_code, SUM(product_units) as product_units FROM marketing.sales
   WHERE product_code <> 'ABC0001'
   GROUP BY store_id, product_code
DATASOURCE_TYPE = 'adb'
```

### Представление на основе двух таблиц {#example_with_join}

```sql
CREATE MATERIALIZED VIEW marketing.sales_and_stores (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  description VARCHAR(256),
  store_id INT NOT NULL,
  store_category VARCHAR(256) NOT NULL,
  region VARCHAR(256) NOT NULL,
  PRIMARY KEY (id, region)
)
DISTRIBUTED BY (id)
DATASOURCE_TYPE (adg)
AS SELECT
 s.id, s.transaction_date, s.product_code, s.product_units, s.description,
 st.id AS store_id, st.category as store_category, st.region
 FROM marketing.sales AS s
 JOIN marketing.stores AS st
 ON s.store_id = st.id
DATASOURCE_TYPE = 'adb'
```

### Представление только на логическом уровне {#logical_example}

```sql
CREATE MATERIALIZED VIEW marketing.stores_by_sold_products_matview (
  store_id INT NOT NULL,
  product_amount INT NOT NULL,
  PRIMARY KEY (store_id)
)
DISTRIBUTED BY (store_id)
DATASOURCE_TYPE (adg)
AS SELECT store_id, SUM(product_units) AS product_amount
  FROM marketing.sales
  GROUP BY store_id
DATASOURCE_TYPE = 'adb'
LOGICAL_ONLY
```