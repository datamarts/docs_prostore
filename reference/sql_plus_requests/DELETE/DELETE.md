---
layout: default
title: DELETE
nav_order: 18
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DELETE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет переместить в архив актуальные записи [логической таблицы](../../../overview/main_concepts/logical_table/logical_table.md), 
соответствующие указанному условию. Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на обновление данных](../../../overview/interactions/llw_processing/llw_processing.md).

Архивация данных возможна только в логической таблице. Архивация данных в 
[логических](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлениях](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступна.

Для обновления большого объема данных следует использовать 
[загрузку данных](../../../working_with_system/data_upload/data_upload.md).
{: .note-wrapper}

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

## Синтаксис {#syntax}

```sql
DELETE FROM [db_name.]table_name [WHERE filter_expression]
```

Параметры:
*   `db_name` — имя логической базы данных. Опционально, если выбрана логическая БД,
    [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `table_name` — имя логической таблицы, данные которой архивируются;
*   `filter_expression` — условие выбора записей, подлежащих архивации. Если ключевое слово WHERE и условие не указаны, 
    архивируются все актуальные данные таблицы.

## Ограничения {#restrictions}

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Столбцы в условии запроса не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`,
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`. 

## Пример {#examples}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- архивация записей логической таблицы sales о покупках в магазине, который был закрыт
DELETE FROM sales WHERE store_id = 234;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```