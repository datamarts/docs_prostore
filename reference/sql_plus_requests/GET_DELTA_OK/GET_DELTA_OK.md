---
layout: default
title: GET_DELTA_OK
nav_order: 31
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# GET_DELTA_OK

Запрос позволяет получить информацию о последней закрытой [дельте](../../../overview/main_concepts/delta/delta.md). 
Перед выполнением запроса необходимо определить логическую базу данных, 
[используемую по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md), 
если она еще не определена.

В ответе возвращается:
*   объект ResultSet c одной записью при успешном выполнении запроса. Если дельта присутствует, запись 
    содержит информацию о дельте, иначе — возвращается пустая запись;
*   исключение при неуспешном выполнении запроса.

Возвращаемая информация включает следующие параметры:
*   `delta_num` — номер дельты,
*   `delta_date` — дата и время закрытия,
*   `[cn_from, cn_to]` — диапазон номеров выполненных [операций записи](../../../overview/main_concepts/write_operation/write_operation.md).

## Синтаксис {#syntax}

```sql
GET_DELTA_OK()
```