---
layout: default
title: Управление операциями записи
nav_order: 1
parent: Управление операциями
grand_parent: Работа с системой
has_children: false
has_toc: false
---

# Управление операциями записи {#write_op_management}

Перезапускать и отменять незавершенные [операции записи](../../../overview/main_concepts/write_operation/write_operation.md) 
можно с помощью запросов, перечисленных в таблице ниже. Подробное описание 
запросов см. [в справочнике запросов](../../../reference/sql_plus_requests/sql_plus_requests.md).

Наличие незавершенных операций записи можно проверить с помощью запроса
[GET_WRITE_OPERATIONS](../../../reference/sql_plus_requests/GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).
{: .note-wrapper}

| Запрос | Операции <br>со статусом 0 <br>(«Выполняется») | Операции <br>со статусом 2 («Отменяется»)
|:-|:-|:-
| [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) | Перезапуск операции по [загрузке данных](../../data_upload/data_upload.md) | Отмена операции 
| [ROLLBACK CRASHED_WRITE_OPERATIONS](../../../reference/sql_plus_requests/ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md) | — | Отмена всех операций 
| Повторение исходного запроса с ключевым словом `RETRY` | Перезапуск операции по [обновлению данных](../../data_update/data_update.md) | —
| [ERASE_WRITE_OPERATION](../../../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md) | Отмена операции | Отмена операции
| [ROLLBACK DELTA](../../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md)* | Отмена всех операций по загрузке данных | Отмена всех операций

\* Запрос также отменяет все завершенные операции в горячей [дельте](../../../overview/main_concepts/delta/delta.md) 
(операции со статусами 1 и 3).

Подробнее о возможных статусах операций см. в разделе
[Операция записи](../../../overview/main_concepts/write_operation/write_operation.md#write_operation_statuses).
{: .note-wrapper}