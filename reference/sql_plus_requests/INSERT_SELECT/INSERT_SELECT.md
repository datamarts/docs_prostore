---
layout: default
title: INSERT SELECT
nav_order: 37
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# INSERT SELECT
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет вставить записи в [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
или [внешнюю writable-таблицу](../../../overview/main_concepts/external_table/external_table.md#writable_table) 
(далее — целевая таблица) из другой логической сущности. При вставке записей в writable-таблицу нужно учитывать 
ограничения, которые конкретная СУБД накладывает на работу со своими таблицами.

Источником записей может быть:
* логическая таблица, 
* [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) представление, 
* [материализованное представление](../../../overview/main_concepts/materialized_view/materialized_view.md),
* внешняя writable-таблица,
* соединение любых из перечисленных сущностей.

Вставка записей в логические и материализованные представления недоступна.

Запрос поддерживается для ADB, ADQM и ADP.
{: .note-wrapper}

Для вставки большого объема данных следует использовать
[загрузку данных](../../../working_with_system/data_upload/data_upload.md) или сочетание
[выгрузки](../../../working_with_system/data_download/data_download.md) и загрузки данных.
{: .tip-wrapper}

Источником данных всегда служит одна [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md):
* [указанная](../../../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type) или
  [наиболее оптимальная СУБД](../../../working_with_system/data_reading/routing/routing.md) —
  если данные выбираются из логических таблиц, логических и материализованных представлений;
* та СУБД хранилища, в которой размещаются связанные 
  [standalone-таблицы](../../../overview/main_concepts/standalone_table/standalone_table.md), — если данные выбираются 
  из [внешних readable-таблиц](../../../overview/main_concepts/external_table/external_table.md#readable_table) 
  и их соединений с другими сущностями.

Вставка данных возможна при любом из условий:
* источник данных и данные целевой таблицы находятся в одной СУБД хранилища,
* источник данных находится в ADB, а данные целевой таблицы — в ADB и (или) ADQM.

Расположение данных логической таблицы можно задавать запросами
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с ключевым словом
`DATASOURCE_TYPE`.
{: .note-wrapper}

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

В зависимости от вида таблицы, указанной в запросе, записи вставляются:
* в [физические таблицы](../../../overview/main_concepts/physical_table/physical_table.md) тех СУБД хранилища, 
  в которых размещены данные указанной логической таблицы;
* в standalone-таблицу, которая связана с указанной внешней таблицей.

Записи, вставленные в логическую таблицу, добавляются как горячие записи. При [фиксации изменений](../COMMIT_DELTA/COMMIT_DELTA.md) 
записи становятся актуальными, а предыдущие актуальные записи — архивными. 
Наличие предыдущей актуальной записи определяется по первичному ключу: все записи логической таблицы с одинаковым 
первичным ключом рассматриваются системой как разные исторические состояния одного объекта.
Подробнее о версионировании записей 
см. в разделе [Версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).

Если [операция записи](../../../overview/main_concepts/write_operation/write_operation.md), запущенная запросом
`INSERT SELECT`, зависла, нужно повторить запрос, указав ключевое слово [RETRY](#retry).

Вставка данных из readable-таблицы в логическую таблицу может привести к расхождениям между СУБД. 
Расхождения возникают, если логическая таблица размещена в нескольких СУБД и за время выполнения запроса данные 
в standalone-таблице успевают обновиться, а некоторые коннекторы успевают вычитать и записать в свою СУБД больше данных, 
чем другие. 
<br>Найти расхождения в данных между СУБД можно с помощью запроса [CHECK_SUM](../CHECK_SUM/CHECK_SUM.md). 
Предотвращение таких расхождений зависит от конкретной инсталляции и решается сторонними средствами. 
Если расхождения возникли, их нужно устранить вручную.
{: .warning-wrapper}

## Синтаксис {#syntax}

Вставка данных во все столбцы таблицы:
```sql
INSERT INTO [db_name.]table_name SELECT query
```

Вставка данных в некоторые столбцы таблицы 
(с заполнением остальных столбцов значениями по умолчанию):
```sql
INSERT INTO [db_name.]table_name (column_list) SELECT query
```

Перезапуск операции по вставке данных:
```sql
RETRY INSERT INTO [db_name.]table_name [(column_list)] SELECT query
```

**Параметры:**

`db_name`

: Имя логической базы данных. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`table_name`

: Имя логической или внешней таблицы, в которую вставляются данные.

`column_list`

: Список имен столбцов указанной таблицы. Имена перечисляются в круглых скобках через запятую. 
  Список опционален, если количество и порядок столбцов в SELECT-подзапросе соответствуют количеству и порядку столбцов 
  в таблице.

`query`

: [SELECT](../SELECT/SELECT.md)-подзапрос для выбора данных.

### Ключевое слово RETRY {#retry}

Ключевое слово перезапускает обработку операции записи, созданной запросом `INSERT SELECT`.
Это может быть полезно, если операция зависла: дельту невозможно [закрыть](../COMMIT_DELTA/COMMIT_DELTA.md) или
[откатить](../ROLLBACK_DELTA/ROLLBACK_DELTA.md), пока есть зависшая операция. Пример запроса см. [ниже](#retry_example).

Перезапустить обработку можно только для незавершенных операций.
Ключевое слово `RETRY` недоступно в запросах на вставку записей во внешнюю writable-таблицу.

Если ключевое слово не указано, система создает новую операцию и обрабатывает ее.

Список незавершенных (в том числе — зависших) операций можно посмотреть можно с помощью запроса
[GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).
{: .tip-wrapper}

## Ограничения {#restrictions}

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Типы вставляемых данных должны соответствовать типам данных столбцов в целевой логической таблице.
* Не допускается параллельное выполнение идентичных запросов.

## Примеры {#examples}

### Вставка данных во все столбцы логической таблицы {#all_columns_of_logical_table}

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
INSERT INTO sales_july_2021 
SELECT * FROM sales WHERE CAST(EXTRACT(MONTH FROM transaction_date) AS INT) = 7 AND 
  CAST(EXTRACT(YEAR FROM transaction_date) AS INT) = 2021 DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в некоторые столбцы логической таблицы {#some_columns_of_logical_table}

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
INSERT INTO current_stores (id, category, region, address)
SELECT id, category, region, address FROM stores FOR SYSTEM_TIME AS OF DELTA_NUM 10 DATASOURCE_TYPE = 'adqm';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в логическую таблицу из другой логической БД {#other_db_example}

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
INSERT INTO sales SELECT * FROM sales.sales WHERE store_id BETWEEN 1234 AND 4567 DATASOURCE_TYPE = 'adp';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в логическую таблицу из логического представления {#view_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логического представления basic_stores с данными о магазинах категории basic
CREATE VIEW basic_stores AS SELECT * FROM stores WHERE category = 'basic';

-- создание таблицы basic_stores_table с данными о магазинах категории basic (с размещением данных в ADB и ADQM)
CREATE TABLE basic_stores_table (
id INT NOT NULL,
category VARCHAR(256),
region VARCHAR(256),
address VARCHAR(256),
description VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adb, adqm);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных в таблицу basic_stores_table
INSERT INTO basic_stores_table SELECT * FROM basic_stores DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Заполнение столбца логической таблицы данными другой таблицы {#column_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логической таблицы с данными покупок и адресов магазинов, где были совершены покупки 
CREATE TABLE sales_with_address (
id INT NOT NULL,
transaction_date TIMESTAMP NOT NULL,
product_code VARCHAR(256),
product_units INT,
store_id INT,
description VARCHAR(256),
store_address VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adb, adqm);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных из таблицы sales (заполнение всех столбцов, кроме store_address)
INSERT INTO sales_with_address (id, transaction_date, product_code, product_units, store_id, description)
SELECT * FROM sales DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;

-- открытие новой (горячей) дельты
BEGIN DELTA;

--- вставка данных адресов из таблицы stores в те строки, где адрес не заполнен
INSERT INTO sales_with_address
SELECT s.id, s.transaction_date, s.product_code, s.product_units, s.store_id, s.description, 
st.region || ', ' || st.address as store_address
FROM stores AS st 
JOIN sales_with_address AS s ON s.store_id = st.id 
WHERE s.store_address IS NULL OR s.store_address = ''
DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в логическую таблицу из внешней readable-таблицы {#readable_to_logical_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логической таблицы agreements_adp с размещением данных в ADP
CREATE TABLE agreements_adp (
  id INT NOT NULL,
  client_id INT NOT NULL,
  number VARCHAR NOT NULL,
  signature_date DATE,
  effective_date DATE,
  closing_date DATE,
  description VARCHAR,
  PRIMARY KEY(id)
)
DISTRIBUTED BY (id)
DATASOURCE_TYPE (adp);

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка данных в логическую таблицу agreements_adp из внешней readable-таблицы agreements_ext_read_adp
INSERT INTO agreements_adp SELECT * FROM agreements_ext_read_adp;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных во внешнюю writable-таблицу из логической таблицы {#logical_to_writable_example}

```sql
-- создание внешней writable-таблицы, связанной с таблицей ADQM
CREATE WRITABLE EXTERNAL TABLE sales.sales_ext_write_adqm (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id)
LOCATION 'core:adqm://dtm__sales.sales'
OPTIONS ('auto.create.table.enable=true');

-- вставка данных во внешнюю writable-таблицу sales_ext_write_adqm из логической таблицы sales
INSERT INTO sales.sales_ext_write_adqm SELECT * FROM sales.sales DATASOURCE_TYPE = 'adqm';
```

### Перезапуск операции по вставке записей {#retry_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;
   
-- вставка записей в таблицу current_stores без указания значения столбца description
INSERT INTO current_stores 
       (id, category, region, address)
SELECT id, category, region, address 
FROM stores FOR SYSTEM_TIME AS OF DELTA_NUM 12 DATASOURCE_TYPE = 'adqm';

-- перезапуск обработки операции по вставке записи
RETRY INSERT INTO current_stores 
       (id, category, region, address)
SELECT id, category, region, address 
FROM stores FOR SYSTEM_TIME AS OF DELTA_NUM 12 DATASOURCE_TYPE = 'adqm';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```  