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

_Standalone-таблица_ — таблица СУБД хранилища, не относящаяся к [логической](../logical_schema/logical_schema.md) и 
[физической](../physical_schema/physical_schema.md) схемам данных. Standalone-таблицу можно рассматривать как 
источник данных для системы.

С данными standalone-таблиц можно работать через систему, используя
единый [синтаксис SQL+](../../../reference/sql_plus_requests/sql_plus_requests.md).
Синтаксис для работы со standalone-таблицей предполагает использование [внешних таблиц](../external_table/external_table.md), 
которые указывают на standalone-таблицу:
* [внешняя readable-таблица](../external_table/external_table.md#readable_table) позволяет 
  [читать](../../../working_with_system/data_reading/data_reading.md) и 
  [выгружать](../../../working_with_system/data_download/data_download.md) данные,
* [внешняя writable-таблица](../external_table/external_table.md#writable_table) позволяет 
  [вставлять](../../../working_with_system/data_update/data_update.md) и 
  [загружать](../../../working_with_system/data_upload/data_upload.md) данные.

Внешняя таблица может указывать на существующую standalone-таблицу. Если standalone-таблица отсутствует, 
она может быть автоматически создана при создании внешней таблицы.

Подробнее об создании внешних таблиц см. в разделах 
[CREATE WRITABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_WRITABLE_EXTERNAL_TABLE/CREATE_WRITABLE_EXTERNAL_TABLE.md) и 
[CREATE READABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md).
{: .note-wrapper}

<a id="img_standalone_table"></a>
![](standalone_table.svg)
{: .figure-center}
*Связи standalone-таблицы с внешними таблицами*
{: .figure-caption-center}

При работе со standalone-таблицей нужно учитывать ограничения той СУБД, в которой находится таблица.
{: .note-wrapper}

Чтобы со standalone-таблицей можно было работать через систему, она должна располагаться:
* в той же базе данных, где хранятся данные связанной [логической базы данных](../logical_db/logical_db.md),
  — если таблица находится в ADB или ADP;
* в том же кластере, где хранятся данные связанной логической базы данных, — если таблица находится в ADQM или ADG.

Standalone-таблицы не поддерживают версионирование данных. Все изменения данных выполняются вне механизма
[дельт](../delta/delta.md) и [операций записи](../write_operation/write_operation.md).

Данные standalone-таблицы можно поместить в логическую таблицу с помощью запроса
[INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md). Это позволит использовать
преимущества логических таблиц: иметь доступ к истории изменений данных, а также изменять данные без использования
внешних readable-таблиц и writable-таблиц. В некоторых случаях это также позволит обойти ограничения, связанные с
конкретной СУБД.
{: .note-wrapper}