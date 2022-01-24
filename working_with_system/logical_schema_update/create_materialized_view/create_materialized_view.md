---
layout: default
title: Создание материализованного представления
nav_order: 8
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Создание материализованного представления {#create_materialized_view}

Чтобы создать [материализованное представление](../../../overview/main_concepts/materialized_view/materialized_view.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE MATERIALIZED VIEW](../../../reference/sql_plus_requests/CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md).
Если нужно создать представление только на логическом уровне, добавьте в запрос ключевое слово
[LOGICAL_ONLY](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#logical_only).

Создание материализованных представлений возможно в ADG и ADQM на основе данных ADB.
{: .note-wrapper}

Создание представления недоступно при наличии любого из факторов:
* горячей [дельты](../../../overview/main_concepts/delta/delta.md),
* незавершенного запроса на создание, удаление или изменение таблицы или представления,
* запрета на изменение сущностей (см. раздел [DENY_CHANGES](../../../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).

Наличие представления можно проверить, как описано в разделе 
[Проверка наличия материализованного представления](../entity_presence_check/entity_presence_check.md#mat_view_check).
Наличие физических таблиц, связанных с материализованным представлением, можно проверить, как описано в разделе 
[Проверка месторасположения логической сущности](../../../working_with_system/other_features/datasource_check/datasource_check.md).

Каждое создание представления записывается в 
[журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса [GET_CHANGES](../../../reference/sql_plus_requests/GET_CHANGES/GET_CHANGES.md).

## Примеры {#examples}

### Создание материализованного представления {#non-logical_example}

```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- создание материализованного представления sales_and_stores
CREATE MATERIALIZED VIEW sales.sales_and_stores (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  description VARCHAR(256),
  store_id INT NOT NULL,
  store_category VARCHAR(256) NOT NULL,
  region VARCHAR(256) NOT NULL,
  PRIMARY KEY (id, region)
)
DISTRIBUTED BY (id)
DATASOURCE_TYPE (adg, adqm)
AS SELECT
 s.id, s.transaction_date, s.product_code, s.product_units, s.description,
 st.id AS store_id, st.category as store_category, st.region
 FROM sales.sales AS s
 JOIN sales.stores AS st
 ON s.store_id = st.id
DATASOURCE_TYPE = 'adb';
```

### Создание материализованного представления только на логическом уровне {#logical_example}

```sql
CREATE MATERIALIZED VIEW sales.stores_by_sold_products_matview (
  store_id INT NOT NULL,
  product_amount INT NOT NULL,
  PRIMARY KEY (store_id)
)
DISTRIBUTED BY (store_id)
DATASOURCE_TYPE (adg)
AS SELECT store_id, SUM(product_units) AS product_amount
  FROM sales.sales
  GROUP BY store_id
DATASOURCE_TYPE = 'adb'
LOGICAL_ONLY
```
