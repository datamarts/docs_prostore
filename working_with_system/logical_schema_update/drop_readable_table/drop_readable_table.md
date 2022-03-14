---
layout: default
title: Удаление внешней readable-таблицы
nav_order: 12.4
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление внешней readable-таблицы {#drop_readable_table}

Чтобы удалить внешнюю readable-таблицу, 
выполните запрос [DROP READABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/DROP_READABLE_EXTERNAL_TABLE/DROP_READABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана 
опция `auto.drop.table.enable=true`, то связанная standalone-таблица также удаляется.

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
DROP READABLE EXTERNAL TABLE sales.agreements_ext_read_adp
OPTIONS ('auto.drop.table.enable=true')
```