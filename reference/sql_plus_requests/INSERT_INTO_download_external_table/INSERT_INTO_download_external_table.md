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

Запрос позволяет выгрузить данные, выбранные [SELECT](../SELECT/SELECT.md)-подзапросом 
к [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
во внешний приемник данных. Данные можно выгружать из следующих сущностей:
* [логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md), 
* [логических представлений](../../../overview/main_concepts/logical_view/logical_view.md) и 
* [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md),
* внешних readable-таблиц,
* соединения перечисленных сущностей.

Для получения небольшого объема данных можно использовать 
[запрос данных](../../../working_with_system/data_reading/data_reading.md).
{: .note-wrapper}

Перед выполнением запроса необходимо создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md)
с указанием [пути к внешнему приемнику данных](../../path_to_kafka_topic/path_to_kafka_topic.md). Подробнее о порядке 
выполнения действий для выгрузки данных см. в разделе [Выгрузка данных](../../../working_with_system/data_download/data_download.md).
{: .note-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на выгрузку данных](../../../overview/interactions/download_processing/download_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

При успешном выполнении запроса данные выгружаются в тот приемник данных, который был указан 
[при создании внешней таблицы выгрузки](../CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md). Описание 
формата данных см. в разделе [Формат выгрузки данных](../../download_format/download_format.md).

Данные выгружаются из следующей СУБД хранилища:
* из [указанной](../../../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type) или 
  [наиболее оптимальной СУБД](../../../working_with_system/data_reading/routing/routing.md) —
  если данные выгружаются из логических таблиц, логических и материализованных представлений;
* из той СУБД хранилища, в которой размещаются связанные standalone-таблицы — если данные выгружаются из внешних
  readable-таблиц.

## Синтаксис {#syntax}

```sql
INSERT INTO [db_name.]ext_table_name SELECT query
```

Параметры:
*   `db_name` — имя логической базы данных, из которой выгружаются данные. Параметр опционален, если выбрана 
    логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `ext_table_name` — имя внешней таблицы выгрузки;
*   `query` — [SELECT](../SELECT/SELECT.md)-подзапрос для выбора выгружаемых данных.

## Ограничения {#restrictions}

Имена и порядок следования столбцов должны совпадать в SELECT-подзапросе на выгрузку данных и внешней таблице выгрузки.

## Пример {#examples}

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

### Выгрузка из внешней readable-таблицы {#readable_table_example}

```sql
-- создание внешней таблицы выгрузки
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

-- запуск выгрузки данных из внешней readable-таблицы
INSERT INTO sales.payments_ext_download
SELECT * FROM sales.payments_ext_read_adg WHERE code = 'MONTH_FEE' AND agreement_id BETWEEN 100 AND 150;
```