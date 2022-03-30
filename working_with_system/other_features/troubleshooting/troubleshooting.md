---
layout: default
title: Разбор ошибок загрузки и обновления данных
nav_order: 2
parent: Другие действия
grand_parent: Работа с системой
has_children: false
has_toc: false
---

# Разбор ошибок загрузки и обновления данных {#upload_troubleshooting}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Раздел описывает причины основных ошибок, возникающих во время [загрузки](../../data_upload/data_upload.md) и
[обновления данных](../../data_update/data_update.md). 
<br>При загрузке и обновлении данных в [standalone-таблицах](../../../overview/main_concepts/standalone_table/standalone_table.md) 
также могут возникать другие ошибки, связанные с ограничениями конкретной СУБД.
При разборе ошибок в СУБД следуйте рекомендациям, доступным в документации соответствующей СУБД.
{: .note-wrapper}

При возникновении ошибок во время загрузки или обновления данных в 
[логических таблицах](../../../overview/main_concepts/logical_table/logical_table.md) система отменяет 
[операции записи](../../../overview/main_concepts/write_operation/write_operation.md) (далее — операции), которые 
не удалось успешно завершить, и возвращает данные в состояние, предшествовавшее загрузке или обновлению. 
При ошибках загрузки и обновления данных в standalone-таблицах изменения не отменяются автоматически, их нужно 
отменять вручную в соответствующей СУБД.

Основные причины ошибок см. в [секции ниже](#error_reasons).

## Управление загрузкой и обновлением данных в логических таблицах {#error_managing}

Процессами загрузки и обновления данных в горячей [дельте](../../../overview/main_concepts/delta/delta.md) можно управлять 
с помощью следующих запросов:
* [GET_WRITE_OPERATIONS](../../../reference/sql_plus_requests/GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md) — возвращает 
  информацию о незавершенных операциях записи;
* [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) —
  возобновляет обработку одной или всех* незавершенных операций записи;
* [ERASE_WRITE_OPERATION](../../../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md) — отменяет 
  незавершенную операцию записи;
* [ROLLBACK CRASHED_WRITE_OPERATIONS](../../../reference/sql_plus_requests/ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md) —
  возобновляет обработку одной или всех незавершенных операций записи со статусом «Отменяется»;
* [ROLLBACK DELTA](../../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md) — отменяет все** операции
  записи в горячей дельте.

\* Кроме операций со статусом «Выполняется», запущенных запросами
[обновления данных](../../../working_with_system/data_update/data_update.md).<br>
\*\* Кроме операций со статусами «Выполняется» и «Отменяется», запущенных запросами обновления данных.

Подробнее о возможных статусах операций см. в разделе 
[Операция записи](../../../overview/main_concepts/write_operation/write_operation.md#write_operation_statuses).
{: .note-wrapper}

## Основные причины ошибок загрузки и обновления данных {#error_reasons}

Основные причины ошибок загрузки данных:
* некорректная схема или записи Avro в [сообщениях топика Kafka](../../../reference/upload_format/upload_format.md);
* несоответствие порядка, количества или типа полей между сообщениями Kafka, 
  [логической таблицей](../../../overview/main_concepts/logical_table/logical_table.md) или 
  [внешней таблицей](../../../overview/main_concepts/external_table/external_table.md) загрузки (кроме поля `sys_op`, 
  которое должно присутствовать в сообщениях, загружаемых в логические таблицы, но должно отсутствовать в таблицах);
* некорректный [путь к топику Kafka](../../../reference/path_to_kafka_topic/path_to_kafka_topic.md) в настройках 
  внешней таблицы загрузки;
* недостаточная продолжительность одного или нескольких интервалов ожидания, заданных в
  [конфигурации системы](../../../maintenance/configuration/system/system.md) и используемых при работе с брокером 
  сообщений Kafka;
* некорректные настройки сервиса мониторинга статусов Kafka в конфигурации системы;
* расхождения времени между серверами инсталляции;
* некорректная установка коннектора, предназначенного для загрузки данных.

Интервалы ожидания при работе с брокером сообщений Kafka настраиваются с помощью параметров конфигурации 
`EDML_FIRST_OFFSET_TIMEOUT_MS` и `EDML_CHANGE_OFFSET_TIMEOUT_MS`, а также параметра `ADB_MPPW_FDW_TIMEOUT_MS`, который 
используется только для ADB.
{: .note-wrapper}

Основные причины ошибок обновления данных:
* несоответствие порядка, количества или типов столбцов между таблицей-приемником данных и запросом на 
обновление данных;
* отсутствие в запросе [INSERT VALUES](../../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md), 
  [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) или 
  [UPSERT VALUES](../../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) значений обязательных столбцов 
  логической таблицы;
* указание в запросе [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) тех 
  [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), для которых недоступна вставка данных. 