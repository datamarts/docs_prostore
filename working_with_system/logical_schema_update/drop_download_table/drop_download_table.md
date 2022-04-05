---
layout: default
title: Удаление внешней таблицы выгрузки
nav_order: 13
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление внешней таблицы выгрузки {#drop_download_table}

Чтобы удалить [внешнюю таблицу выгрузки](../../../overview/main_concepts/external_table/external_table.md), 
выполните запрос [DROP DOWNLOAD EXTERNAL TABLE](../../../reference/sql_plus_requests/DROP_DOWNLOAD_EXTERNAL_TABLE/DROP_DOWNLOAD_EXTERNAL_TABLE.md). 
При успешном выполнении запроса внешняя таблица выгрузки удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md).

Наличие внешней таблицы можно проверить, как описано в разделе [Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
-- выбор базы данных marketing по умолчанию
USE marketing;

-- удаление внешней таблицы выгрузки
DROP DOWNLOAD EXTERNAL TABLE sales_ext_download;
```