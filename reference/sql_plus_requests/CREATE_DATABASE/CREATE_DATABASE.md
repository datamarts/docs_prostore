﻿---
layout: default
title: CREATE DATABASE
nav_order: 14
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE DATABASE

Запрос позволяет создать [логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md) 
в текущем [окружении](../../../overview/main_concepts/environment/environment.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Перед работой с логической базой данных выберите ее в качестве [используемой по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md) 
— это позволит обращаться к логическим сущностям без имени логической БД.
{: .tip-wrapper}

## Синтаксис {#syntax}

Создание логической БД:

```sql
CREATE DATABASE db_name
```

Создание логической БД только на логическом уровне:

```sql
CREATE DATABASE db_name LOGICAL_ONLY
```

**Параметры:**

`db_name`

: Имя создаваемой логической базы данных. Может содержать латинские буквы, цифры и символы подчеркивания.

### Ключевое слово LOGICAL_ONLY {#logical_only}

Ключевое слово `LOGICAL_ONLY` позволяет создать логическую базу данных только на логическом уровне
(в [логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md)), без 
пересоздания связанной физической базы данных в [хранилище данных](../../../overview/main_concepts/data_storage/data_storage.md).

Если ключевое слово не указано, создается как логическая, так и связанная с ней физическая база данных.

## Ограничения {#restrictions}

* Имя логической базы данных должно начинаться с латинской буквы, после первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания в любом порядке.
* Логическая БД не может иметь имя `INFORMATION_SCHEMA`, а также имена, перечисленные в разделе 
[Зарезервированные слова](../../reserved_words/reserved_words.md). 

## Примеры {#examples}

### Создание логической БД {#non-logical_example}

```sql
CREATE DATABASE marketing
```

### Создание логической БД только на логическом уровне {#logical_example}

```sql
CREATE DATABASE marketing1 LOGICAL_ONLY
```