---
layout: default
title: Изменение логического представления
nav_order: 6
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Изменение логического представления {#alter_view}

Чтобы изменить [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) 
в [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [ALTER VIEW](../../../reference/sql_plus_requests/ALTER_VIEW/ALTER_VIEW.md) 
или `CREATE OR REPLACE VIEW` (см. [CREATE VIEW](../../../reference/sql_plus_requests/CREATE_VIEW/CREATE_VIEW.md)).
При успешном выполнении запроса логическое представление изменит свой вид.

Изменение представления недоступно при наличии любого из факторов:
* горячей [дельты](../../../overview/main_concepts/delta/delta.md),
* незавершенного запроса на создание, удаление или изменение таблицы или представления,
* запрета на изменение сущностей (см. раздел [DENY_CHANGES](../../../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).

Каждое изменение представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса [GET_CHANGES](../../../reference/sql_plus_requests/GET_CHANGES/GET_CHANGES.md).

## Примеры {#examples}

### Создание логического представления {#creating_example}

```sql
-- выбор sales как логической базы данных по умолчанию
USE sales;

-- создание логического представления
CREATE VIEW stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales
  GROUP BY store_id
  ORDER BY product_amount DESC
  LIMIT 10;
```

### Изменение логического представления {#altering_example}

```sql
ALTER VIEW stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales
  GROUP BY store_id
  ORDER BY product_amount ASC
  LIMIT 20
```

### Пересоздание логического представления {#recreation_example}

```sql
CREATE OR REPLACE VIEW stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales
  GROUP BY store_id
  ORDER BY product_amount DESC
  LIMIT 30
```