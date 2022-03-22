---
layout: default
title: INSERT FROM upload_external_table
nav_order: 34.5
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# INSERT FROM upload_external_table
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос загружает данные из топика Kafka в [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
или [standalone-таблицу](../../../overview/main_concepts/standalone_table/standalone_table.md).
Загружаемые данные должны соответствовать [формату загрузки данных](../../upload_format/upload_format.md). 
При загрузке данных в standalone-таблицу нужно учитывать ее ограничения в конкретной СУБД.
{: .review-highlight}

Перед выполнением запроса необходимо создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md), 
если она отсутствует, и загрузить данные в топик Kafka. Подробнее о действия по загрузке данных см. в разделе 
[Загрузка данных](../../../working_with_system/data_upload/data_upload.md).

Загрузка данных в [логические](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованные представления](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступна.
{: .note-wrapper}

Для загрузки небольшого объема данных можно использовать 
[обновление данных](../../../working_with_system/data_update/data_update.md).
{: .note-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на загрузку данных](../../../overview/interactions/upload_processing/upload_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

При успешном выполнении запроса данные загружаются в следующие СУБД [хранилища](../../../overview/main_concepts/data_storage/data_storage.md): 
{: .review-highlight}
* в те СУБД, которые выбраны для размещения данных таблицы — если данные загружаются в логическую таблицу;
* в ту СУБД, которая содержит standalone-таблицу — если данные загружаются в standalone-таблицу.
{: .review-highlight}

Расположение данных логической таблицы можно задавать запросами
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с ключевым словом
`DATASOURCE_TYPE`.

При загрузке данных в standalone-таблицу ADG, в SELECT-подзапросе нужно указать поле `bucket_id` со значением `0` 
(см. пример [ниже](#ex_writable_adg)). В этом случае значение `bucket_id` рассчитается в ADG.
{: .note-wrapper .review-highlight}

## Синтаксис {#syntax}

Запрос с явным перечислением столбцов внешней таблицы загрузки:
```sql
INSERT INTO [db_name.]table_name SELECT column_list FROM [db_name.]upload_ext_table_name
```

Запрос с использованием символа `*`:
```sql
INSERT INTO [db_name.]table_name SELECT * FROM [db_name.]upload_ext_table_name
```

**Параметры:**

`db_name`

: Имя логической базы данных. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`table_name`

: Имя таблицы, в которую загружаются данные. Возможные значения:
  * имя логической таблицы,
  * имя [внешней writable-таблицы](../../../overview/main_concepts/external_table/external_table.md#writable_table), 
    указывающей на нужную standalone-таблицу.
{: .review-highlight}

`column_list`

: Список имен всех столбцов внешней таблицы загрузки. 
  Имена и порядок следования указанных столбцов должны совпадать с именами и порядком следования столбцов (полей) 
  в таблице, куда загружаются данные, и топике Kafka, из которого загружаются данные.

`upload_ext_table_name`

: Имя внешней таблицы загрузки.

В сообщениях Kafka, загружаемых в логические таблицы, последним полем обязательно указывается служебное поле `sys_op`. 
Соответствующий столбец `sys_op` отсутствует в логической таблице и внешней таблице загрузки, однако имена и порядок 
всех остальных столбцов (полей) должны совпадать во всех трех объектах: в сообщении, логической и внешней таблицах 
(см. раздел [Формат загрузки данных](../../upload_format/upload_format.md)).
{: .note-wrapper}

## Ограничения {#restrictions}

Загрузка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).

## Примеры {#examples}

### Загрузка данных в логическую таблицу {#logical_table_example}

```sql
-- открытие новой (горячей) дельты
BEGIN DELTA;

-- запуск загрузки данных в логическую таблицу sales
INSERT INTO sales.sales SELECT * FROM sales.sales_ext_upload;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Загрузка данных в standalone-таблицу через внешнюю writable-таблицу {#writable_table_example}

Загрузка в standalone-таблицу ADP, на которую указывает внешняя writable-таблица `agreements_ext_write_adp`:

```sql
INSERT INTO sales.agreements_ext_write_adp SELECT * FROM sales.agreements_ext_upload;
```

<a id="ex_writable_adg"></a>
Загрузка в standalone-таблицу ADG, на которую указывает внешняя writable-таблица `payments_ext_write_adg`:

```sql
INSERT INTO sales.payments_ext_write_adg SELECT *, 0 as bucket_id FROM sales.payments_ext_upload;
```