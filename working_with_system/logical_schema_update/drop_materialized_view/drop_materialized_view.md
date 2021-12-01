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
При необходимости добавьте в запрос ключевое слово:
* [DATASOURCE_TYPE](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#datasource_type) со списком 
  [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md) — чтобы указать, из каких СУБД нужно удалить 
  данные представления;
* [LOGICAL_ONLY](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#logical_only) — чтобы удалить 
  представление только на логическом уровне.

Удаление представления недоступно при наличии любого из факторов:
* горячей [дельты](../../../overview/main_concepts/delta/delta.md),
* незавершенного запроса на создание, удаление или изменение таблицы или представления,
* запрета на изменение сущностей (см. раздел [DENY_CHANGES](../../../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).

Наличие представления можно проверить, как описано в разделе
[Проверка наличия материализованного представления](../entity_presence_check/entity_presence_check.md#mat_view_check).
Наличие физических таблиц, связанных с материализованным представлением, можно проверить, как описано в разделе 
[Проверка месторасположения логической сущности](../../../working_with_system/other_features/datasource_check/datasource_check.md).
{: .note-wrapper}

Каждое удаление представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса [GET_CHANGES](../../../reference/sql_plus_requests/GET_CHANGES/GET_CHANGES.md).

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