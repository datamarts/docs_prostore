---
layout: default
title: INSERT INTO download_external_table
nav_order: 35
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# INSERT INTO download_external_table
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос выгружает данные из [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md) 
в топик Kafka, указанный 
[при создании внешней таблицы выгрузки](../CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md) 
(download_external_table). Формат выгрузки описан в разделе
[Формат выгрузки данных](../../download_format/download_format.md).
{: .review-highlight}

Данные можно выгружать из следующих сущностей и их соединений:
{: .review-highlight}
* [логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md), 
* [логических представлений](../../../overview/main_concepts/logical_view/logical_view.md) и 
* [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md),
* [standalone-таблиц](../../../overview/main_concepts/standalone_table/standalone_table.md).
{: .review-highlight}

Данные standalone-таблицы выгружаются с помощью 
[внешней readable-таблицы](../../../overview/main_concepts/external_table/external_table.md#readable_table).
{: .review-highlight}

Для получения небольшого объема данных можно использовать 
[запрос данных](../../../working_with_system/data_reading/data_reading.md).
{: .note-wrapper}

Перед выполнением запроса необходимо создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md)
с указанием [пути к топику Kafka](../../path_to_kafka_topic/path_to_kafka_topic.md). Подробнее о порядке 
выполнения действий для выгрузки данных см. в разделе [Выгрузка данных](../../../working_with_system/data_download/data_download.md).
{: .note-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на выгрузку данных](../../../overview/interactions/download_processing/download_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Данные выгружаются из следующей СУБД [хранилища](../../../overview/main_concepts/data_storage/data_storage.md):
{: .review-highlight}
* [указанной в запросе](../../../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type) или 
  [наиболее оптимальной](../../../working_with_system/data_reading/routing/routing.md) —
  если данные выгружаются из логических таблиц, логических и материализованных представлений;
* содержащей standalone-таблицу — если данные выгружаются из standalone-таблицы или ее соединений с другими сущностями.
{: .review-highlight}

## Синтаксис {#syntax}

```sql
INSERT INTO [db_name.]ext_table_name SELECT query
```

**Параметры:**

`db_name`

: Имя логической базы данных, из которой выгружаются данные. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`ext_table_name`

: Имя внешней таблицы выгрузки.

`query`

: [SELECT](../SELECT/SELECT.md)-подзапрос для выбора выгружаемых данных.

## Ограничения {#restrictions}

Имена и порядок следования столбцов должны совпадать в SELECT-подзапросе на выгрузку данных и внешней таблице выгрузки.

## Примеры {#examples}

### Выгрузка из наиболее оптимальной СУБД {#default_db_example}

```sql
INSERT INTO sales.sales_ext_download
SELECT * FROM sales.sales WHERE sales.product_units > 2
```

### Выгрузка из указанной СУБД {#defined_db_example}

```sql
INSERT INTO sales.sales_ext_download 
SELECT * FROM sales.sales WHERE description = 'Покупка по акции 1+1' DATASOURCE_TYPE = 'adqm'
```

### Выгрузка из материализованного представления {#matview_example}

```sql
INSERT INTO sales.sales_by_stores_ext_download
SELECT * FROM sales.sales_by_stores WHERE product_code IN ('ABC0002', 'ABC0003', 'ABC0004') DATASOURCE_TYPE = 'adqm'
```

### Выгрузка из standalone-таблицы {#standalone_table_example}

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

-- выгрузка данных из standalone-таблицы, на которую указывает внешняя readable-таблица payments_ext_read_adg
INSERT INTO sales.payments_ext_download
SELECT s.id, s.agreement_id, s.code, s.amount, s.currency_code, s.description 
FROM sales.payments_ext_read_adg AS s 
WHERE code = 'MONTH_FEE' AND agreement_id BETWEEN 100 AND 150;
```