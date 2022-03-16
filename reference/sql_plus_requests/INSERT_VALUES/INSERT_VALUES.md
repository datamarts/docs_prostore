---
layout: default
title: INSERT VALUES
nav_order: 38
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# INSERT VALUES
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет вставить новые записи и обновить существующие записи в [логической таблице](../../../overview/main_concepts/logical_table/logical_table.md) 
или [внешней writable-таблице](../../../overview/main_concepts/external_table/external_table.md#writable_table). 
При вставке записей в writable-таблицу нужно учитывать ограничения, которые конкретная СУБД накладывает на
работу со своими таблицами.

Существование записи в таблице определяется по значению первичного ключа. Если в таблице существует запись 
со значением первичного ключа, указанным в запросе, запись обновляется; иначе добавляется новая запись.
Новые и существующие записи заполняются одинаково: поля, указанные в запросе, заполняются значениями из запроса, а 
пропущенные поля — значениями по умолчанию.

Вставка данных в [логические](../../../overview/main_concepts/logical_view/logical_view.md)
и [материализованные представления](../../../overview/main_concepts/materialized_view/materialized_view.md)
недоступна.

Запрос поддерживается для ADB, ADQM и ADP.
{: .note-wrapper}

В отличие от [UPSERT VALUES](../UPSERT_VALUES/UPSERT_VALUES.md), запрос `INSERT VALUES` полностью обновляет существующую запись.
Для частичного обновления существующих записей следует использовать запрос [UPSERT VALUES](../UPSERT_VALUES/UPSERT_VALUES.md).
<br>Для обновления большого объема данных следует использовать
[загрузку данных](../../../working_with_system/data_upload/data_upload.md).
{: .note-wrapper}

Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на обновление данных](../../../overview/interactions/llw_processing/llw_processing.md).

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

В зависимости от вида таблицы, указанной в запросе, записи вставляются:
* в физические таблицы тех [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
  [хранилища](../../../overview/main_concepts/data_storage/data_storage.md), в которых размещены данные указанной 
  логической таблицы;
* в [standalone-таблицу](../../../overview/main_concepts/standalone_table/standalone_table.md), связанную с указанной 
  внешней таблицей. 

Месторасположение данных логической таблицы можно задавать запросами 
[CREATE TABLE](../CREATE_TABLE/CREATE_TABLE.md) и [DROP TABLE](../DROP_TABLE/DROP_TABLE.md) с ключевым словом 
`DATASOURCE_TYPE`.
{: .note-wrapper}

Записи, вставленные в логическую таблицу, добавляются как горячие записи. При [фиксации изменений](../COMMIT_DELTA/COMMIT_DELTA.md)
записи становятся актуальными, а предыдущие актуальные записи — архивными. 
Подробнее о версионировании
см. в разделе [Версионирование данных](../../../working_with_system/data_upload/data_versioning/data_versioning.md).

Если [операция записи](../../../overview/main_concepts/write_operation/write_operation.md), запущенная запросом
`INSERT VALUES`, зависла, нужно повторить запрос. Список зависших операций можно получить запросом
[GET_WRITE_OPERATIONS](../GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md).

## Синтаксис {#syntax}

Вставка данных во все столбцы таблицы:
```sql
INSERT INTO [db_name.]table_name VALUES (value_list_1), (value_list_2), ...
```

Вставка данных только в некоторые столбцы таблицы
(с заполнением остальных столбцов значениями по умолчанию):
```sql
INSERT INTO [db_name.]table_name (column_list) VALUES (value_list_1), (value_list_2), ...
```

Параметры:
* `db_name` — имя логической базы данных. Опционально, если выбрана логическая БД, 
    [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
* `table_name` — имя логической или внешней таблицы, в которую вставляются данные;
* `column_list` — список имен столбцов указанной таблицы. Имена перечисляются в круглых скобках через запятую. 
    Список опционален, если количество и порядок вставляемых значений (в списке `value_list_N`) соответствуют количеству и 
    порядку столбцов в таблице;
* `value_list_N` — список вставляемых значений. Значения указываются в круглых скобках через запятую. Каждый такой 
  список — это строка, вставляемая в таблицу.

## Ограничения {#restrictions}

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Не допускается параллельное выполнение идентичных запросов.

## Пример {#examples}

### Вставка данных во все столбцы логической таблицы {#all_columns_of_logical_table}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка трех записей в логическую таблицу sales
INSERT INTO sales 
VALUES (100011, '2021-08-21 23:34:10', 'ABC0001', 2, 123, 'Покупка по акции "1+1"'), 
       (100012, '2021-08-22 10:05:56', 'ABC0001', 1, 234, 'Покупка без акций'), 
       (100113, '2021-08-22 13:17:47', 'ABC0002', 4, 123, 'Покупка по акции "Лето"');

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных в указанные столбцы логической таблицы {#some_columns_of_logical_table}

```sql
-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка двух записей в логическую таблицу sales (без опционального значения description)
INSERT INTO sales.sales 
       (id, transaction_date, product_code, product_units, store_id)
VALUES (100014, '2021-08-23 09:34:10', 'ABC0003', 3, 123), 
       (100012, '2021-08-23 20:05:56', 'ABC0001', 6, 234);

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных во все столбцы внешней writable-таблицы {#all_columns_of_writable_table}

```sql
INSERT INTO sales.agreements_ext_write_adp 
VALUES (100, 111111, 'AB12345', '2022-02-01', '2022-02-02', '2024-02-02', 'Договор с ООО "Квадрат"'), 
       (101, 222222, 'AB67890', '2022-02-11', '2022-02-12', '2025-02-12', 'Договор с ООО "Круг"');
```

### Вставка данных в указанные столбцы внешней writable-таблицы {#some_columns_of_writable_table}

Вставка в таблицу без указания некоторых опциональных значений:
```sql
INSERT INTO sales.agreements_ext_write_adp (id, client_id, number, signature_date)
VALUES (102, 333333, 'AB11111', '2022-01-01');
```