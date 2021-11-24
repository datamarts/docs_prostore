---
layout: default
title: Удаление логического представления
nav_order: 7
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление логического представления {#drop_view}

Чтобы удалить [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) 
из [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [DROP VIEW](../../../reference/sql_plus_requests/DROP_VIEW/DROP_VIEW.md).
При успешном выполнении запроса логическое представление удаляется из 
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md). 
Удаление логического представления никак не отражается в [хранилище](../../../overview/main_concepts/data_storage/data_storage.md).

Наличие представления можно проверить, как описано в разделе 
[Проверка наличия логического представления](../entity_presence_check/entity_presence_check.md#view_check).

Каждое удаление представления записывается в 
[журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса `GET_CHANGES`. <ссылка>

## Пример {#examples}

```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- удаление представления stores_by_sold_products
DROP VIEW stores_by_sold_products;
```