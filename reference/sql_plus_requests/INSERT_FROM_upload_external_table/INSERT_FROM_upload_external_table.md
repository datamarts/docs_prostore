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

Запрос позволяет загрузить данные из внешнего источника данных в [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
или внешнюю writable-таблицу.
Загружаемые данные должны соответствовать [формату загрузки данных](../../upload_format/upload_format.md).

Перед выполнением запроса необходимо создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md)
и загрузить данные в топик Kafka.
Подробнее о порядке выполнения действий для загрузки данных см. в разделе
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
* в те СУБД, которые выбраны для размещения данных таблицы — если данные загружаются в логическую таблицу;
* в ту СУБД, которая содержит таблицу — если данные загружаются в standalone-таблицу.

Месторасположение данных логической таблицы можно задавать запросами
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с ключевым словом
`DATASOURCE_TYPE`.

## Синтаксис {#syntax}

Запрос с явным перечислением столбцов внешней таблицы:
```sql
INSERT INTO [db_name.]table_name SELECT column_list FROM [db_name.]ext_table_name
```

Запрос с использованием символа `*`:
```sql
INSERT INTO [db_name.]table_name SELECT * FROM [db_name.]ext_table_name
```

Параметры:
*   `db_name` — имя логической базы данных. Опционально, если выбрана логическая БД, 
    [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `table_name` — имя логической таблицы или внешней writable-таблицы, в которую загружаются данные;
*   `column_list` — список имен всех столбцов внешней таблицы загрузки. 
    Имена и порядок следования указанных столбцов должны совпадать с именами и порядком следования столбцов (полей) 
    в таблице, куда загружаются данные, и топике Kafka, из которого загружаются данные <>;
*   `ext_table_name` — имя внешней таблицы загрузки.

В сообщениях Kafka, загружаемых в логические таблицы, последним полем обязательно указывается служебное поле `sys_op`. 
Соответствующий столбец `sys_op` отсутствует в логической таблице и внешней таблице загрузки, однако имена и порядок 
всех остальных столбцов (полей) должны совпадать во всех трех объектах: в сообщении, логической и внешней таблицах 
(см. раздел [Формат загрузки данных](../../upload_format/upload_format.md)).
{: .note-wrapper}

## Ограничения {#restrictions}

Загрузка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).

## Пример {#examples}

Пример загрузки данных в логическую таблицу:
```sql
-- открытие новой (горячей) дельты
BEGIN DELTA;

-- запуск загрузки данных в логическую таблицу sales
INSERT INTO sales.sales SELECT * FROM sales.sales_ext_upload;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```