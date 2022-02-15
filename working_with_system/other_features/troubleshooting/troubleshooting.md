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

В случае возникновения ошибок во время [загрузки](../../data_upload/data_upload.md) или 
[обновления данных](../../data_update/data_update.md) система отменяет 
[операции записи](../../../overview/main_concepts/write_operation/write_operation.md) (далее — операции), которые 
не удалось успешно завершить, и возвращает данные в состояние, предшествовавшее загрузке или обновлению. 
Основные причины ошибок см. в [секции ниже](#error_reasons).

При необходимости используйте запросы для управления процессами загрузки и обновления данных:
* [GET_WRITE_OPERATIONS](../../../reference/sql_plus_requests/GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md) — возвращает 
  информацию об операциях горячей [дельты](../../../overview/main_concepts/delta/delta.md), находящихся в статусах 
  «Выполняется» и «Ошибка»;
* [RESUME_WRITE_OPERATION](../../../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md) — 
  возобновляет обработку операций горячей дельты, находящихся в статусах «Выполняется» и «Ошибка»;
* [ROLLBACK CRASHED_WRITE_OPERATIONS](../../../reference/sql_plus_requests/ROLLBACK_CRASHED_WRITE_OPERATIONS/ROLLBACK_CRASHED_WRITE_OPERATIONS.md) — 
  возобновляет обработку операций горячей дельты, находящихся в статусе «Ошибка»;
* [ROLLBACK DELTA](../../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md) — отменяет операции 
  горячей дельты.

## Основные причины ошибок загрузки и обновления данных {#error_reasons}

Основные причины ошибок загрузки данных:
* некорректная схема или записи Avro в [сообщениях топика Kafka](../../../reference/upload_format/upload_format.md);
* несоответствие порядка, количества или типа полей между сообщениями Kafka, 
  [логической таблицей](../../../overview/main_concepts/logical_table/logical_table.md) или 
  [внешней таблицей](../../../overview/main_concepts/external_table/external_table.md) загрузки (кроме поля `sys_op`, 
  которое должно присутствовать в сообщениях, но должно отсутствовать в таблицах);
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
* несоответствие порядка, количества или типов столбцов между логической таблицей-приемником данных и запросом на 
обновление данных;
* отсутствие в запросе [INSERT VALUES](../../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md) или 
  [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) значений обязательных столбцов 
  логической таблицы;
* указание в запросе [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) тех 
  [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), для которых недоступна вставка данных. 