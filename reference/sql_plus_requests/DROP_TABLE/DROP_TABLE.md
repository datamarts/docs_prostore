﻿---
layout: default
title: DROP TABLE
nav_order: 25
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет удалить [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
и ее данные. 

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Для удаления данных логической таблицы только из некоторых [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md) можно указать
ключевое слово `DATASOURCE_TYPE` (см. секцию [Ключевое слово DATASOURCE_TYPE](#datasource_type)).

Удаленные данные невозможно восстановить средствами системы.
{: .warning-wrapper}

Если при обработке запроса произошла ошибка, изменение сущностей логической базы данных становится недоступно. В этом
случае нужно выполнить запрос [ERASE_CHANGE_OPERATION](../ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md).

Каждое удаление таблицы записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md).

## Синтаксис {#syntax}

```sql
DROP TABLE [IF EXISTS] [db_name.]table_name
[DATASOURCE_TYPE = datasource_alias]
[LOGICAL_ONLY]
```

**Параметры:** 

`db_name`

: Имя логической базы данных, из которой удаляется логическая таблица. 
  Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`table_name`

: Имя удаляемой логической таблицы.

`datasource_alias`

: Псевдоним СУБД хранилища, из которой удаляются данные логической таблицы. 
  Возможные значения: `adb`, `adqm`, `adg`, `adp`. Значение можно указывать без кавычек, в одинарных кавычках 
  (например, `'adb'`) или двойных кавычках (например, `"adb"`).
  <br>Если ключевое слово `DATASOURCE_TYPE` с псевдонимом не указано, данные удаляются из всех СУБД хранилища.

### Ключевое слово IF EXISTS {#if_exists}

Ключевое слово `IF EXISTS` включает проверку наличия логической таблицы до попытки ее удаления.
Если ключевое слово указано в запросе, система возвращает успешный ответ как по успешно
удаленной, так и несуществующей таблице; иначе — только по успешно удаленной таблице.

### Ключевое слово DATASOURCE_TYPE {#datasource_type}

Ключевое слово `DATASOURCE_TYPE` позволяет указать СУБД хранилища, из которых необходимо
удалить данные логической таблицы. Сама логическая таблица удаляется из
логической схемы данных при удалении данных таблицы из последней СУБД хранилища.

Если ключевое слово не указано, данные таблицы удаляются из всех СУБД хранилища.

### Ключевое слово LOGICAL_ONLY {#logical_only}

Ключевое слово `LOGICAL_ONLY` позволяет удалить логическую таблицу только на логическом уровне
(из [логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md)), без
удаления связанных [физических таблиц](../../../overview/main_concepts/physical_table/physical_table.md)
и размещенных в них данных из хранилища данных.

Если ключевое слово не указано, удаляется как логическая, так и связанные с ней физические таблицы.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Примеры {#examples}

### Удаление таблицы с удалением данных из всех СУБД {#all_db_example}

```sql
DROP TABLE marketing.sales
```

### Удаление таблицы и ее данных с проверкой наличия таблицы {#example_with_check}

```sql
DROP TABLE IF EXISTS marketing.sales_unknown_existence
```

### Удаление таблицы с удалением данных из ADB и ADG {#adb_adg_example}

```sql
-- удаление таблицы из ADB
DROP TABLE marketing.stores DATASOURCE_TYPE = adb;

-- удаление таблицы из ADG
DROP TABLE marketing.stores DATASOURCE_TYPE = adg;
```

### Удаление таблицы только на логическом уровне {#logical_example}

```sql
DROP TABLE marketing.sales1 LOGICAL_ONLY
```