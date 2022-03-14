---
layout: default
title: Выгрузка данных
nav_order: 5
parent: Работа с системой
has_children: false
---

# Выгрузка данных {#data_download}

Система позволяет выгружать большие объемы данных, а также изменений, выполненных 
в указанных дельтах. Данные можно выгружать из [логических таблиц](../../overview/main_concepts/logical_table/logical_table.md), 
[логических](../../overview/main_concepts/logical_view/logical_view.md) и 
[материализованных представлений](../../overview/main_concepts/materialized_view/materialized_view.md), а также 
внешних readable-таблиц.

Под большим объемом данных подразумевается количество записей от нескольких сотен до нескольких миллионов.
Для получения небольшого объема данных можно использовать функцию [запроса данных](../data_reading/data_reading.md).
{: .note-wrapper}

Данные выгружаются из системы в виде [сообщений Kafka](../../reference/upload_format/upload_format.md).
Поэтому, если в брокере сообщений Kafka не настроено автоматическое создание топиков, нужно создать топики вручную.
Чтобы создать топик, следуйте любой из инструкций, доступных в документации Kafka:
* [Quick Start](https://kafka.apache.org/documentation/#quickstart),
* [Adding and removing topics](https://kafka.apache.org/documentation/#basic_ops_add_topic).

Рекомендации о разделении данных по топикам см. в разделе [Внешняя таблица](../../overview/main_concepts/external_table/external_table.md).
{: .tip-wrapper}

Чтобы выгрузить данные из системы:
1. [Создайте](../../reference/sql_plus_requests/CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md) 
    [внешнюю таблицу](../../overview/main_concepts/external_table/external_table.md) 
    выгрузки, если она еще не создана.
2. Выполните запрос [INSERT INTO download_external_table](../../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md). 
   В запросе укажите внешнюю таблицу выгрузки, определяющую параметры выгрузки.

Созданные внешние таблицы выгрузки можно использовать повторно или удалить.

Данные выгружаются из следующей СУБД хранилища:
* из указанной или [наиболее оптимальной СУБД](../../../working_with_system/data_reading/routing/routing.md) —
  если данные выгружаются из логических таблиц, логических и
  материализованных представлений;
* из той СУБД хранилища, в которой размещаются связанные standalone-таблицы — если данные выгружаются из внешних
  readable-таблиц.

## Пример {#example}
```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание внешней таблицы для выгрузки из логической таблицы sales
CREATE DOWNLOAD EXTERNAL TABLE sales_ext_download (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;

-- запуск выгрузки данных из логической таблицы sales
INSERT INTO sales_ext_download 
SELECT * FROM sales WHERE product_units > 2 FOR SYSTEM_TIME AS OF DELTA_NUM 10;

-- создание внешней таблицы для выгрузки из материализованного представления sales_by_stores
CREATE DOWNLOAD EXTERNAL TABLE testdb_doc.sales_by_stores_ext_download (
store_id INT,
product_code VARCHAR(256),
product_units INT
)
LOCATION 'kafka://$kafka/sales_by_stores_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;

-- запуск выгрузки данных из материализованного представления sales_by_stores
INSERT INTO sales.sales_by_stores_ext_download
SELECT * FROM sales.sales_by_stores WHERE product_code IN ('ABC0002', 'ABC0003', 'ABC0004') DATASOURCE_TYPE = 'adqm';

-- создание внешней таблицы для выгрузки из внешней readable-таблицы payments_ext_read_adg
CREATE DOWNLOAD EXTERNAL TABLE sales.payments_ext_download (
  id INT NOT NULL,
  agreement_id INT,
  code VARCHAR(16),
  amount DOUBLE,
  currency_code VARCHAR(3),
  description VARCHAR,
  bucket_id INT NOT NULL,
)
LOCATION 'kafka://zk1:2181,zk2:2181,zk3:2181/agreements_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;

-- запуск выгрузки данных из внешней readable-таблицы payments_ext_read_adg
INSERT INTO sales.payments_ext_download
SELECT * FROM sales.payments_ext_read_adg WHERE code = 'MONTH_FEE' AND agreement_id BETWEEN 100 AND 150;
```