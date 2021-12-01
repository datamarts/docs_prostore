---
layout: default
title: GET_DELTA_BY_NUM
nav_order: 29
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# GET_DELTA_BY_NUM

Запрос позволяет получить информацию о закрытой [дельте](../../../overview/main_concepts/delta/delta.md) 
по ее номеру. Перед выполнением запроса необходимо определить логическую базу данных, 
[используемую по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md), 
если она еще не определена.

В ответе возвращается:
*   объект ResultSet c одной записью, содержащей информацию о дельте, при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Возвращаемая информация включает следующие параметры:
*   `delta_num` — номер дельты,
*   `delta_date` — дата и время закрытия дельты,
*   `[cn_from, cn_to]` — диапазон номеров выполненных [операций записи](../../../overview/main_concepts/write_operation/write_operation.md).

## Синтаксис {#syntax}

```sql
GET_DELTA_BY_NUM(delta_num)
```

Параметры:
*   `delta_num` — целый неотрицательный номер дельты.
