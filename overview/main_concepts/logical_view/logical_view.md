---
layout: default
title: Логическое представление
nav_order: 5
parent: Основные понятия
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Логическое представление {#logical_view}

_Логическое представление_ — сохраненный запрос к данным 
[логических таблиц](../logical_table/logical_table.md), [standalone-таблиц](../standalone_table/standalone_table.md)
или их соединений. Пример логического представления — список контрагентов, объединенный с их контактами и 
информацией о благонадежности.

Работа с логическими представлениями напоминает работу с реляционными представлениями. 
Логические представления можно [создавать](../../../working_with_system/logical_schema_update/create_view/create_view.md), 
[изменять](../../../working_with_system/logical_schema_update/alter_view/alter_view.md) и 
[удалять](../../../working_with_system/logical_schema_update/drop_view/drop_view.md), а их данные — 
[запрашивать](../../../working_with_system/data_reading/data_reading.md) 
и [выгружать](../../../working_with_system/data_download/data_download.md). Также представление можно использовать как
источник данных в других запросах.

Информацию о запросе, создавшем представление, можно получить с помощью запроса 
[GET_ENTITY_DDL](../../../reference/sql_plus_requests/GET_ENTITY_DDL/GET_ENTITY_DDL.md).
{: .note-wrapper}

Логическое представление указывает на таблицы и не хранит сами данные. [Загрузка](../../../working_with_system/data_upload/data_upload.md) 
и [обновление](../../../working_with_system/data_update/data_update.md) данных в логических представлениях недоступны.