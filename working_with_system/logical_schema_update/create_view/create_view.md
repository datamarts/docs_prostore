---
layout: default
title: Создание логического представления
nav_order: 5
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Создание логического представления {#download_format}

Чтобы создать [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) 
в [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE VIEW](../../../reference/sql_plus_requests/CREATE_VIEW/CREATE_VIEW.md). 
При успешном выполнении запроса логическое представление появляется в 
[логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md).

Наличие представления можно проверить, как описано в разделе 
[Проверка наличия логического представления](../entity_presence_check/entity_presence_check.md#view_check).

Каждое создание представления записывается в 
[журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса `GET_CHANGES`. <ссылка>

## Пример {#examples}

```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- создание представления stores_by_sold_products
CREATE VIEW stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales.sales
  GROUP BY store_id
  ORDER BY product_amount DESC
  LIMIT 30;
```