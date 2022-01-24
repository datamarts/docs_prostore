---
layout: default
title: CREATE VIEW
nav_order: 18
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE VIEW

Запрос позволяет создать или заменить [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md). Логическое представление 
можно создать на основе данных одной или нескольких [логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md).

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Если при обработке запроса происходит ошибка, последующее изменение сущностей логической базы данных невозможно. В этом 
случае нужно повторить запрос. Действие перезапустит обработку запроса, и после ее завершения можно будет продолжить 
работу с логической БД.

Каждое создание представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md).

## Синтаксис {#syntax}

Создание логического представления:
```sql
CREATE VIEW [db_name.]view_name AS SELECT query
```

Создание логического представления с заменой существующего представления, если такое будет найдено:
```sql
CREATE OR REPLACE VIEW [db_name.]view_name AS SELECT query
```

Параметры:
* `db_name` — имя логической базы данных, в которой создается или заменяется логическое представление. 
  Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
* `view_name` — имя создаваемого или заменяемого логического представления. В запросе на создание 
  представления имя должно быть уникально среди логических сущностей логической БД;
* `query` — [SELECT](../SELECT/SELECT.md)-подзапрос, на основе которого строится логическое представление.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* В подзапросе `query` не допускается использование:
  * логических представлений,
  * [системных представлений](../../system_views/system_views.md) `INFORMATION_SCHEMA`,
  * ключевого слова [FOR SYSTEM_TIME](../SELECT/SELECT.md#for_system_time).
* Ключевое слово `DATASOURCE_TYPE`, указанное в подзапросе `query`, игнорируется.

## Пример {#examples}

```sql
CREATE VIEW sales.stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales.sales
  GROUP BY store_id
  ORDER BY product_amount DESC
  LIMIT 20
```