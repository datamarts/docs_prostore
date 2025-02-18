---
layout: default
title: UPSERT VALUES
nav_order: 45
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# UPSERT VALUES
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос вставляет записи в [логической таблицу](../../../overview/main_concepts/logical_table/logical_table.md)
или [standalone-таблицу](../../../overview/main_concepts/standalone_table/standalone_table.md).

Синтаксис вставки в standalone-таблицу подразумевает использование
[внешней writable-таблицы](../../../overview/main_concepts/external_table/external_table.md#writable_table), которая
указывает на standalone-таблицу.
При вставке данных в standalone-таблицу нужно учитывать ее ограничения в конкретной СУБД.

Запрос поддерживается для ADB и ADP.
{: .note-wrapper}

Если в логической таблице уже есть запись со значением первичного ключа, указанным в запросе, то запись обновляется значениями 
из запроса. Иначе, если запись отсутствует, то добавляется новая запись со значениями из запроса, а пропущенные поля 
заполняются значениями по умолчанию.

Вставка данных в [логические](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованные представления](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступна.

В отличие от [INSERT VALUES](../INSERT_VALUES/INSERT_VALUES.md), запрос `UPSERT VALUES` обновляет существующую запись 
только теми значениями, которые указаны в запросе. 
Для полного обновления существующих записей следует использовать запрос [INSERT VALUES](../INSERT_VALUES/INSERT_VALUES.md).
<br>Для обновления большого объема данных следует использовать
[загрузку данных](../../../working_with_system/data_upload/data_upload.md).
{: .note-wrapper}

Запрос обрабатывается в порядке, описанном в разделе 
[Порядок обработки запросов на обновление данных](../../../overview/interactions/llw_processing/llw_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

Месторасположение данных таблицы можно задавать запросами 
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с ключевым словом 
`DATASOURCE_TYPE`.
{: .note-wrapper}

Записи, вставленные в логическую таблицу, добавляются как горячие записи. При [фиксации изменений](../COMMIT_DELTA/COMMIT_DELTA.md)
записи становятся актуальными, а предыдущие актуальные записи — архивными. 
Подробнее о версионировании
см. в разделе [Версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).

Незавершенную [операцию](../../../overview/main_concepts/write_operation/write_operation.md) по вставке данных
можно перезапустить, повторив исходный запрос с ключевым словом [RETRY](#retry). Подробнее обо всех способах
обработки незавершенных операций см. в разделе
[Управление операциями записи](../../../working_with_system/operation_management/write_op_management/write_op_management.md).

## Синтаксис {#syntax}

Вставка данных во все столбцы таблицы:
```sql
UPSERT INTO [db_name.]table_name VALUES (value_list_1), (value_list_2), ...
```

Вставка данных в некоторые столбцы таблицы (с сохранением значений остальных столбцов без изменений):
```sql
UPSERT INTO [db_name.]table_name (column_list) VALUES (value_list_1), (value_list_2), ...
```

Перезапуск операции по вставке данных:
```sql
RETRY UPSERT INTO [db_name.]table_name [(column_list)] VALUES (value_list_1), (value_list_2), ...
```

**Параметры:**

`db_name`

: Имя логической базы данных. Опционально, если выбрана логическая БД, 
  [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`table_name`

: Имя таблицы, в которую вставляются данные. Возможные значения:
* имя логической таблицы,
* имя [внешней writable-таблицы](../../../overview/main_concepts/external_table/external_table.md#writable_table),
  указывающей на нужную standalone-таблицу.

`column_list`

: Список имен столбцов указанной таблицы. Имена перечисляются в круглых скобках через запятую. 
  Список опционален, если количество и порядок вставляемых значений (в списке `value_list_N`) соответствуют количеству и 
  порядку столбцов в таблице.

`value_list_N`

: Список вставляемых значений. Значения указываются в круглых скобках через запятую. Каждый такой 
  список — это строка, вставляемая в таблицу. 

### Ключевое слово RETRY {#retry}

Ключевое слово перезапускает обработку незавершенной [операции записи](../../../overview/main_concepts/write_operation/write_operation.md), 
созданной запросом `UPSERT VALUES`. Пример запроса см. [ниже](#retry_example). Список незавершенных операций можно получить
с помощью запроса [GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).

Если ключевое слово не указано, система создает новую операцию и обрабатывает ее.

Ключевое слово `RETRY` недоступно в запросах на вставку записей в standalone-таблицу.

Горячую дельту невозможно [закрыть](../COMMIT_DELTA/COMMIT_DELTA.md) или 
[откатить](../ROLLBACK_DELTA/ROLLBACK_DELTA.md), пока в ней есть незавершенные операции записи.
{: .note-wrapper}

## Ограничения {#restrictions}

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Не допускается параллельное выполнение идентичных запросов.

## Примеры {#examples}

### Вставка данных во все столбцы логической таблицы {#all_columns_of_logical_table}

```sql
-- выбор логической базы данных marketing в качестве базы данных по умолчанию
USE marketing;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка трех записей в логическую таблицу sales
UPSERT INTO sales 
VALUES (200011, '2021-08-21 23:34:10', 'ABC0001', 2, 123, 'Покупка по акции "1+1"'), 
       (200012, '2021-08-22 10:05:56', 'ABC0001', 1, 234, 'Покупка без акций'), 
       (200013, '2021-08-22 13:17:47', 'ABC0002', 4, 123, 'Покупка по акции "Лето"');

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в указанные столбцы логической таблицы {#some_columns_of_logical_table}

```sql
-- выбор логической базы данных marketing в качестве базы данных по умолчанию
USE marketing;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка двух записей в логическую таблицу sales (без опционального значения description)
UPSERT INTO sales 
       (id, transaction_date, product_code, product_units, store_id)
VALUES (200014, '2021-08-23 09:34:10', 'ABC0003', 3, 123), 
       (200012, '2021-08-23 20:05:56', 'ABC0001', 6, 234); 

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных во все столбцы standalone-таблицы {#all_columns_of_standalone_table}

```sql
-- вставка записей в standalone-таблицу, на которую указывает внешняя writable-таблица agreements_ext_write_adp
UPSERT INTO marketing.agreements_ext_write_adp 
VALUES (200, 444444, 'AB22222', '2022-02-08', '2022-02-09', '2024-02-09', ''), 
       (201, 555555, 'AB33333', '2022-02-10', '2022-02-11', '2025-02-11', 'Договор с ООО "Овал"');
```

### Вставка данных в указанные столбцы standalone-таблицы {#some_columns_of_standalone_table}

```sql
-- вставка записей в standalone-таблицу, на которую указывает внешняя writable-таблица agreements_ext_write_adp,
--  без некоторых опциональных значений
UPSERT INTO marketing.agreements_ext_write_adp (id, client_id, number, signature_date)
VALUES (202, 999999, 'AB44444', '2022-01-01');
```

### Перезапуск операции по вставке записей {#retry_example}

```sql
-- выбор логической базы данных marketing в качестве базы данных по умолчанию
USE marketing;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка записи в логическую таблицу sales (без опционального значения description)       
UPSERT INTO sales
       (id, transaction_date, product_code, product_units, store_id)
VALUES (200015, '2021-10-15 10:11:01', 'ABC0003', 1, 123);

-- перезапуск обработки операции по вставке записи
RETRY UPSERT INTO sales
       (id, transaction_date, product_code, product_units, store_id)
VALUES (200015, '2021-10-15 10:11:01', 'ABC0003', 1, 123); 

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```     