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

Запрос на обновление данных обрабатывается в следующем порядке:
1. Внешняя информационная система отправляет запрос [INSERT VALUES](../../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md),
   [INSERT SELECT](../../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md),
   [UPSERT VALUES](../../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) или 
   [DELETE](../../../reference/sql_plus_requests/DELETE/DELETE.md) через JDBC-драйвер Prostore.
2. Запрос поступает в сервис исполнения запросов Prostore.
3. Сервис исполнения запросов отправляет запрос на обновление данных в соответствующие 
   [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
   [хранилища](../../main_concepts/data_storage/data_storage.md). 
   <br>Запрос отправляется в 
   те СУБД, в которых хранятся данные логической таблицы, или в ту СУБД, в которой размещается 
   [standalone-таблица](../../../overview/main_concepts/standalone_table/standalone_table.md).
4. Информация о процессе обновления данных сохраняется в [сервисной базе данных](../../main_concepts/service_db/service_db.md).
5. По завершении загрузки всех данных сервис исполнения запросов отправляет в каждую задействованную СУБД команду 
   на [версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).
6. JDBC-драйвер возвращает ответ во внешнюю информационную систему. Ответ возвращается синхронно — после успешного 
   обновления всех данных.

Подробнее о компонентах системы см. в разделе [Компоненты системы](../../components/components.md), 
обо всех внешних связях системы см. в разделе [Связи с другими системами и компонентами](../interactions.md).