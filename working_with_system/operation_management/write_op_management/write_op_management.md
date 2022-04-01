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

При ошибках [загрузки](../../data_upload/data_upload.md) и
[обновления данных](../../data_update/data_update.md) 
в [логических таблицах](../../../overview/main_concepts/logical_table/logical_table.md) 
система автоматически отменяет неуспешные 
[операции записи](../../../overview/main_concepts/write_operation/write_operation.md) и возвращает данные СУБД 
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md)
в состояние, которое предшествовало загрузке или обновлению. Однако, если произошел сбой, например из-за неполадок
в сети, операция может остаться незавершенной (в статусе «Выполняется» или «Отменяется»). 
В этом случае нужно вручную перезапустить или отменить операцию.

<!--
Незавершенная операция имеет статус «Выполняется» или «Отменяется».
Для операций записи со статусом «Отменяется» перезапуск и отмена означают один и тот же результат: отмену операции. 
Для операций записи со статусом «Выполняется» перезапуск означает возобновление обработки, отмена — отмену операции.

Подробнее о возможных статусах операций см. в разделе 
[Операция записи](../../../overview/main_concepts/write_operation/write_operation.md#write_operation_statuses).
{: .note-wrapper}

### Перезапуск операции записи {#write_op_retry}

Чтобы перезапустить операцию со статусом «Выполняется»:
* чтобы перезапустить операцию по загрузке данных, выполните запрос 
  [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md);
* чтобы перезапустить операцию по обновлению данных, повторите исходный запрос, добавив в его начало ключевое слово `RETRY`.

### Отмена операции записи {#write_op_cancel}

* Чтобы отменить операцию со статусом «Выполняется», выполните запрос 
  [ERASE_WRITE_OPERATION](../../../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md) или 
  [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) 
  с указанием номера операции.
* Чтобы отменить операцию со статусом «Отменяется», выполните запрос 
  [ERASE_WRITE_OPERATION](../../../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md).

### Массовые перезапуск и отмена операций {#all_write_op}

Чтобы массово перезапустить или отменить операции записи, используйте следующие запросы:
* [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) 
  без указания номера операции — перезапускает операции по загрузке данных в статусе «Выполняется» 
  и отменяет все операции в статусе «Отменяется»; 
* [ROLLBACK CRASHED_WRITE_OPERATIONS](../../../reference/sql_plus_requests/ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md) — 
  отменяет все виды операций в статусе «Отменяется»;
* [ROLLBACK DELTA](../../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md) — отменяет все незавершенные и 
  завершенные операции записи в горячей дельте, кроме операций по обновлению данных в статусе «Выполняется».
-->

<!--

## Отмена операции по изменению логической схемы данных {#change_operations}

Чтобы отменить операцию по изменению логической схемы, выполните запрос 
[ERASE_CHANGE_OPERATION](../../../reference/sql_plus_requests/ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md).
-->

Подробнее о возможных статусах операций см. в разделе
[Операция записи](../../../overview/main_concepts/write_operation/write_operation.md#write_operation_statuses).
{: .note-wrapper}

Перезапускать и отменять операции записи можно с помощью запросов, перечисленных в таблице ниже. Подробное описание 
запросов см. [в справочнике запросов](../../../reference/sql_plus_requests/sql_plus_requests.md).

| Запрос | Операции <br>со статусом 0 <br>(«Выполняется») | Операции <br>со статусом 2 («Отменяется»)
|:-|:-|:-
| [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) | Перезапуск одной или всех операций по загрузке данных | Отмена одной или всех операций 
| [ROLLBACK CRASHED_WRITE_OPERATIONS](../../../reference/sql_plus_requests/ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md) | — | Отмена всех операций 
| Повторение исходного запроса с ключевым словом `RETRY` | Перезапуск одной операции по обновлению данных | —
| [ERASE_WRITE_OPERATION](../../../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md) | Отмена одной операции | Отмена одной операции
| [ROLLBACK DELTA](../../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md)* | Отмена всех операций по загрузке данных | Отмена всех операций

\* Запрос также отменяет все завершенные операции в горячей [дельте](../../../overview/main_concepts/delta/delta.md) 
(операции со статусами 1 и 3).