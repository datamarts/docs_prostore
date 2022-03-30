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

Если операция имеет статус «Отменяется», то запускается отмена этой операции; если операция имеет 
статус «Выполняется», то возобновляется отслеживание загрузки данных в [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md).
Аналогичный процесс автоматически выполняется для всех незавершенных операций 
[при рестарте системы](../../../overview/interactions/restart_processing/restart_processing.md).

Запрос `RESUME_WRITE_OPERATION` не возобновляет обработку операций со статусом «Выполняется», запущенных запросами 
[обновления данных](../../../working_with_system/data_update/data_update.md). Каждую такую операцию можно возобновить, 
повторив исходный запрос обновления данных с ключевым словом `RETRY`, или отменить запросом 
[ERASE_WRITE_OPERATION](../ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md).
{: .note-wrapper}

Возобновить обработку можно для одной или всех незавершенных операций горячей дельты.
Перед выполнением запроса необходимо определить 
[логическую базу данных](../../../overview/main_concepts/logical_db/logical_db.md),
[используемую по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md),
если она еще не определена.

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса или отсутствии незавершенных операций записи.

Если нужно возобновить обработку только для операций со статусом «Отменяется», можно использовать запрос 
[ROLLBACK CRASHED_WRITE_OPERATIONS](../ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md).
{: .tip-wrapper}

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
  [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md).
  <br>Номер операции можно узнать с помощью запроса
  [GET_WRITE_OPERATIONS](../../sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md).

## Пример {#examples}

```sql
RESUME_WRITE_OPERATION(14)
```