---
layout: default
title: Управление схемой данных
nav_order: 2
parent: Работа с системой
has_children: true
has_toc: false
---

# Управление схемой данных {#logical_schema_update}

Система позволяет организовать структуру данных с помощью [логической схемы данных](../../overview/main_concepts/logical_schema/logical_schema.md). 
Доступны следующие действия по управлению логической схемой данных:
* изменение логической схемы:
  * [создание логической базы данных](create_db/create_db.md),
  * [удаление логической базы данных](drop_db/drop_db.md),
  * [создание логической таблицы](create_table/create_table.md),
  * [удаление логической таблицы](drop_table/drop_table.md),
  * [создание логического представления](create_view/create_view.md),
  * [изменение логического представления](alter_view/alter_view.md),
  * [удаление логического представления](drop_view/drop_view.md),
  * [создание материализованного представления](create_materialized_view/create_materialized_view.md),
  * [удаление материализованного представления](drop_materialized_view/drop_materialized_view.md),
  * [создание внешней таблицы загрузки](create_upload_table/create_upload_table.md),
  * [удаление внешней таблицы загрузки](drop_upload_table/drop_upload_table.md),
  * [создание внешней таблицы выгрузки](create_download_table/create_download_table.md),
  * [удаление внешней таблицы выгрузки](drop_download_table/drop_download_table.md),
* [проверка наличия логической сущности](entity_presence_check/entity_presence_check.md),
* [запрос метаданных логической схемы](request_from_schema/request_from_schema.md).

Запросы на обновление логической схемы данных обрабатываются в порядке, описанном в разделе 
[Порядок обработки запросов на обновление логической схемы](../../overview/interactions/ddl_processing/ddl_processing.md).
Информация о каждом изменении записывается в [системный топик Kafka](../../reference/system_topic_format/system_topic_format.md).