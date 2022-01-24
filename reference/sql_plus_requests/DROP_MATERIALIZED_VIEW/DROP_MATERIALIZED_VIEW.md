---
layout: default
title: DROP MATERIALIZED VIEW
nav_order: 23
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP MATERIALIZED VIEW
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет удалить [материализованное представление](../../../overview/main_concepts/materialized_view/materialized_view.md) 
и его данные. 

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Если при обработке запроса происходит ошибка, последующее изменение сущностей логической базы данных невозможно. В этом
случае нужно повторить запрос. Действие перезапустит обработку запроса, и после ее завершения можно будет продолжить
работу с логической БД.

Каждое удаление представления записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
можно посмотреть с помощью запроса [GET_CHANGES](../GET_CHANGES/GET_CHANGES.md).

## Синтаксис {#syntax}

```sql
DROP MATERIALIZED VIEW [IF EXISTS] [db_name.]materialized_view_name
[DATASOURCE_TYPE = datasource_alias]
[LOGICAL_ONLY]
```

Параметры:
* `db_name` — имя логической базы данных, из которой удаляется материализованное представление. 
   Опционально, если выбрана логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
* `table_name` — имя удаляемого материализованного представления;
* `datasource_alias` — псевдоним СУБД хранилища, из которой удаляются данные материализованного представления. 
  Возможные значения: `adqm`, `adg`. Значение можно указывать без кавычек, в одинарных кавычках 
  (например, `'adg'`) или двойных кавычках (например, `"adg"`). Если ключевое слово `DATASOURCE_TYPE` 
  с псевдонимом не указано, данные удаляются из всех СУБД хранилища.

### Ключевое слово IF EXISTS {#if_exists}

Ключевое слово `IF EXISTS` включает проверку наличия материализованного представления до попытки 
его удаления. Если ключевое слово указано в запросе, система возвращает успешный ответ как по успешно
удаленному, так и несуществующему представлению; иначе — только по успешно удаленному представлению.

### Ключевое слово DATASOURCE_TYPE {#datasource_type}

Ключевое слово `DATASOURCE_TYPE` позволяет указать [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md), из которых необходимо
удалить данные материализованного представления. В текущей версии данные представления могут размещаться в ADG и (или) 
ADQM. Само материализованное представление удаляется из логической схемы данных при удалении 
его данных из последней СУБД хранилища.

Если ключевое слово не указано, данные представления удаляются из всех СУБД хранилища.

### Ключевое слово LOGICAL_ONLY {#logical_only}

Ключевое слово `LOGICAL_ONLY` позволяет удалить материализованное представление только на логическом уровне
(из [логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md)), без
удаления связанных [физических таблиц](../../../overview/main_concepts/physical_table/physical_table.md)
и размещенных в них данных из хранилища данных.

Если ключевое слово не указано, удаляется как материализованное представление, так и связанные с ним 
физические таблицы.

## Ограничения {#restrictions}

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../../../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Примеры {#examples}

### Удаление представления с удалением данных из всех СУБД {#all_db_example}

```sql
DROP MATERIALIZED VIEW sales.sales_and_stores
```

### Удаление представления с проверкой его наличия {#example_with_check}

```sql
DROP MATERIALIZED VIEW IF EXISTS sales.mat_view_with_unknown_existence
```

### Удаление представления с удалением данных из ADG {#adg_example}

```sql
DROP MATERIALIZED VIEW sales.sales_and_stores DATASOURCE_TYPE = 'adg'
```

### Удаление представления только на логическом уровне {#logical_example}

```sql
DROP MATERIALIZED VIEW sales.stores_by_sold_products_matview LOGICAL_ONLY
```