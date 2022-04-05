---
layout: default
title: Обновление данных
nav_order: 4
parent: Работа с системой
has_children: false
has_toc: false
---

# Обновление данных {#data_update}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Система позволяет обновлять небольшие объемы данных: добавлять новые записи, изменять и удалять текущие записи. 

Можно обновлять данные [логических таблиц](../../overview/main_concepts/logical_table/logical_table.md) и 
[standalone-таблиц](../../overview/main_concepts/standalone_table/standalone_table.md).
Обновление данных в [логических](../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлениях](../../overview/main_concepts/materialized_view/materialized_view.md)
недоступно.

Под небольшим объемом данных подразумеваются десятки записей.
Для обновления большого объема данных следует использовать [загрузку данных](../data_upload/data_upload.md).
{: .note-wrapper}

Обновление данных поддерживается в ADB, ADQM и ADP.
{: .note-wrapper}

## Обновление данных логической таблицы {#update_in_logical_table}

Чтобы обновить данные логической таблицы:
1. [Создайте логическую таблицу](../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md),
   если она еще не создана.
2. Выполните запрос [BEGIN DELTA](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)
   на открытие [дельты](../../overview/main_concepts/delta/delta.md), если она еще не открыта.
3. Выполните запрос на обновление данных:
    * [INSERT VALUES](../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md),
      [INSERT SELECT](../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) или
      [UPSERT VALUES](../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) —
      для добавления новых или изменения существующих данных;
    * [DELETE](../../reference/sql_plus_requests/DELETE/DELETE.md) — для удаления данных.
4. Если необходимо, обновите или загрузите другие данные.
   <br>В открытой дельте можно выполнять множество запросов на обновление и загрузку данных. При этом в каждую логическую
   таблицу в одной дельте можно добавлять записи с разными значениями первичного ключа или полные дубликаты.
5. Выполните запрос [COMMIT DELTA](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md)
   для сохранения изменений и закрытия дельты.

При успешном выполнении действий состояние данных в логических таблицах обновляется, как описано в разделе
[Версионирование данных](../data_upload/data_versioning/data_versioning.md).

Пока дельта не закрыта, внесенные изменения можно отменить запросом
[ROLLBACK DELTA](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).
{: .note-wrapper}

Незавершенную [операцию](../../overview/main_concepts/write_operation/write_operation.md) по обновлению 
данных можно перезапустить, повторив исходный запрос с ключевым словом `RETRY` (см. примеры [ниже](#retry_example)).
Подробнее обо всех способах обработки незавершенных операций см. в разделе
[Управление операциями записи](../../working_with_system/operation_management/write_op_management/write_op_management.md).
{: .note-wrapper}

## Обновление данных standalone-таблицы {#update_in_standalone_table}

Чтобы обновить данные в standalone-таблице:
1. [Создайте внешнюю writable-таблицу](../../reference/sql_plus_requests/CREATE_WRITABLE_EXTERNAL_TABLE/CREATE_WRITABLE_EXTERNAL_TABLE.md),
   указывающую на нужную standalone-таблицу, если writable-таблица еще не создана.
2. Выполните запрос на обновление данных:
    * [INSERT VALUES](../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md),
      [INSERT SELECT](../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) или
      [UPSERT VALUES](../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) —
      для добавления новых или изменения существующих данных;
    * [DELETE](../../reference/sql_plus_requests/DELETE/DELETE.md) — для удаления данных.

При обновлении данных в standalone-таблице нужно учитывать ограничения таблицы в конкретной СУБД.
{: .note-wrapper}

## Примеры {#examples}

### Обновление данных в логических таблицах {#logical_table_example}

```sql
-- выбор логической базы данных marketing в качестве базы данных по умолчанию
USE marketing;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка трех записей в логическую таблицу sales
INSERT INTO sales 
VALUES (100011, '2021-08-21 23:34:10', 'ABC0001', 2, 123, 'Покупка по акции "1+1"'), 
       (100012, '2021-08-22 10:05:56', 'ABC0001', 1, 234, 'Покупка без акций'), 
       (1000113, '2021-08-22 13:17:47', 'ABC0002', 4, 123, 'Покупка по акции "Лето"');

-- удаление записей логической таблицы sales о покупках в магазине, который был закрыт
DELETE FROM sales WHERE store_id = 234;

-- создание логической таблицы sales_july_2021, которая будет содержать данные о продажах за июль 2021 и размещаться в ADB
CREATE TABLE sales_july_2021 (
id INT NOT NULL,
transaction_date TIMESTAMP NOT NULL,
product_code VARCHAR(256) NOT NULL,
product_units INT NOT NULL,
store_id INT NOT NULL,
description VARCHAR(256),
PRIMARY KEY (id)
) DISTRIBUTED BY (id)
DATASOURCE_TYPE (adb);

-- вставка данных из таблицы sales в новую таблицу sales_july_2021 
INSERT INTO sales_july_2021 
SELECT * FROM sales WHERE CAST(EXTRACT(MONTH FROM transaction_date) AS INT) = 7 AND 
  CAST(EXTRACT(YEAR FROM transaction_date) AS INT) = 2021 DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Обновление данных в standalone-таблице {#standalone_table_example}

```sql
-- вставка записей в standalone-таблицу, на которую указывает внешняя writable-таблица agreements_ext_write_adp
INSERT INTO sales.agreements_ext_write_adp 
VALUES (100, 111111, 'AB12345', '2022-02-01', '2022-02-02', '2024-02-02', 'Договор с ООО "Квадрат"'), 
       (101, 222222, 'AB67890', '2022-02-11', '2022-02-12', '2025-02-12', 'Договор с ООО "Круг"');
       
-- удаление записей по одному клиенту
DELETE FROM sales.agreements_ext_write_adp WHERE client_id = 234;       

-- создание внешней writable-таблицы с созданием связанной standalone-таблицы в ADQM
CREATE WRITABLE EXTERNAL TABLE sales.sales_ext_write_adqm (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id)
LOCATION 'core:adqm://dtm__sales.sales'
OPTIONS ('auto.create.table.enable=true');

-- вставка данных из логической таблицы sales в standalone-таблицу, на которую указывает внешняя writable-таблица sales_ext_write_adqm
INSERT INTO sales.sales_ext_write_adqm SELECT * FROM sales.sales DATASOURCE_TYPE = 'adqm';
```

### Перезапуск операций по вставке и удалению записей {#retry_example}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка записи в логическую таблицу sales без опционального значения description      
UPSERT INTO sales
       (id, transaction_date, product_code, product_units, store_id)
VALUES (200015, '2021-10-15 10:11:01', 'ABC0003', 1, 123);

-- перезапуск обработки операции по вставке записи
RETRY UPSERT INTO sales
       (id, transaction_date, product_code, product_units, store_id)
VALUES (200015, '2021-10-15 10:11:01', 'ABC0003', 1, 123); 

-- удаление записей логической таблицы sales о покупках в магазине
DELETE FROM sales WHERE store_id = 456;

-- перезапуск обработки операции по удалению записей
RETRY DELETE FROM sales WHERE store_id = 456;

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```   