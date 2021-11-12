---
layout: default
title: CHECK_MATVIEW
nav_order: 5
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CHECK_MATVIEW

Запрос позволяет получить информацию по одному или всем 
[материализованным представлениям](../../../overview/main_concepts/materialized_view/materialized_view.md) 
[логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md).

По каждому материализованному представлению доступна следующая информация:
* `datamart_name` — имя логической базы данных, к которой относится представление;
* `matview_name` — имя представления;
* `view_query` — [SELECT](../SELECT/SELECT.md)-запрос, на котором построено представление;
* `datasource_type` — список [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), в которых размещены данные представления. 
  Возможные значения: adb, adqm;
* `last_sync_time` — дата и время последней синхронизации представления с СУБД-источником;
* `last_sync_delta` — номер последней [дельты](../../../overview/main_concepts/delta/delta.md) в представлении;
* `last_sync_error_reason` — причина ошибки последней синхронизации представления, если такая ошибка была;
* `sync_now` — признак того, синхронизируется ли представление в текущий момент. Возможные значения: yes — синхронизируется, 
  no — нет;
* `sync_period` — периодичность синхронизации представления (в миллисекундах);
* `datamart_delta_ok` — последняя закрытая дельта в логической базе данных.

Успешный ответ содержит объект ResultSet, где каждая строка соответствует одному материализованному представлению, 
неуспешный ответ содержит исключение.

## Синтаксис {#syntax}

```sql
CHECK_MATVIEW([[db_name].materialized_view_name])
```

Параметры:
*   `db_name` — имя логической базы данных, в которой находятся запрашиваемые материализованные представления. Параметр
    опционален, если выбрана логическая БД, 
    [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `materialized_view_name` — имя материализованного представления, по которому запрашивается информация.

## Примеры {#examples}

### Запрос информации по всем представлениям логической БД, выбранной по умолчанию {#all_views}

```sql
CHECK_MATVIEW()
```

### Запрос информации по одному представлению {#one_view}

Запрос в логической БД, выбранной по умолчанию:

```sql
CHECK_MATVIEW(sales_december_2020)
```

Запрос в указанной логической БД:

```sql
CHECK_MATVIEW(sales.sales_december_2020)
```


<!--
### Запрос информации по одному представлению логической БД, выбранной по умолчанию {#one_in_current_db}

```sql
CHECK_MATVIEW(sales_december_2020)
```

### Запрос информации по одному представлению указанной логической БД {#one_in_specified_db}

```sql
CHECK_MATVIEW(sales.sales_december_2020)
```
-->

<скриншот>