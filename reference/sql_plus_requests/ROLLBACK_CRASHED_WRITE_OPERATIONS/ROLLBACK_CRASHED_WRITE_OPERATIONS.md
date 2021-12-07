---
layout: default
title: ROLLBACK CRASHED_WRITE_OPERATIONS
nav_order: 37
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# ROLLBACK CRASHED_WRITE_OPERATIONS

Запрос позволяет возобновить обработку [операций записи](../../../overview/main_concepts/write_operation/write_operation.md) 
горячей [дельты](../../../overview/main_concepts/delta/delta.md), находящихся в статусе «Ошибка».

При ошибках [загрузки](../../../working_with_system/data_upload/data_upload.md) и 
[обновления данных](../../../working_with_system/data_update/data_update.md) система автоматически отменяет неуспешные 
операции записи и возвращает данные СУБД [хранилища](../../../overview/main_concepts/data_storage/data_storage.md) 
в состояние, которое предшествовало загрузке или обновлению. Однако, если произошел сбой, например из-за неполадок 
в сети, операция может остаться в статусе «Ошибка». В этом случае следует устранить 
причины, по которым операция не была завершена, и возобновить ее обработку с помощью запроса 
ROLLBACK CRASHED_WRITE_OPERATIONS или [RESUME_WRITE_OPERATION](../RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md).

Перед выполнением запроса необходимо определить логическую базу данных, 
[используемую по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md), 
если она еще не определена.

Успешный ответ содержит объект ResultSet, где каждая строка соответствует одной возобновленной операции записи
со статусом «Ошибка», неуспешный ответ содержит исключение. Пустая строка в ответе означает, что в горячей дельте нет 
операций со статусом «Ошибка».

По каждой возобновленной операции записи возвращается следующая информация:
* `table_name` — имя [логической таблицы](../../../overview/main_concepts/logical_table/logical_table.md)-приемника 
  данных;
* `sys_cn_operations` — номер операции записи.

## Синтаксис {#syntax}

```sql
ROLLBACK CRASHED_WRITE_OPERATIONS
```