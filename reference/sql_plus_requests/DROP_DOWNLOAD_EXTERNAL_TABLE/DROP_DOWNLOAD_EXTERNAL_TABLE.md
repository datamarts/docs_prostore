---
layout: default
title: DROP DOWNLOAD EXTERNAL TABLE
nav_order: 22
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP DOWNLOAD EXTERNAL TABLE

Запрос позволяет удалить [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md) 
выгрузки из [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

## Синтаксис {#syntax}

```sql
DROP DOWNLOAD EXTERNAL TABLE [db_name.]ext_table_name
```

Параметры:
*   `db_name` — имя логической базы данных, из которой удаляется внешняя таблица выгрузки. Указывается 
    опционально, если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `ext_table_name` — имя удаляемой внешней таблицы выгрузки.

## Ограничения {#restrictions}

Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Пример {#examples}

```sql
DROP DOWNLOAD EXTERNAL TABLE sales.sales_ext_download
```