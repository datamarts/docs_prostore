---
layout: default
title: UPSERT SELECT
nav_order: 38
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# UPSERT SELECT
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет вставить несколько записей в [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
(далее — целевая таблица) из другой логической сущности: логической таблицы, 
[логического](../../../overview/main_concepts/logical_view/logical_view.md) 
или [материализованного представления](../../../overview/main_concepts/materialized_view/materialized_view.md).
Вставка данных в логические и материализованные представления 
недоступна.

Запрос UPSERT SELECT не поддерживается для ADG.
{: .note-wrapper}

Источником данных служит одна [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md): указанная в запросе или, если такая 
не указана, — СУБД хранилища, наиболее оптимальная для исполнения запроса. Подробнее о выборе оптимальной СУБД для 
исполнения запроса см. в разделе
[Маршрутизация запросов к данным](../../../working_with_system/data_reading/routing/routing.md).

Вставка данных возможна, если выполнено любое из условий:
* данные целевой таблицы размещены только в одной СУБД хранилища, и источником данных служит та же СУБД 
  хранилища;
* данные целевой таблицы размещены в ADB, ADQM и (или) ADG, и источником данных служит ADB.

Месторасположение данных логической таблицы можно задавать запросами
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с указанием ключевого слова
`DATASOURCE_TYPE`.
{: .note-wrapper}

Для вставки большого объема данных следует использовать 
[загрузку данных](../../../working_with_system/data_upload/data_upload.md) или сочетание 
[выгрузки](../../../working_with_system/data_download/data_download.md) и загрузки данных.
{: .note-wrapper}

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

При успешном выполнении запроса вставленные записи сохраняются как актуальные записи, а предыдущие актуальные записи, 
если такие найдены, перемещаются в архив. Наличие предыдущей актуальной записи определяется по первичному ключу: 
все записи таблицы с одинаковым первичным ключом рассматриваются системой как различные исторические состояния одного 
объекта. Подробнее о версионировании
см. в разделе [Версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).

## Синтаксис {#syntax}

Вставка данных во все столбцы логической таблицы:
```sql
UPSERT INTO [db_name.]table_name SELECT query
```

Вставка данных только в некоторые столбцы логической таблицы 
(с заполнением остальных столбцов значениями, которые определены в СУБД хранилища как значения по умолчанию):
```sql
UPSERT INTO [db_name.]table_name (column_list) SELECT query
```

Параметры:
* `db_name` — имя логической базы данных. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
* `table_name` — имя логической таблицы, в которую вставляются данные;
* `column_list` — список имен столбцов логической таблицы. Имена указываются в круглых скобках через запятую. 
  Список опционален, если количество и порядок столбцов в SELECT-подзапросе соответствуют количеству и порядку столбцов 
  в логической таблице;
* `query` — [SELECT](../SELECT/SELECT.md)-подзапрос для выбора данных. Если в подзапросе указано ключевое слово 
  `DATASOURCE_TYPE` с псевдонимом СУБД хранилища, данные выбираются из указанной СУБД, иначе — из СУБД, 
  которая является [наиболее оптимальной](../../../working_with_system/data_reading/routing/routing.md) 
  для исполнения запроса.

## Ограничения {#restrictions}

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Столбцы в запросе не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`,
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
* Типы вставляемых данных должны соответствовать типам данных столбцов целевой логической таблицы.

## Пример {#examples}

### Вставка данных во все столбцы таблицы {#all_columns_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логической таблицы sales_july_2021 с данными о продажах за июль 2021 (с размещением данных в ADB)
CREATE TABLE sales_july_2021 (
id INT NOT NULL,
transaction_date TIMESTAMP NOT NULL,
product_code VARCHAR(256) NOT NULL,
product_units INT NOT NULL,
store_id INT NOT NULL,
description VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adb);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных из таблицы sales в таблицу sales_july_2021 
UPSERT INTO sales_july_2021 
SELECT * FROM sales WHERE CAST(EXTRACT(MONTH FROM transaction_date) AS INT) = 7 AND 
  CAST(EXTRACT(YEAR FROM transaction_date) AS INT) = 2021 DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в некоторые столбцы таблицы {#some_columns_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логической таблицы current_stores с выборкой из таблицы stores (с размещением данных в ADQM)
CREATE TABLE current_stores (
  id INT NOT NULL,
  category VARCHAR(256),
  region VARCHAR(256),
  address VARCHAR(256),
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id)
DATASOURCE_TYPE (adqm);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных в таблицу current_stores без указания значения столбца description
UPSERT INTO current_stores (id, category, region, address)
SELECT id, category, region, address FROM stores FOR SYSTEM_TIME AS OF DELTA_NUM 10 DATASOURCE_TYPE = 'adqm';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных из таблицы другой логической БД {#other_db_example}

```sql
-- создание новой логической БД sales_new
CREATE DATABASE sales_new;

-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales_new;

-- создание логической таблицы sales в логической БД sales_new (с размещением данных в ADP)
CREATE TABLE sales (
id INT NOT NULL,
transaction_date TIMESTAMP NOT NULL,
product_code VARCHAR(256) NOT NULL,
product_units INT NOT NULL,
store_id INT NOT NULL,
description VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adp);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных в таблицу sales из аналогичной таблицы другой логической БД
UPSERT INTO sales SELECT * FROM sales.sales WHERE store_id BETWEEN 1234 AND 4567 DATASOURCE_TYPE = 'adp';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных из логического представления {#view_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логического представления basic_stores с данными о магазинах категории basic
CREATE VIEW basic_stores AS SELECT * FROM stores WHERE category = 'basic';

-- создание таблицы basic_stores_table с данными о магазинах категории basic (с размещением данных в ADB, ADQM и ADG)
CREATE TABLE basic_stores_table (
id INT NOT NULL,
category VARCHAR(256),
region VARCHAR(256),
address VARCHAR(256),
description VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adb, adqm, adg);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных в таблицу basic_stores_table
UPSERT INTO basic_stores_table SELECT * FROM basic_stores DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

