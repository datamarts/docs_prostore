---
layout: default
title: CREATE DOWNLOAD EXTERNAL TABLE
nav_order: 14
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE DOWNLOAD EXTERNAL TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет создать [внешнюю таблицу выгрузки](../../../overview/main_concepts/external_table/external_table.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

После успешного выполнения запроса можно выполнять запросы 
[INSERT INTO download_external_table](../INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md) на выгрузку данных. 
Подробнее о порядке выполнения действий для выгрузки данных см. в разделе 
[Выгрузка данных](../../../working_with_system/data_download/data_download.md).

Изменение внешней таблицы недоступно. Для замены внешней таблицы необходимо 
удалить ее и создать новую.
{: .note-wrapper}

## Синтаксис {#syntax}

```sql
CREATE DOWNLOAD EXTERNAL TABLE [db_name.]ext_table_name(
  column_name_1 datatype_1,
  column_name_2 datatype_2,
  column_name_3 datatype_3
)
LOCATION receiver_URI
FORMAT 'AVRO'
[CHUNK_SIZE records_per_message]
```

Параметры:
*   `db_name` — имя логической базы данных, в которой создается внешняя таблица. Указывается опционально, 
    если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `ext_table_name` — имя создаваемой внешней таблицы выгрузки. Для удобства различения таблиц выгрузки 
    и таблиц загрузки рекомендуется задавать имя таблицы, указывающее на ее тип, например 
    `sales_ext_download`;
*   `column_name_N` — имя столбца таблицы;
*   `datatype_N` — тип данных столбца `column_name_N`. Возможные значения см. в разделе 
    [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md);
*   `receiver_URI` — путь к топику Kafka 
    (см. [Формат пути к внешнему приемнику данных](../../path_to_kafka_topic/path_to_kafka_topic.md));
*   `records_per_message` — максимальное количество записей, выгружаемых из хранилища в одном сообщении 
    топика Каfka.
    
## Ограничения {#restrictions}

Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Пример {#examples}

```sql
CREATE DOWNLOAD EXTERNAL TABLE sales.sales_ext_download (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000
```