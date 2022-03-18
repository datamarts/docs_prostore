---
layout: default
title: Загрузка данных
nav_order: 3
parent: Работа с системой
has_children: true
has_toc: false
---

# Загрузка данных {#data_upload}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Система позволяет параллельно загружать большие объемы данных. 

Можно загружать данные в [логические таблицы](../../overview/main_concepts/logical_table/logical_table.md) и 
[внешние writable-таблицы](../../overview/main_concepts/external_table/external_table.md#writable_table).
Загрузка данных в [логические](../../overview/main_concepts/logical_view/logical_view.md) 
и [материализованные представления](../../overview/main_concepts/materialized_view/materialized_view.md) 
недоступна.

Под большим объемом данных подразумевается количество записей от нескольких сотен до нескольких миллионов. 
Для загрузки небольшого объема данных можно использовать функцию [обновления данных](../data_update/data_update.md).
{: .note-wrapper}

## Подготовка к загрузке {#preparing}

Данные загружаются в систему из [сообщений топиков Kafka](../../reference/upload_format/upload_format.md). 
Поэтому, если в брокере сообщений Kafka не настроено автоматическое создание топиков, нужно создать топики вручную.
Чтобы создать топик, следуйте любой из инструкций в документации Kafka:
*   [Quick Start](https://kafka.apache.org/documentation/#quickstart),
*   [Adding and removing topics](https://kafka.apache.org/documentation/#basic_ops_add_topic).

Рекомендации о разделении данных по топикам см. в разделе [Внешняя таблица](../../overview/main_concepts/external_table/external_table.md).
{: .tip-wrapper}

Перед загрузкой данных подготовьте данные и логические сущности:
   1. Загрузите данные из внешней информационной системы в топик Kafka.  
      Данные должны соответствовать [формату загрузки данных](../../reference/upload_format/upload_format.md).
   2. [Создайте логическую таблицу](../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md) или 
      [внешнюю writable-таблицу](../../reference/sql_plus_requests/CREATE_WRITABLE_EXTERNAL_TABLE/CREATE_WRITABLE_EXTERNAL_TABLE.md), 
      если она еще не создана.
   3. [Создайте](../../reference/sql_plus_requests/CREATE_UPLOAD_EXTERNAL_TABLE/CREATE_UPLOAD_EXTERNAL_TABLE.md)
      [внешнюю таблицу](../../overview/main_concepts/external_table/external_table.md)
      загрузки, если она еще не создана.

## Загрузка данных в логическую таблицу {#upload_to_logical_table}

Чтобы загрузить данные из внешней информационной системы в логическую таблицу:
1. Выполните запрос [BEGIN DELTA](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)
   на открытие [дельты](../../overview/main_concepts/delta/delta.md), если она еще не открыта.
2. Выполните запрос [INSERT FROM upload_external_table](../../reference/sql_plus_requests/INSERT_FROM_upload_external_table/INSERT_FROM_upload_external_table.md)
   на загрузку данных. В запросе нужно указать внешнюю таблицу загрузки, определяющую параметры загрузки.
3. Если необходимо, загрузите или обновите другие данные.
   <br>В открытой дельте можно выполнять множество запросов на обновление и загрузку данных. При этом в каждую логическую
   таблицу в одной дельте можно добавлять записи с разными значениями первичного ключа или полные дубликаты.
4. Выполните запрос [COMMIT DELTA](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md)
   для сохранения изменений и закрытия дельты.

При успешном выполнении действий состояние данных в логических таблицах обновляется, как описано в разделе
[Версионирование данных](data_versioning/data_versioning.md).

Пока дельта не закрыта, все ее изменения данных можно отменить запросом
[ROLLBACK DELTA](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).
<br> Созданные внешние таблицы загрузки можно использовать повторно или удалить.
{: .note-wrapper}

## Загрузка данных в writable-таблицу {#upload_to_writable_table}

Чтобы загрузить данные из внешней информационной системы во внешнюю writable-таблицу, выполните запрос 
[INSERT FROM upload_external_table](../../reference/sql_plus_requests/INSERT_FROM_upload_external_table/INSERT_FROM_upload_external_table.md) 
на загрузку данных. В запросе нужно указать внешнюю таблицу загрузки, определяющую параметры загрузки.

При загрузке данных в writable-таблицу нужно учитывать ограничения, которые конкретная СУБД накладывает на
работу со своими таблицами.
{: .note-wrapper}

## Примеры {#examples}

### Загрузка в логическую таблицу {#logical_table_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание логической таблицы
CREATE TABLE sales (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id);

-- создание внешней таблицы загрузки
CREATE UPLOAD EXTERNAL TABLE sales_ext_upload (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales'
FORMAT 'AVRO'
MESSAGE_LIMIT 1000;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- запуск загрузки данных в логическую таблицу
INSERT INTO sales SELECT * FROM sales.sales_ext_upload;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Загрузка во внешнюю writable-таблицу {#writable_table_example}

Загрузка через внешнюю writable-таблицу в standalone-таблицу ADP, созданную через систему (см. параметр `auto.create.table.enable=true`):

```sql
-- создание внешней writable-таблицы
CREATE WRITABLE EXTERNAL TABLE sales.agreements_ext_write_adp (
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
LOCATION 'core:adp://sales.agreements'
OPTIONS ('auto.create.table.enable=true');

-- создание внешней таблицы загрузки
CREATE UPLOAD EXTERNAL TABLE sales.agreements_ext_upload (
  id INT NOT NULL,
  client_id INT NOT NULL,
  number VARCHAR NOT NULL,
  signature_date DATE,
  effective_date DATE,
  closing_date DATE,
  description VARCHAR
) 
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/agreements'
FORMAT 'AVRO'
OPTIONS ('auto.create.sys_op.enable=false')

-- запуск загрузки данных во внешнюю writable-таблицу
INSERT INTO sales.agreements_ext_write_adp SELECT * FROM sales.agreements_ext_upload;
```

Подробнее об опции `auto.create.table.enable` см. в разделах 
[CREATE WRITABLE EXTERNAL TABLE](../../reference/sql_plus_requests/CREATE_WRITEABLE_EXTERNAL_TABLE/CREATE_WRITEABLE_EXTERNAL_TABLE.md) и 
[CREATE READABLE EXTERNAL TABLE](../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md).
{: .note-wrapper}

Загрузка через внешнюю writable-таблицу в существующую standalone-таблицу ADG:

```sql
-- создание внешней writable-таблицы
CREATE WRITABLE EXTERNAL TABLE sales.payments_ext_write_adg (
id INT NOT NULL,
agreement_id INT,
code VARCHAR(16),
amount DOUBLE,
currency_code VARCHAR(3),
description VARCHAR,
bucket_id INT NOT NULL
)
LOCATION 'core:adg://dtm__sales__payments';

-- создание внешней таблицы загрузки
CREATE UPLOAD EXTERNAL TABLE sales.payments_ext_upload (
id INT NOT NULL,
agreement_id INT,
code VARCHAR(16),
amount DOUBLE,
currency_code VARCHAR(3),
description VARCHAR
)
LOCATION  'kafka://$kafka/payments'
FORMAT 'AVRO'
OPTIONS ('auto.create.sys_op.enable=false');

-- запуск загрузки данных во внешнюю writable-таблицу
INSERT INTO sales.payments_ext_write_adg SELECT *, 0 as bucket_id FROM sales.payments_ext_upload;
```