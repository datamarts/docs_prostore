---
layout: default
title: Порядок обработки запросов на выгрузку данных
nav_order: 4
parent: Связи с другими системами и компонентами
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Порядок обработки запросов на выгрузку данных {#download_processing}

Запрос на выгрузку данных обрабатывается в следующем порядке:
1. Внешняя информационная система формирует запрос [INSERT INTO download_external_table](../../../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md) 
   через JDBC-драйвер Prostore.
2. Запрос поступает в сервис исполнения запросов Prostore.
3. Сервис запрашивает актуальную информацию о 
   [логической схеме данных](../../main_concepts/logical_schema/logical_schema.md) 
   в [сервисной базе данных](../../main_concepts/service_db/service_db.md) и определяет, из какой [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
   [хранилища](../../main_concepts/data_storage/data_storage.md) следует выгрузить данные. Выбор СУБД зависит от параметров
   запроса, месторасположения данных и [конфигурации системы](../../../maintenance/configuration/system/system.md).
4. Сервис исполнения запросов отправляет в коннектор выбранной СУБД команду на выгрузку данных.
5. Коннектор выгружает данные в топик Kafka, с которым связана внешняя таблица выгрузки, 
   указанная в запросе 
   [INSERT INTO download_external_table](../../../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md).
7. JDBC-драйвер возвращает ответ во внешнюю информационную систему. Ответ возвращается синхронно: после успешной выгрузки
   всех данных.
    
Подробнее о компонентах системы см. в разделе [Компоненты системы](../../components/components.md), 
обо всех внешних связях системы см. в разделе [Связи с другими системами и компонентами](../interactions.md).