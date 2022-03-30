---
layout: default
title: Порядок обработки запросов на загрузку данных
nav_order: 2
parent: Связи с другими системами и компонентами
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Порядок обработки запросов на загрузку данных {#upload_processing}

Запрос на загрузку данных обрабатывается в следующем порядке:
1. Внешняя информационная система отправляет запрос 
   [INSERT SELECT FROM upload_external_table](../../../reference/sql_plus_requests/INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md) 
   через JDBC-драйвер Prostore.
2. Запрос поступает в сервис исполнения запросов Prostore.
3. Сервис исполнения запросов отправляет команду на загрузку данных в соответствующие коннекторы и 
   отслеживает состояние загрузки с помощью сервиса мониторинга статусов Kafka. 
   <br>Команда отправляется в коннекторы 
   тех [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
   [хранилища](../../main_concepts/data_storage/data_storage.md), в которых хранятся данные 
   [логической таблицы](../../main_concepts/logical_table/logical_table.md), или той СУБД, где размещается 
   [standalone-таблица](../../main_concepts/standalone_table/standalone_table.md).
4. Информация о процессе загрузки данных сохраняется в [сервисной базе данных](../../main_concepts/service_db/service_db.md).
5. Каждый задействованный коннектор загружает данные из топика Kafka в свою СУБД хранилища.
   <br>Используется топик, с которым связана внешняя 
   таблица загрузки, указанная в запросе [INSERT SELECT FROM upload_external_table](../../../reference/sql_plus_requests/INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md).
6. По завершении загрузки каждого или всех пакетов данных (в зависимости от СУБД) сервис исполнения 
   запросов отправляет в каждую задействованную СУБД команду на 
   [версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).
7. JDBC-драйвер возвращает ответ во внешнюю информационную систему. Ответ возвращается синхронно — после успешной загрузки всех данных.

Подробнее о компонентах системы см. в разделе [Компоненты системы](../../components/components.md), 
обо всех внешних связях системы см. в разделе [Связи с другими системами и компонентами](../interactions.md).