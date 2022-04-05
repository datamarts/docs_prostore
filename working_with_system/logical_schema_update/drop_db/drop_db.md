---
layout: default
title: Удаление логической базы данных
nav_order: 2
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Удаление логической базы данных {#drop_db}

Чтобы удалить [логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md) и ее данные, 
выполните запрос [DROP DATABASE](../../../reference/sql_plus_requests/DROP_DATABASE/DROP_DATABASE.md) 
(см. примеры [ниже](#examples)). Если нужно удалить логическую базу данных только на логическом уровне, без
удаления связанной физической базы данных и размещенных в ней данных из 
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md),
добавьте в запрос ключевое слово
[LOGICAL_ONLY](../../../reference/sql_plus_requests/DROP_DATABASE/DROP_DATABASE.md#logical_only).

Наличие логической базы данных можно проверить, как описано в разделе 
[Проверка наличия логической базы данных](../entity_presence_check/entity_presence_check.md#db_check).

## Примеры {#examples}

### Удаление логической базы данных {#deleting_db_example}

``` sql
DROP DATABASE marketing
```

### Удаление логической базы данных только на логическом уровне {#logical_deleting_db_example}

``` sql
DROP DATABASE marketing1 LOGICAL_ONLY
```