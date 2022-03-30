---
layout: default
title: Удаление внешней writable-таблицы
nav_order: 12.6
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление внешней writable-таблицы {#drop_writable_table}

Чтобы удалить [внешнюю writable-таблицу](../../../overview/main_concepts/external_table/external_table.md#writable_table), 
выполните запрос [DROP WRITABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/DROP_WRITABLE_EXTERNAL_TABLE/DROP_WRITABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана 
опция `auto.drop.table.enable=true`, из СУБД хранилища удаляется связанная 
[standalone-таблица](../../../overview/main_concepts/standalone_table/standalone_table.md).

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Примеры {#examples}

### Удаление внешней таблицы с удалением standalone-таблицы {#example_with_options}

```sql
DROP WRITABLE EXTERNAL TABLE sales.agreements_ext_write_adp
OPTIONS ('auto.drop.table.enable=true')
```

### Удаление внешней таблицы без удаления standalone-таблицы {#example_without_options}

```sql
DROP WRITABLE EXTERNAL TABLE sales.payments_ext_write_adg
```