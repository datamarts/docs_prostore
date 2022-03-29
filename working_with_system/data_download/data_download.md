---
layout: default
title: Выгрузка данных
nav_order: 5
parent: Работа с системой
has_children: false
---

# Выгрузка данных {#data_download}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Система позволяет выгружать большие объемы данных, а также изменений, выполненных 
в указанных дельтах. Данные можно выгружать из следующих сущностей и их соединений: 
* [логических таблиц](../../overview/main_concepts/logical_table/logical_table.md), 
* [логических представлений](../../overview/main_concepts/logical_view/logical_view.md),
* [материализованных представлений](../../overview/main_concepts/materialized_view/materialized_view.md),
* [standalone-таблиц](../../overview/main_concepts/standalone_table/standalone_table.md).

Под большим объемом данных подразумевается количество записей от нескольких сотен до нескольких миллионов.
Для получения небольшого объема данных можно использовать функцию [запроса данных](../data_reading/data_reading.md).
{: .note-wrapper}

Данные выгружаются из системы в виде [сообщений Kafka](../../reference/upload_format/upload_format.md).
Поэтому, если в брокере сообщений Kafka не настроено автоматическое создание топиков, нужно создать топики вручную.
Чтобы создать топик, следуйте любой из инструкций в документации Kafka:
* [Quick Start](https://kafka.apache.org/documentation/#quickstart),
* [Adding and removing topics](https://kafka.apache.org/documentation/#basic_ops_add_topic).

Рекомендации о разделении данных по топикам см. в разделе 
[Внешняя таблица](../../overview/main_concepts/external_table/external_table.md#topic_recommendations).
{: .tip-wrapper}

Чтобы выгрузить данные из системы:
1. [Создайте](../../reference/sql_plus_requests/CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md) 
   [внешнюю таблицу выгрузки](../../overview/main_concepts/external_table/external_table.md#download_table), 
   если она еще не создана.
2. Выполните запрос [INSERT INTO download_external_table](../../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md). 
   В запросе укажите внешнюю таблицу выгрузки, определяющую параметры выгрузки.

Созданные внешние таблицы выгрузки можно использовать повторно или удалить.

Данные выгружаются из следующей СУБД хранилища:
* [указанной в запросе](../../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type) или 
  [наиболее оптимальной](../../working_with_system/data_reading/routing/routing.md) —
  если данные выгружаются из логических таблиц, логических и
  материализованных представлений;
* содержащей standalone-таблицу — если данные выгружаются из standalone-таблицы или ее соединений с другими сущностями.

## Примеры {#example}

### Выгрузка из логической таблицы {#from_logical_table}

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
```

### Выгрузка из материализованного представления {#from_matview}

```sql
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
```

### Выгрузка из standalone-таблицы {#from_standalone_table}

```sql
-- создание внешней таблицы выгрузки
CREATE DOWNLOAD EXTERNAL TABLE sales.payments_ext_download (
id INT NOT NULL,
agreement_id INT,
code VARCHAR(16),
amount DOUBLE,
currency_code VARCHAR(3),
description VARCHAR
)
LOCATION 'kafka://$kafka/agreements_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;

-- запуск выгрузки данных из standalone-таблицы, на которую указывает внешняя readable-таблица payments_ext_read_adg
INSERT INTO sales.payments_ext_download
SELECT s.id, s.agreement_id, s.code, s.amount, s.currency_code, s.description
FROM sales.payments_ext_read_adg AS s
WHERE code = 'MONTH_FEE' AND agreement_id BETWEEN 100 AND 150;
```