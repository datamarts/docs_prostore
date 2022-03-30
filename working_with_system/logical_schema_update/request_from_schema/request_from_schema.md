---
layout: default
title: Запрос метаданных логической схемы
nav_order: 15
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Запрос метаданных логической схемы {#request_from_schema}

Чтобы запросить метаданные объектов [логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md), 
выполните запрос [SELECT FROM INFORMATION_SCHEMA](../../../reference/sql_plus_requests/SELECT_FROM_INFORMATION_SCHEMA/SELECT_FROM_INFORMATION_SCHEMA.md). 
Доступно получение информации о сущностях и их свойствах, перечисленных 
в разделе [Системные представления (INFORMATION_SCHEMA)](../../../reference/system_views/system_views.md).

## Примеры {#examples}

### Запрос списка логических БД окружения {#db_list_example}

```sql
SELECT schema_name FROM INFORMATION_SCHEMA.schemata
ORDER BY schema_name
```

### Запрос списка сущностей в логической БД {#entity_list_example}

```sql
SELECT * FROM INFORMATION_SCHEMA.tables
WHERE table_schema = 'MARKETING'
```

### Запрос списка столбцов сущностей в логической БД {#column_list_example}

```sql
SELECT TC.table_schema, TC.table_name, TT.table_type, TC.column_name
FROM information_schema.columns AS TC
JOIN information_schema.tables AS TT
  ON TC.table_schema = TT.table_schema and TC.table_name = TT.table_name  
WHERE TT.table_schema = 'MARKETING'   
ORDER BY TC.table_schema, TC.table_name
```