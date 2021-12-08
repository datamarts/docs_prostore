---
layout: default
title: Выгрузка данных
nav_order: 5
parent: Работа с системой
has_children: false
---

# Выгрузка данных {#data_download}

Система позволяет выгружать большие объемы данных, а также изменений, выполненных 
в указанных дельтах. Данные можно выгружать из [логических таблиц](../../overview/main_concepts/logical_table/logical_table.md)
и [логических представлений](../../overview/main_concepts/logical_view/logical_view.md). Выгрузка данных из
[материализованных представлений](../../overview/main_concepts/materialized_view/materialized_view.md) недоступна.

Под большим объемом данных подразумевается количество записей от нескольких сотен до нескольких миллионов.
Для получения небольшого объема данных можно использовать функцию [запроса данных](../data_reading/data_reading.md).
{: .note-wrapper}

Система определяет СУБД хранилища, [оптимальную для выгрузки данных](../data_reading/routing/routing.md), 
в зависимости от параметров запроса, месторасположения данных и конфигурации системы. Возможные способы выбора данных к выгрузке см. в секции 
[FOR SYSTEM_TIME](../../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time)
раздела [SELECT](../../reference/sql_plus_requests/SELECT/SELECT.md).

Данные выгружаются [в виде сообщений Kafka](../../reference/download_format/download_format.md), поэтому для их загрузки
нужен топик Kafka. Если в брокере сообщений Kafka настроено автоматическое создание топиков, 
то дополнительные действия не требуются. Иначе топик необходимо создать, если он отсутствует. Подробнее о создании 
топиков см. в документации Kafka:
*   раздел [Quick Start](https://kafka.apache.org/documentation/#quickstart),
*   раздел [Adding and removing topics](https://kafka.apache.org/documentation/#basic_ops_add_topic).

Чтобы выгрузить данные из таблицы или представления во внешнюю информационную систему:
1.  [Создайте](../../reference/sql_plus_requests/CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md) 
    [внешнюю таблицу](../../overview/main_concepts/external_table/external_table.md) 
    выгрузки, если она еще не создана.
2.  Выполните запрос [INSERT INTO download_external_table](../../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md) 
    на выгрузку данных в топик. В запросе нужно указать внешнюю таблицу выгрузки, определяющую параметры 
    выгрузки.
3.  Выгрузите данные из топика во внешнюю информационную систему.

Созданные внешние таблицы выгрузки можно использовать повторно или удалить.

## Пример {#example}
```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- создание внешней таблицы выгрузки sales_ext_download
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

-- создание внешней таблицы выгрузки stores_ext_download
CREATE DOWNLOAD EXTERNAL TABLE sales.stores_ext_download (
id INT NOT NULL,
category VARCHAR(256) NOT NULL,
region VARCHAR(256) NOT NULL,
address VARCHAR(256) NOT NULL,
description VARCHAR(256)
) 
LOCATION  'kafka://$kafka/stores_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;

-- запуск выгрузки данных из логической таблицы stores
INSERT INTO stores_ext_download 
SELECT * FROM stores WHERE region = 'Москва' DATASOURCE_TYPE = 'adqm';
```