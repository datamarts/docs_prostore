---
layout: default
title: Создание логической базы данных
nav_order: 1
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Создание логической базы данных {#create_db}

Чтобы создать [логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE DATABASE](../../../reference/sql_plus_requests/CREATE_DATABASE/CREATE_DATABASE.md) 
(см. примеры [ниже](#creating_db_example)). Если логическую базу данных нужно создать только на логическом уровне, без
пересоздания связанной физической базы данных в [хранилище данных](../../../overview/main_concepts/data_storage/data_storage.md),
добавьте в запрос ключевое слово 
[LOGICAL_ONLY](../../../reference/sql_plus_requests/CREATE_DATABASE/CREATE_DATABASE.md#logical_only).
  
Наличие логической базы данных можно проверить, как описано в разделе [Проверка наличия логической базы данных](../entity_presence_check/entity_presence_check.md#db_check).

Для удобства написания последующих запросов к этой логической базе данных можно выбрать 
ее в качестве [используемой по умолчанию](../../other_features/default_db_set-up/default_db_set-up.md).
{: .tip-wrapper}

## Примеры {#examples}

### Создание логической базы данных {#creating_db_example}

``` sql
CREATE DATABASE marketing
```

### Создание логической базы данных только на логическом уровне {#logical_creating_db_example}

``` sql
CREATE DATABASE marketing1 LOGICAL_ONLY
```

### Выбор логической БД по умолчанию {#using_db_example}

``` sql
USE marketing
```