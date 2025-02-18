﻿---
layout: default
title: CREATE UPLOAD EXTERNAL TABLE
nav_order: 18
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

По умолчанию создается таблица, которая предназначена для загрузки данных в 
[логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) и содержит скрытое 
служебное поле `sys_op`. Если загружаемые данные не содержат поле `sys_op` (например, они предназначены для
[standalone-таблицы](../../../overview/main_concepts/standalone_table/standalone_table.md)), 
нужно указать в запросе ключевое слово `OPTIONS` со значением `auto.create.sys_op.enable=false`.

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

После успешного выполнения запроса можно выполнять 
[запросы на загрузку данных](../INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md). 
Подробнее о действиях по загрузке данных см. в разделе 
[Загрузка данных](../../../working_with_system/data_upload/data_upload.md).

Изменение внешней таблицы недоступно. Для замены внешней таблицы необходимо удалить ее и создать новую.
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
[OPTIONS 'option_list')]
```

**Параметры:**

`db_name`

: Имя логической базы данных, в которой создается внешняя таблица. Опционально, 
  если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`ext_table_name`

: Имя создаваемой внешней таблицы. Должно быть уникально среди всех логических
  сущностей логической БД, а также должно удовлетворять другим условиям, перечисленным в секции [Ограничения](#restrictions). 
  Чтобы можно было различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип 
  таблицы, например `sales_ext_upload`.

`column_name_N`

: Имя столбца таблицы. Должно удовлетворять условиям, перечисленным в секции [Ограничения](#restrictions).

`datatype_N`

: Тип данных столбца `column_name_N`. Возможные значения см. в разделе 
  [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md).

`source_URI`

: Путь к топику Kafka (см. [Формат пути к топику Kafka](../../path_to_kafka_topic/path_to_kafka_topic.md)).

`messages_per_segment`

: Максимальное количество сообщений, загружаемых 
  в [хранилище](../../../overview/main_concepts/data_storage/data_storage.md) 
  в составе одного блока на один поток загрузки. Значение подбирается в зависимости от параметров 
  производительности инфраструктуры.

`option_list`

: Список дополнительных параметров и их значений в формате `option1=value1;option2=value2...`.
   Возможные параметры:
  * `auto.create.sys_op.enable` — признак добавления скрытого служебного поля `sys_op` во внешнюю таблицу, 
    возможные значения: `true` (по умолчанию) — добавить поле и использовать таблицу для загрузки данных в логическую таблицу,
    `false` — не добавлять поле и использовать таблицу для загрузки данных в standalone-таблицу.

## Ограничения {#restrictions}

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Имена таблицы и ее столбцов должны начинаться с латинской буквы, после первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания в любом порядке.
* Таблица и ее столбцы не могут иметь имена, перечисленные в разделе [Зарезервированные слова](../../reserved_words/reserved_words.md).

## Примеры {#examples}

### Таблица для загрузки в логическую таблицу {#to_logical_table}

```sql
CREATE UPLOAD EXTERNAL TABLE marketing.sales_ext_upload (
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

### Таблица для загрузки в standalone-таблицу {#to_standalone_table}

```sql
CREATE UPLOAD EXTERNAL TABLE marketing.payments_ext_upload (
  id INT NOT NULL,
  agreement_id INT,
  code VARCHAR(16),
  amount DOUBLE,
  currency_code VARCHAR(3),
  description VARCHAR
)
LOCATION  'kafka://$kafka/payments'
FORMAT 'AVRO'
OPTIONS ('auto.create.sys_op.enable=false')
```