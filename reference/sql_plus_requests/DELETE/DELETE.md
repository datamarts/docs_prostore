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

Запрос удаляет записи согласно условию, указанному в блоке `WHERE`.

Удаляются следующие записи:
* актуальные записи [логической таблицы](../../../overview/main_concepts/logical_table/logical_table.md) — если в 
  запросе указана логическая таблица; 
* записи [standalone-таблицы](../../../overview/main_concepts/standalone_table/standalone_table.md) — если в запросе 
  указана [внешняя writable-таблица](../../../overview/main_concepts/external_table/external_table.md#writable_table).
{: .review-highlight}

Удаляемые записи логической таблицы становятся архивными, оставаясь при этом доступными для чтения и выгрузки. Записи
standalone-таблицы удаляются насовсем и не могут быть восстановлены средствами системы.
{: .review-highlight}

Удаление данных из [логических](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступно.

Удаление записей логической таблицы доступно из ADB, ADQM и ADP, удаление записей standalone-таблицы — из ADB и ADP.
{: .note-wrapper .review-highlight}

Для обновления большого объема данных следует использовать 
[загрузку данных](../../../working_with_system/data_upload/data_upload.md).
{: .tip-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на обновление данных](../../../overview/interactions/llw_processing/llw_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Если [операция записи](../../../overview/main_concepts/write_operation/write_operation.md), запущенная запросом
`DELETE`, зависла, нужно повторить запрос, указав ключевое слово [RETRY](#retry).
{: .review-highlight}

Запрос `DELETE` не удаляет историю изменений данных в логической таблице. 
<br>Чтобы удалить часть данных с историей изменений или только историю изменений, выполните запрос [TRUNCATE HISTORY](../TRUNCATE_HISTORY/TRUNCATE_HISTORY.md), 
чтобы удалить все данные таблицы вместе с историей изменений — запрос [DROP TABLE](../DROP_TABLE/DROP_TABLE.md). 
Обратите внимание, что данные, удаленные запросами `TRUNCATE HISTORY` и `DROP TABLE`, невозможно восстановить средствами 
системы. 
{: .tip-wrapper}

## Синтаксис {#syntax}

Удаление записей:
```sql
DELETE FROM [db_name.]table_name [WHERE filter_expression]
```

Перезапуск операции по удалению записей:
```sql
RETRY DELETE FROM [db_name.]table_name [WHERE filter_expression]
```

**Параметры:**

`db_name`

: Имя логической базы данных. Опционально, если выбрана логическая БД,
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`table_name`

: Имя таблицы, из которой удаляются записи. Возможные значения: {: .review-highlight}
  * имя логической таблицы, 
  * имя внешней writable-таблицы, указывающей на нужную standalone-таблицу.
{: .review-highlight}

`filter_expression`

: Условие выбора удаляемых записей. Если условие не указано, удаляются все актуальные данные таблицы или все данные 
  standalone-таблицы (в зависимости от вида таблицы).
{: .review-highlight}

### Ключевое слово RETRY {#retry}

Ключевое слово перезапускает обработку операции записи, созданной запросом `DELETE`. 
Это может быть полезно, если операция зависла: дельту невозможно [закрыть](../COMMIT_DELTA/COMMIT_DELTA.md) или 
[откатить](../ROLLBACK_DELTA/ROLLBACK_DELTA.md), пока есть зависшая операция. Пример запроса см. [ниже](#retry_example).
{: .review-highlight}

Перезапустить обработку можно только для незавершенных операций.
Ключевое слово `RETRY` недоступно в запросах на удаление записей standalone-таблицы.
{: .review-highlight}

Если ключевое слово не указано, система создает новую операцию и обрабатывает ее.
{: .review-highlight}

Список незавершенных (в том числе — зависших) операций можно посмотреть можно с помощью запроса
[GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).
{: .tip-wrapper}
{: .review-highlight}

## Ограничения {#restrictions}

* Удаление записей из логических таблиц возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* В блоке `WHERE` не допускается использование функций, которые приводят к разным результатам в разных СУБД хранилища. 
  Примеры таких функций — это операции над числами с плавающей запятой: сравнение с ними, округление и т.д.
* Не допускается параллельное выполнение идентичных запросов.

## Примеры {#examples}

### Удаление записей логической таблицы {#example_logical_table}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- удаление записей логической таблицы sales о покупках в закрытом магазине
DELETE FROM sales WHERE store_id = 234;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Удаление записей standalone-таблицы через внешнюю writable-таблицу {#example_writable_table}

```sql
-- удаление записей standalone-таблицы, на которую указывает внешняя writable-таблица agreements_ext_write_adp
DELETE FROM sales.agreements_ext_write_adp WHERE client_id < 100;
```

### Перезапуск операции по удалению записей {#retry_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- удаление записей логической таблицы sales о покупках в магазине
DELETE FROM sales WHERE store_id = 123;

-- перезапуск обработки операции по удалению записей
RETRY DELETE FROM sales WHERE store_id = 123;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```    