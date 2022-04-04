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
[standalone-таблицы](../../overview/main_concepts/standalone_table/standalone_table.md).
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
      [standalone-таблицу](../../overview/main_concepts/standalone_table/standalone_table.md), если она еще не создана.
   3. [Создайте](../../reference/sql_plus_requests/CREATE_UPLOAD_EXTERNAL_TABLE/CREATE_UPLOAD_EXTERNAL_TABLE.md)
      [внешнюю таблицу загрузки](../../overview/main_concepts/external_table/external_table.md#upload_table), если она 
      еще не создана.
   4. Если нужно загрузить данные в standalone-таблицу, 
      [создайте](../../reference/sql_plus_requests/CREATE_WRITABLE_EXTERNAL_TABLE/CREATE_WRITABLE_EXTERNAL_TABLE.md)
      [внешнюю writable-таблицу](../../overview/main_concepts/external_table/external_table.md#writable_table).

## Загрузка данных в логическую таблицу {#upload_to_logical_table}

Чтобы загрузить данные из внешней информационной системы в логическую таблицу:
1. Выполните запрос [BEGIN DELTA](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)
   на открытие [дельты](../../overview/main_concepts/delta/delta.md), если она еще не открыта.
2. Выполните запрос [INSERT SELECT FROM upload_external_table](../../reference/sql_plus_requests/INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md)
   на загрузку данных. В запросе нужно указать внешнюю таблицу загрузки, определяющую параметры загрузки.
3. Если необходимо, загрузите или [обновите](../data_update/data_update.md) другие данные.
   <br>В открытой дельте можно выполнять множество запросов на обновление и загрузку данных. При этом в каждую логическую
   таблицу в одной дельте можно добавлять записи с разными значениями первичного ключа или полные дубликаты.
4. Выполните запрос [COMMIT DELTA](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md)
   для сохранения изменений и закрытия дельты.

При успешном выполнении действий состояние данных в логических таблицах обновляется, как описано в разделе
[Версионирование данных](data_versioning/data_versioning.md).

Пока дельта не закрыта, внесенные изменения можно отменить запросом 
[ROLLBACK DELTA](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).
{: .note-wrapper}

Незавершенную [операцию](../../overview/main_concepts/write_operation/write_operation.md) по загрузке данных можно 
перезапустить или отменить. Подробнее о способах обработки незавершенных операций см. в разделе
[Управление операциями записи](../../working_with_system/operation_management/write_op_management/write_op_management.md).
{: .note-wrapper}

Созданные внешние таблицы загрузки можно использовать повторно или удалить.

## Загрузка данных в standalone-таблицу {#upload_to_standalone_table}

Чтобы загрузить данные из внешней информационной системы в standalone-таблицу, выполните запрос 
[INSERT SELECT FROM upload_external_table](../../reference/sql_plus_requests/INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md) 
на загрузку данных. В запросе нужно указать внешнюю таблицу загрузки, определяющую параметры загрузки, а также
внешнюю writable-таблицу, ссылающуюся на целевую standalone-таблицу.

При загрузке данных в standalone-таблицу нужно учитывать ограничения таблицы в конкретной СУБД.
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

### Загрузка в standalone-таблицу {#standalone_table_example}

Загрузка в standalone-таблицу ADP, созданную через систему (см. параметр `auto.create.table.enable=true`):

```sql
-- создание внешней writable-таблицы с созданием связанной standalone-таблицы в ADP
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

-- запуск загрузки данных в standalone-таблицу, на которую указывает внешняя writable-таблица agreements_ext_write_adp
INSERT INTO sales.agreements_ext_write_adp SELECT * FROM sales.agreements_ext_upload;
```

Подробнее об опции `auto.create.table.enable` см. в разделах 
[CREATE WRITABLE EXTERNAL TABLE](../../reference/sql_plus_requests/CREATE_WRITEABLE_EXTERNAL_TABLE/CREATE_WRITEABLE_EXTERNAL_TABLE.md) и 
[CREATE READABLE EXTERNAL TABLE](../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md).
{: .note-wrapper}

Загрузка в существующую standalone-таблицу ADG:

```sql
-- создание внешней writable-таблицы, указывающей на существующую standalone-таблицу ADG
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

-- запуск загрузки данных в standalone-таблицу, на которую указывает внешняя writable-таблица payments_ext_write_adg
INSERT INTO sales.payments_ext_write_adg SELECT *, 0 as bucket_id FROM sales.payments_ext_upload;
```