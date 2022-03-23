---
layout: default
title: ERASE_WRITE_OPERATION
nav_order: 27.7
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# ERASE_WRITE_OPERATION

Запрос отменяет незавершенную [операцию записи](../../../overview/main_concepts/write_operation/write_operation.md). 
Можно отменить любую операцию: запущенную запросом [загрузки данных](../../../working_with_system/data_upload/data_upload.md) 
или запущенную запросом [обновления данных](../../../working_with_system/data_update/data_update.md).

Успешный ответ содержит ResultSet с одной строкой, в которой представлена информация об отмененной операции. 
Неуспешный ответ содержит исключение.

По отмененной операции возвращается следующая информация:
* `sys_cn` — номер операции записи;
* `status` — статус операции записи. Возможные значения: 3 — отменена;
* `destination_table_name` — имя таблицы-приемника данных;
* `external_table_name` — имя [внешней таблицы](../../../overview/main_concepts/external_table/external_table.md)
  загрузки, которая участвовала в операции записи. Значение отсутствует, если внешняя таблица не участвовала
  в операции (например, операция была запущена функцией 
  [обновления данных](../../../working_with_system/data_update/data_update.md));
* `query` — исходный запрос операции записи.

## Синтаксис {#syntax}

```sql
ERASE_WRITE_OPERATION(sys_cn[, db_name])
```

**Параметры:**

`sys_cn`

: Номер операции записи. Номера незавершенных операций можно получить с помощью запроса 
  [GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).

`db_name`

: Имя логической базы данных, к которой относится операция. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

## Примеры {#examples}

Отмена операции в указанной логической базе данных:

```sql
ERASE_WRITE_OPERATION(10, sales)
```

Отмена операции в логической базы данных, выбранной по умолчанию:

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- запрос журнала для sales
ERASE_WRITE_OPERATION(10);
```

<!--
На рисунке ниже показан пример успешного ответа на запрос `ERASE_CHANGE_OPERATION`.

![]()
{: .figure-center}
*Ответ ERASE_CHANGE_OPERATION*
{: .figure-caption-center}
-->