---
layout: default
title: Удаление внешней writable-таблицы
nav_order: 12.6
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление внешней writable-таблицы {#drop_writable_table}

Чтобы удалить внешнюю writable-таблицу, 
выполните запрос [DROP WRITABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/DROP_WRITABLE_EXTERNAL_TABLE/DROP_WRITABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана 
опция `auto.drop.table.enable=true`, то связанная standalone-таблица также удаляется.

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
DROP WRITABLE EXTERNAL TABLE sales.agreements_ext_write
OPTIONS ('auto.drop.table.enable=true')
```