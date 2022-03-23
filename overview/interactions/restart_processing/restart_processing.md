---
layout: default
title: Порядок перезапуска системы
nav_order: 6
parent: Связи с другими системами и компонентами
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Порядок перезапуска системы {#restart_processing}

При перезапуске системы выполняются следующие действия:
1. Возобновляются незавершенные [операции записи](../../main_concepts/write_operation/write_operation.md): 
   * по всем операциям со статусом «Отменяется» запускается отмена,
   * по операциям со статусом «Выполняется», запущенным запросами 
   [загрузки данных](../../../working_with_system/data_upload/data_upload.md), возобновляется 
   отслеживание загрузки данных в [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
   [хранилища](../../main_concepts/data_storage/data_storage.md).
2. Запускается синхронизация [материализованных представлений](../../main_concepts/materialized_view/materialized_view.md) 
   [окружения](../../main_concepts/environment/environment.md), если она включена в 
   [конфигурации системы](../../../maintenance/configuration/system/system.md).
   
Синхронизацией материализованных представлений можно управлять с помощью параметра конфигурации 
`MATERIALIZED_VIEWS_SYNC_PERIOD_MS`.
{: .note-wrapper}