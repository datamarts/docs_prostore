---
layout: default
title: Удаление логической таблицы
nav_order: 4
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление логической таблицы {#drop_table}

Чтобы удалить [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
и ее данные, выполните запрос [DROP TABLE](../../../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md).

* Если данные логической таблицы нужно удалить только из некоторых [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), добавьте в запрос ключевое слово
  [DATASOURCE_TYPE](../../../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md#datasource_type) с псевдонимами
  требуемых СУБД.
* Если логическую таблицу нужно удалить только на логическом уровне, без
  удаления связанных [физических таблиц](../../../overview/main_concepts/physical_table/physical_table.md) и 
  данных из хранилища, добавьте в запрос ключевое слово
  [LOGICAL_ONLY](../../../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md#logical_only).

Наличие логической таблицы можно проверить, как описано в разделе 
[Проверка наличия логической таблицы](../entity_presence_check/entity_presence_check.md#table_check). Наличие
физических таблиц, связанных с логической, можно проверить, как описано в разделе 
[Проверка месторасположения логической сущности](../../../working_with_system/other_features/datasource_check/datasource_check.md).

Каждое удаление таблицы записывается в 
[лог изменений логических сущностей](../../../overview/main_concepts/changelog/changelog.md). Лог изменений 
можно посмотреть с помощью запроса `GET_CHANGES`. <ссылка>

## Примеры {#examples}

### Удаление логической таблицы из одной СУБД {#adqm_example}

```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- удаление таблицы sales из СУБД ADQM
DROP TABLE sales DATASOURCE_TYPE = 'adqm';
```

### Удаление логической таблицы из всех СУБД {#all_db_example}

```sql
DROP TABLE sales.sales
```

### Удаление логической таблицы только на логическом уровне {#logical_deleting_table_example}

```sql
DROP TABLE sales.sales1 LOGICAL_ONLY
```