---
layout: default
title: CREATE UPLOAD EXTERNAL TABLE
nav_order: 17
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE UPLOAD EXTERNAL TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md) 
загрузки в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

После успешного выполнения запроса можно выполнять запросы 
[INSERT INTO logical_table](../INSERT_INTO_logical_table/INSERT_INTO_logical_table.md) на загрузку данных. Подробнее о порядке 
выполнения действий для загрузки данных см. в разделе 
[Загрузка данных](../../../working_with_system/data_upload/data_upload.md).

Изменение внешней таблицы недоступно. Для замены внешней таблицы необходимо 
удалить ее и создать новую.
{: .note-wrapper}

## Синтаксис {#syntax}

```sql
CREATE UPLOAD EXTERNAL TABLE [db_name.]ext_table_name (
  column_name_1 datatype_1,
  column_name_2 datatype_2,
  column_name_3 datatype_3
)
LOCATION source_URI
FORMAT 'AVRO'
[MESSAGE_LIMIT messages_per_segment]
```

Параметры:
*   `db_name` — имя логической базы данных, в которой создается внешняя таблица. Указание опционально, 
    если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `ext_table_name` — имя создаваемой внешней таблицы загрузки. Для удобства различения таблиц выгрузки 
    и таблиц загрузки рекомендуется задавать имя таблицы, указывающее на ее тип, например 
    `sales_ext_upload`;
*   `column_name_N` — имя столбца таблицы;
*   `datatype_N` — тип данных столбца `column_name_N`. Возможные значения см. в разделе 
    [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md);
*   `source_URI` — путь к топику Kafka 
    (см. [Формат пути к внешнему приемнику данных](../../path_to_kafka_topic/path_to_kafka_topic.md));
*   `messages_per_segment` — максимальное количество сообщений, загружаемых 
    в [хранилище](../../../overview/main_concepts/data_storage/data_storage.md) 
    в составе одного блока на один поток загрузки. Значение подбирается в зависимости от параметров 
    производительности инфраструктуры.

## Ограничения {#restrictions}

Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Пример {#examples}

```sql
CREATE UPLOAD EXTERNAL TABLE sales.sales_ext_upload (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales'
FORMAT 'AVRO'
MESSAGE_LIMIT 1000
```