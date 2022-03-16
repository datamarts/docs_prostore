---
layout: default
title: DELETE
nav_order: 20
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

Запрос позволяет удалить записи согласно условию, указанному в блоке `WHERE`.
В зависимости от вида таблицы удаляются следующие записи:
* актуальные записи [логической таблицы](../../../overview/main_concepts/logical_table/logical_table.md),
* записи [standalone-таблицы](../../../overview/main_concepts/standalone_table/standalone_table.md).

Удаляемые записи логической таблицы становятся архивными, оставаясь при этом доступными для чтения и выгрузки. Записи
standalone-таблицы удаляются насовсем и не могут быть восстановлены средствами системы.

Удаление данных из [логических](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступно.

Удаление записей логической таблицы доступно из ADB, ADQM и ADP, удаление записей standalone-таблицы — из ADB и ADP.
{: .note-wrapper}

Для обновления большого объема данных следует использовать 
[загрузку данных](../../../working_with_system/data_upload/data_upload.md).
{: .tip-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на обновление данных](../../../overview/interactions/llw_processing/llw_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Если [операция записи](../../../overview/main_concepts/write_operation/write_operation.md), запущенная запросом
`DELETE`, зависла, нужно повторить запрос. Список зависших операций можно получить запросом 
[GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).

Запрос `DELETE` не удаляет историю изменений данных в логической таблице. 
<br>Чтобы удалить часть данных с историей изменений или только историю изменений, выполните запрос [TRUNCATE HISTORY](../TRUNCATE_HISTORY/TRUNCATE_HISTORY.md), 
чтобы удалить все данные таблицы вместе с историей изменений — запрос [DROP TABLE](../DROP_TABLE/DROP_TABLE.md). 
Обратите внимание, что данные, удаленные запросами `TRUNCATE HISTORY` и `DROP TABLE`, невозможно восстановить средствами 
системы. 
{: .tip-wrapper}

## Синтаксис {#syntax}

```sql
DELETE FROM [db_name.]table_name [WHERE filter_expression]
```

Параметры:
*   `db_name` — имя логической базы данных. Опционально, если выбрана логическая БД,
    [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `table_name` — имя таблицы, из которой удаляются записи;
*   `filter_expression` — условие выбора удаляемых записей. Если условие не указано, 
    удаляются все актуальные данные таблицы или все данные standalone-таблицы (в зависимости от вида таблицы).

## Ограничения {#restrictions}

* Удаление записей из логических таблиц возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* В блоке `WHERE` не допускается использование функций, которые приводят к разным результатам в разных СУБД хранилища. 
  Примеры таких функций — это операции над числами с плавающей запятой: сравнение с ними, округление и т.д.
* Не допускается параллельное выполнение идентичных запросов.

## Пример {#examples}

### Удаление записей из логической таблицы {#example_logical_table}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- удаление записей по диапазону магазинов из логической таблицы sales
DELETE FROM sales WHERE store_id BETWEEN 100 AND 200;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Удаление записей из внешней writable-таблицы {#example_writable_table}

```sql
-- удаление записей по одному клиенту из внешней writable-таблицы
DELETE FROM sales.agreements_ext_write_adp WHERE client_id = 234;
```