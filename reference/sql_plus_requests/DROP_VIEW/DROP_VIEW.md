---
layout: default
title: DROP VIEW
nav_order: 27
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP VIEW

Запрос позволяет удалить [логическое представление](../../../overview/main_concepts/logical_view/logical_view.md) 
из [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md).
При успешном выполнении запроса логическое представление удаляется из
[логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md).

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Если при обработке запроса произошла ошибка, изменение сущностей логической базы данных становится недоступно. В этом
случае нужно выполнить запрос [ERASE_CHANGE_OPERATION](../ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md).

Каждое удаление представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md).

## Синтаксис {#syntax}

```sql
DROP VIEW [db_name.]view_name
```

**Параметры:**

`db_name`

: Имя логической базы данных, из которой удаляется логическое представление. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`view_name`

: Имя удаляемого логического представления.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Пример {#examples}

```sql
DROP VIEW marketing.stores_by_sold_products
```