---
layout: default
title: Физическая схема данных
nav_order: 10
parent: Основные понятия
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Физическая схема данных {#physical_schema}

_Физическая схема данных_ — структура хранения данных логических сущностей в 
[физических таблицах](../physical_table/physical_table.md) [хранилища](../data_storage/data_storage.md).

Для каждой [логической таблицы](../logical_table/logical_table.md) и каждого 
[материализованного представления](../materialized_view/materialized_view.md) система 
**автоматически** создает и поддерживает набор связанных [физических таблиц](../physical_table/physical_table.md) 
в некоторых или всех [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) хранилища. 

Состав СУБД, в которых создаются физические таблицы и, соответственно, хранятся данные логических сущностей, 
можно регулировать с помощью ключевого слова `DATASOURCE_TYPE` в запросах на создание и удаление логических таблиц и 
материализованных представлений. Подробнее см. в разделах 
[CREATE TABLE](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md),
[DROP TABLE](../../../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md),
[CREATE MATERIALIZED VIEW](../../../reference/sql_plus_requests/CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md) и 
[DROP MATERIALIZED VIEW](../../../reference/sql_plus_requests/DROP_MATERIALIZED_VIEW/DROP_MATERIALIZED_VIEW.md).
{: .note-wrapper}

Состав и содержимое физических таблиц зависят от типа СУБД и описаны в таблице ниже. Например, в ADP все горячие 
записи хранятся в физической таблице с суффиксом `_staging`, а все актуальные 
и архивные записи — в таблице с суффиксом `_actual`.

| Физическая таблица | ADB | ADG | ADQM | ADP
|:-|:-:|:-:|:-:|:-:
| <table>_staging | +<br>Горячие записи | +<br>Горячие записи | − | +<br>Горячие записи
| tbl_buffer | − | − | +<br>Идентификаторы горячих записей | −
| <table>_actual | +<br>Актуальные и архивные записи | +<br>Актуальные записи | +<br>Горячие, актуальные и архивные записи **всех** узлов кластера | +<br>Актуальные и архивные записи
| <table>_history | − | +<br>Архивные записи | − | −
| <table>_actual_shard | − | − | +<br>Горячие, актуальные и архивные записи узла кластера | −
| tbl_buffer_shard | − | − | +<br>Идентификаторы горячих записей узла кластера | −