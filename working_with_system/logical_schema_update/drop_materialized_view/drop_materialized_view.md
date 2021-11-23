---
layout: default
title: Удаление материализованного представления
nav_order: 9
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление материализованного представления {#drop_materialized_view}

Чтобы удалить [материализованное представление](../../../overview/main_concepts/materialized_view/materialized_view.md) 
и его данные, выполните запрос [DROP MATERIALIZED VIEW](../../../reference/sql_plus_requests/DROP_MATERIALIZED_VIEW/DROP_MATERIALIZED_VIEW.md).
Если материализованное представление нужно удалить только на логическом уровне, без
удаления связанных [физических таблиц](../../../overview/main_concepts/physical_table/physical_table.md) и данных 
из [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), добавьте в запрос ключевое слово
[LOGICAL_ONLY](../../../reference/sql_plus_requests/DROP_MATERIALIZED_VIEW/DROP_MATERIALIZED_VIEW.md#logical_only).

Наличие представления можно проверить, как описано в разделе
[Проверка наличия материализованного представления](../entity_presence_check/entity_presence_check.md#mat_view_check).
Наличие физических таблиц, связанных с материализованным представлением, можно проверить, как описано в разделе 
[Проверка месторасположения логической сущности](../../../working_with_system/other_features/datasource_check/datasource_check.md).

Каждое удаление представления записывается в 
[лог изменений логических сущностей](../../../overview/main_concepts/changelog/changelog.md). Лог изменений 
можно посмотреть с помощью запроса `GET_CHANGES`. <ссылка>

## Примеры {#examples}

### Удаление материализованного представления из одной СУБД {#adg_example}
```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- удаление представления sales_and_stores
DROP MATERIALIZED VIEW sales_and_stores DATASOURCE_TYPE = 'adg';
```

### Удаление материализованного представления из всех СУБД {#all_db_example}

```sql
DROP MATERIALIZED VIEW sales.sales_and_stores
```

### Удаление материализованного представления только на логическом уровне {#logical_example}

```sql
DROP MATERIALIZED VIEW sales.stores_by_sold_products_matview LOGICAL_ONLY
```