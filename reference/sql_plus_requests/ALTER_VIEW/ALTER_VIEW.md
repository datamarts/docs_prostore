---
layout: default
title: ALTER VIEW
nav_order: 2
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# ALTER VIEW

Запрос позволяет изменить вид [логического представления](../../../overview/main_concepts/logical_view/logical_view.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md).

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Логическое представление можно также изменить с помощью запроса `CREATE OR REPLACE VIEW` 
(см. [CREATE VIEW](../CREATE_VIEW/CREATE_VIEW.md)).
{: .note-wrapper}

Если при обработке запроса произошла ошибка, изменение сущностей логической базы данных становится недоступно. В этом
случае нужно выполнить запрос [ERASE_CHANGE_OPERATION](../ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md).

Каждое изменение представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md).

## Синтаксис {#syntax}

```sql
ALTER VIEW [db_name.]view_name AS SELECT query
```

**Параметры:**

`db_name`

: Имя логической базы данных, в которой находится логическое представление. 
  Опционально, если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`view_name`

: Имя изменяемого логического представления.

`query`

: [SELECT](../SELECT/SELECT.md)\-подзапрос, на основе которого строится новый вид 
  логического представления.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Подзапрос `query` не может содержать:
  * логические и [материализованные представления](../../../overview/main_concepts/materialized_view/materialized_view.md),
  * [системные представления](../../system_views/system_views.md) `INFORMATION_SCHEMA`,
  * ключевое слово [FOR SYSTEM_TIME](../SELECT/SELECT.md#for_system_time).
* Ключевое слово `DATASOURCE_TYPE`, указанное в подзапросе `query`, игнорируется.

## Пример {#examples}

```sql
ALTER VIEW marketing.stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM marketing.sales
  GROUP BY store_id
  ORDER BY product_amount ASC
LIMIT 20
```
