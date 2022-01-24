---
layout: default
title: DROP DATABASE
nav_order: 21
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP DATABASE

Запрос позволяет удалить [логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md) 
и все ее данные.

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

При удалении логической базы данных без ключевого слова `LOGICAL_ONLY` удаляются **все** ее 
данные и вся история изменений этих данных. Удаленные данные невозможно восстановить средствами системы.
{: .note-wrapper}

## Синтаксис {#syntax}

Удаление логической БД:

```sql
DROP DATABASE db_name
```

Удаление логической БД только на логическом уровне:

```sql
DROP DATABASE db_name LOGICAL_ONLY
```

Параметры:
*   `db_name` — имя удаляемой логической базы данных.

### Ключевое слово LOGICAL_ONLY {#logical_only}

Ключевое слово `LOGICAL_ONLY` позволяет удалить логическую базу данных и все ее дочерние логические сущности 
только на логическом уровне (из [логической схемы данных](../../../overview/main_concepts/logical_schema/logical_schema.md)),  
без удаления связанной физической базы данных и размещенных в ней данных из 
[хранилища данных](../../../overview/main_concepts/data_storage/data_storage.md).

Если ключевое слово не указано, удаляется как логическая, так и связанная с ней физическая база данных.

## Ограничения {#restrictions}

Недоступно удаление сервисной базы данных `INFORMATION_SCHEMA`.

## Примеры {#examples}

### Удаление логической БД {#non-logical_example}

```sql
DROP DATABASE sales
```

### Удаление логической БД только на логическом уровне {#logical_example}

```sql
DROP DATABASE sales1 LOGICAL_ONLY
```