---
layout: default
title: Причины ошибок при загрузке и обновлении данных
nav_order: 4.5
parent: Справочная информация
has_children: false
has_toc: false
---

# Причины ошибок при загрузке и обновлении данных {#upload_troubleshooting}

Раздел описывает причины основных ошибок, возникающих во время вставки данных — [загрузки](../../working_with_system/data_upload/data_upload.md) и
[обновления данных](../../working_with_system/data_update/data_update.md).

При загрузке и обновлении данных в [standalone-таблицах](../../overview/main_concepts/standalone_table/standalone_table.md) 
также могут возникать ошибки, связанные с ограничениями конкретной СУБД.
При разборе ошибок в СУБД следуйте рекомендациям, доступным в документации соответствующей СУБД.
{: .note-wrapper}

Если ошибка возникает во время вставки данных в 
[логические таблицы](../../overview/main_concepts/logical_table/logical_table.md), система отменяет 
[операции записи](../../overview/main_concepts/write_operation/write_operation.md), которые 
не удалось успешно завершить, и возвращает данные в состояние, предшествовавшее этим операциям. 
При ошибках вставки данных в standalone-таблицах изменения не отменяются автоматически,
и их нужно отменять вручную в соответствующей СУБД.

Способы управления операциями записи см. в разделе 
[Управление операциями записи](../../working_with_system/operation_management/write_op_management/write_op_management.md).
{: .tip-wrapper}

## Основные причины ошибок при загрузке данных {#upload_errors}

* Некорректная схема или записи Avro в [сообщениях топика Kafka](../upload_format/upload_format.md).
* Несоответствие порядка, количества или типа полей между сообщениями Kafka, 
  таблицей-приемником данных или 
  [внешней таблицей загрузки](../../overview/main_concepts/external_table/external_table.md#upload_table) (кроме поля `sys_op`, 
  которое должно присутствовать в сообщениях Kafka, загружаемых в логические таблицы, но должно отсутствовать в таблицах).
* Некорректный [путь к топику Kafka](../path_to_kafka_topic/path_to_kafka_topic.md) в свойствах 
  внешней таблицы загрузки.
* Недостаточная продолжительность одного или нескольких интервалов ожидания, которые заданы в
  [конфигурации системы](../../maintenance/configuration/system/system.md) и используются при работе с брокером 
  сообщений Kafka.
* Некорректные настройки сервиса мониторинга статусов Kafka в конфигурации системы.
* Расхождения времени между серверами инсталляции.
* Некорректная установка коннектора, предназначенного для загрузки данных.

Интервалы ожидания при работе с брокером сообщений Kafka настраиваются с помощью параметров конфигурации 
`EDML_FIRST_OFFSET_TIMEOUT_MS` и `EDML_CHANGE_OFFSET_TIMEOUT_MS`, а также параметра `ADB_MPPW_FDW_TIMEOUT_MS`, который 
используется только для ADB.
{: .note-wrapper}

## Основные причины ошибок при обновлении данных {#update_errors}

* Несоответствие порядка, количества или типов столбцов между таблицей-приемником данных и запросом на 
  обновление данных.
* Отсутствие в запросе [INSERT VALUES](../sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md), 
  [INSERT SELECT](../sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) или 
  [UPSERT VALUES](../sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) значений обязательных столбцов 
  таблицы-приемника данных.
* Указание в запросе [INSERT SELECT](../sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) тех 
  [СУБД](../../introduction/supported_DBMS/supported_DBMS.md) 
  [хранилища](../../overview/main_concepts/data_storage/data_storage.md), для которых недоступна вставка данных. 