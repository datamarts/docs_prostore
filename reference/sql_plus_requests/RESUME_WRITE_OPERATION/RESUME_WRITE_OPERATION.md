---
layout: default
title: RESUME_WRITE_OPERATION
nav_order: 39
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# RESUME_WRITE_OPERATION

Запрос возобновляет обработку незавершенных [операций записи](../../../overview/main_concepts/write_operation/write_operation.md) 
горячей [дельты](../../../overview/main_concepts/delta/delta.md). 

Под незавершенными операциями понимаются операции со статусами «Выполняется» и «Отменяется». Возможные статусы операций
см. в разделе [Операция записи](../../../overview/main_concepts/write_operation/write_operation.md#write_operation_statuses).

Возобновить обработку можно для одной или всех незавершенных операций горячей дельты.
Перед выполнением запроса необходимо определить
[логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md),
[используемую по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md),
если она еще не определена.

Запрос `RESUME_WRITE_OPERATION` не возобновляет обработку операций со статусом «Выполняется», запущенных запросами
[обновления данных](../../../working_with_system/data_update/data_update.md). Способы обработки операций в зависимости
от их типа см. в разделе
[Управление операциями записи](../../../working_with_system/operation_management/write_op_management/write_op_management.md).
{: .note-wrapper}

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса или отсутствии незавершенных операций записи.

При успешном выполнении запроса:
* запускается отмена операции — если операция находится в статусе «Отменяется», 
* возобновляется отслеживание загрузки данных в [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md) — если операция находится в статусе «Выполняется».

Аналогичный процесс по возобновлению операций автоматически выполняется 
[при рестарте системы](../../../overview/interactions/restart_processing/restart_processing.md).

## Синтаксис {#syntax}

Возобновление обработки одной незавершенной операции:
```sql
RESUME_WRITE_OPERATION(sys_cn)
```

Возобновление обработки всех незавершенных операций:
```sql
RESUME_WRITE_OPERATION()
```

**Параметры:**

`sys_cn`

: Номер операции записи, обработку которой нужно возобновить. Если номер 
  не указан, возобновляется обработка всех незавершенных операций, которые есть в горячей дельте 
  [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md) (кроме операций по обновлению данных 
  в статусе «Выполняется»).

Номер операции можно узнать с помощью запроса 
[GET_WRITE_OPERATIONS](../../sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md).
{: .tip-wrapper}

## Пример {#examples}

```sql
RESUME_WRITE_OPERATION(14)
```