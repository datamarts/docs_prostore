---
layout: default
title: USE INFORMATION_SCHEMA
nav_order: 45
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# USE INFORMATION_SCHEMA

Запрос позволяет изменить [логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md) 
[по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md) 
на [сервисную базу данных](../../../overview/main_concepts/service_db/service_db.md).

В ответе возвращается:
*   объект ResultSet c одной записью, содержащей имя сервисной базы данных (`information_schema`), 
    при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Если выполнить запрос перед [запросами к сервисной базе данных](../SELECT_FROM_INFORMATION_SCHEMA/SELECT_FROM_INFORMATION_SCHEMA.md),
это позволит не указывать в запросах префикс `INFORMATION_SCHEMA` перед именами системных
представлений.
{: .tip-wrapper}

## Синтаксис {#syntax}

```sql
USE INFORMATION_SCHEMA
```