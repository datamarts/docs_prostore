---
layout: default
title: Standalone-таблица
nav_order: 6.5
parent: Основные понятия
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Standalone-таблица {#standalone_table}

_Standalone-таблица_ — таблица СУБД хранилища, расположенная вне [физической схемы данных](../physical_schema/physical_schema.md).

Данные standalone-таблиц можно читать и записывать через систему, используя 
единый [синтаксис SQL+](../../../reference/sql_plus_requests/sql_plus_requests.md) для всех СУБД. 
Чтение и запись в такие таблицы выполняются через проекции (внешние таблицы): 
чтение — через [внешние readable-таблицы](../external_table/external_table.md#readable_table), 
запись — через [внешние writable-таблицы](../external_table/external_table.md#writable_table).

Внешняя таблица может быть связана с существующей standalone-таблицей. Если standalone-таблица отсутствует, 
она может быть автоматически создана при создании внешней таблицы.

Подробнее об создании внешних таблиц см. в разделах 
[CREATE WRITABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_WRITEABLE_EXTERNAL_TABLE/CREATE_WRITEABLE_EXTERNAL_TABLE.md) и 
[CREATE READABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md).
{: .note-wrapper}

В зависимости от набора внешних таблиц проекция standalone-таблицы в системе может быть:
* полной и включать возможности чтения и записи,
* частичной и включать возможность только записи или только чтения.

![](standalone_table.svg)
{: .figure-center}
*Связи standalone-таблицы с внешними таблицами*
{: .figure-caption-center}

Для данных standalone-таблицы, полностью спроецированной в систему, доступны те же действия, что и для данных 
[логических таблиц](../logical_table/logical_table.md):
* [загрузка данных](../../../working_with_system/data_upload/data_upload.md),
* [обновление данных](../../../working_with_system/data_update/data_update.md),
* [чтение данных](../../../working_with_system/data_reading/data_reading.md),
* [выгрузка данных](../../../working_with_system/data_download/data_download.md).

Для частично спроецированной таблицы доступны либо загрузка и обновление, либо выгрузка и чтение.

Для standalone-таблиц действуют все ограничения той СУБД, которая содержит эти таблицы. Подробнее об ограничениях
системы см. в разделе [Ограничения системы](../../../restrictions/restrictions.md), об ограничениях 
конкретной СУБД см. в документации СУБД.

Standalone-таблицы не поддерживают версионирование данных. Все изменения данных выполняются вне механизма
[дельт](../delta/delta.md) и [операций записи](../write_operation/write_operation.md).

Данные standalone-таблицы можно поместить в логическую таблицу с помощью запроса
[INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md). Это позволит использовать
преимущества логических таблиц: иметь доступ к истории изменений данных, а также изменять данные без использования
внешних readable-таблиц и writable-таблиц. В некоторых случаях это также позволит обойти ограничения, связанные с
конкретной СУБД.
{: .note-wrapper}

Чтобы standalone-таблицу можно было спроецировать в систему, она должна располагаться:
* в той же физической базе данных, где хранятся данные связанной [логической базы данных](../logical_db/logical_db.md), 
  — если таблица находится в ADB или ADP;
* в том же кластере, где хранятся данные связанной логической базы данных, — если таблица находится в ADQM или ADG.