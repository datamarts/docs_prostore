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

Запрос удаляет записи [логической таблицы](../../../overview/main_concepts/logical_table/logical_table.md) или
[standalone-таблицы](../../../overview/main_concepts/standalone_table/standalone_table.md) согласно условию, 
указанному в блоке `WHERE`.

Удаляемые записи логической таблицы становятся архивными, но остаются доступными при чтении и выгрузке исторических данных.
Записи standalone-таблицы удаляются насовсем и не могут быть восстановлены средствами системы.

Синтаксис удаления из standalone-таблицы подразумевает использование
[внешней writable-таблицы](../../../overview/main_concepts/external_table/external_table.md#writable_table), которая
указывает на нужную standalone-таблицу.

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

Незавершенную [операцию](../../../overview/main_concepts/write_operation/write_operation.md) по удалению данных 
можно перезапустить, повторив исходный запрос с ключевым словом [RETRY](#retry). Подробнее обо всех способах 
обработки незавершенных операций см. в разделе 
[Управление операциями записи](../../../working_with_system/operation_management/write_op_management/write_op_management.md).

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

: Имя таблицы, из которой удаляются записи. Возможные значения:
  * имя логической таблицы, 
  * имя внешней writable-таблицы, указывающей на нужную standalone-таблицу.

`filter_expression`

: Условие выбора удаляемых записей. Если условие не указано, удаляются все актуальные данные таблицы или все данные 
  standalone-таблицы (в зависимости от вида таблицы).

### Ключевое слово RETRY {#retry}

Ключевое слово перезапускает обработку незавершенной [операции записи](../../../overview/main_concepts/write_operation/write_operation.md),
созданной запросом `DELETE`. Пример запроса см. [ниже](#retry_example). Список незавершенных операций можно получить
с помощью запроса [GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).

Если ключевое слово не указано, система создает новую операцию и обрабатывает ее.

Ключевое слово `RETRY` недоступно в запросах на удаление записей из standalone-таблицы.

Горячую дельту невозможно [закрыть](../COMMIT_DELTA/COMMIT_DELTA.md) или
[откатить](../ROLLBACK_DELTA/ROLLBACK_DELTA.md), пока в ней есть незавершенные операции записи.
{: .note-wrapper}

## Ограничения {#restrictions}

* Удаление записей из логических таблиц возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* В блоке `WHERE` не допускается использование функций, которые приводят к разным результатам в разных СУБД хранилища. 
  Примеры таких функций — это операции над числами с плавающей запятой: сравнение с ними, округление и т.д.
* Не допускается параллельное выполнение идентичных запросов.

## Примеры {#examples}

### Удаление записей логической таблицы {#example_logical_table}

```sql
-- выбор логической базы данных marketing в качестве базы данных по умолчанию
USE marketing;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- удаление записей логической таблицы sales о покупках в закрытом магазине
DELETE FROM sales WHERE store_id = 234;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Удаление записей standalone-таблицы {#example_standalone_table}

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