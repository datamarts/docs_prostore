---
layout: default
title: Создание логической таблицы
nav_order: 3
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Создание логической таблицы {#create_table}

Чтобы создать [логическую таблицу](../../../overview/main_concepts/logical_table/logical_table.md) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE TABLE](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md).
При необходимости добавьте в запрос ключевое слово:
* [DATASOURCE_TYPE](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#datasource_type) со списком
  [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md) — чтобы разместить данные таблицы только в 
  некоторых СУБД хранилища;
* [LOGICAL_ONLY](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#logical_only) — чтобы создать таблицу
  только на логическом уровне.

Создание таблицы недоступно при наличии любого из факторов:
* горячей [дельты](../../../overview/main_concepts/delta/delta.md),
* незавершенного запроса на создание, удаление или изменение таблицы или представления,
* запрета на изменение сущностей (см. раздел [DENY_CHANGES](../../../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).

Наличие логической таблицы можно проверить, как описано в разделе 
[Проверка наличия логической таблицы](../entity_presence_check/entity_presence_check.md#table_check). Наличие 
физических таблиц, связанных с логической, можно проверить, как описано в разделе
[Проверка месторасположения логической сущности](../../../working_with_system/other_features/datasource_check/datasource_check.md).
{: .note-wrapper}

Каждое создание таблицы записывается в [журнал](../../../overview/main_concepts/changelog/changelog.md). Журнал 
доступен с помощью запроса [GET_CHANGES](../../../reference/sql_plus_requests/GET_CHANGES/GET_CHANGES.md).

## Примеры {#examples}

### Создание логической таблицы {#non-logical_example}

```sql
-- выбор базы данных sales по умолчанию
USE sales;

-- создание таблицы sales
CREATE TABLE sales (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id);
```

### Создание логической таблицы только на логическом уровне {#logical_example}

```sql
CREATE TABLE sales.sales1 (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id)
LOGICAL_ONLY
```

### Создание логической таблицы с размещением данных в ADQM и ADG {#adqm_adg_example}

```sql
CREATE TABLE sales.clients (
  id INT NOT NULL,
  first_name VARCHAR(256) NOT NULL,
  last_name VARCHAR(256) NOT NULL,
  patronymic_name VARCHAR(256),
  birth_date DATE,
  PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adqm,adg)
```