---
layout: default
title: Удаление внешней таблицы загрузки
nav_order: 11
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление внешней таблицы загрузки {#drop_upload_table}

Чтобы удалить [внешнюю таблицу загрузки](../../../overview/main_concepts/external_table/external_table.md), 
выполните запрос [DROP UPLOAD EXTERNAL TABLE](../../../reference/sql_plus_requests/DROP_UPLOAD_EXTERNAL_TABLE/DROP_UPLOAD_EXTERNAL_TABLE.md). 
При успешном выполнении запроса внешняя таблица удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md).

Наличие внешней таблицы можно проверить, как описано в разделе [Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
-- выбор базы данных marketing по умолчанию
USE marketing;

-- удаление внешней таблицы загрузки
DROP UPLOAD EXTERNAL TABLE sales_ext_upload;
```