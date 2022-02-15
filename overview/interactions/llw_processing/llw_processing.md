---
layout: default
title: Порядок обработки запросов на обновление данных
nav_order: 3
parent: Связи с другими системами и компонентами
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Порядок обработки запросов на обновление данных {#llw_processing}

Запрос на обновление данных в [логической таблице](../../main_concepts/logical_table/logical_table.md) 
обрабатывается в следующем порядке:
1.  Внешняя информационная система формирует запрос [INSERT VALUES](../../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md), 
    [DELETE](../../../reference/sql_plus_requests/DELETE/DELETE.md) или 
    [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md), используя JDBC-драйвер Prostore.
2.  Запрос поступает в сервис исполнения запросов Prostore.
3.  Сервис исполнения запросов анализирует запрос и сохраняет информацию о процессе обновления данных в 
    [сервисной базе данных](../../main_concepts/service_db/service_db.md).
4.  Сервис исполнения запросов отправляет запрос на обновление данных в каждую из целевых 
    [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
    [хранилища](../../main_concepts/data_storage/data_storage.md). Под целевыми понимаются 
    СУБД, в которых размещаются данные логической таблицы 
    (см. [CREATE TABLE](../../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md)).
5.  По завершении загрузки всех данных сервис исполнения запросов отправляет в целевые СУБД команду 
    на выполнение задач по 
    [версионированию данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).
6.  После успешного обновления данных JDBC-драйвер возвращает синхронный ответ во внешнюю 
    информационную систему.

Подробнее о компонентах системы см. в разделе [Компоненты системы](../../components/components.md), 
обо всех внешних связях системы см. в разделе [Связи с другими системами и компонентами](../interactions.md).