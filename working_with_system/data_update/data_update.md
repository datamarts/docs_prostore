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

Система позволяет обновлять небольшие объемы данных, добавляя новые и удаляя устаревшие данные. Данные можно обновлять 
в [логических таблицах](../../overview/main_concepts/logical_table/logical_table.md) и внешних writable-таблицах.
Обновление данных в [логических](../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлениях](../../overview/main_concepts/materialized_view/materialized_view.md)
недоступно.

Под небольшим объемом данных подразумеваются десятки записей.
Для обновления большого объема данных следует использовать [загрузку данных](../data_upload/data_upload.md).
{: .note-wrapper}

Обновление данных поддерживается в ADB, ADQM и ADP.
{: .note-wrapper}

Чтобы обновить данные в логической таблице или внешней writable-таблице:
1. [Создайте](../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md)
   логическую таблицу или внешнюю writable-таблицу, если она еще не создана.
2. При обновлении данных логической таблицы: выполните запрос [BEGIN DELTA](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)
   на открытие [дельты](../../overview/main_concepts/delta/delta.md), если она еще не открыта.
3. Выполните запрос на обновление данных:
      * [INSERT VALUES](../../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md), 
        [INSERT SELECT](../../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md) или 
        [UPSERT VALUES](../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) — 
        для добавления новых или изменения существующих данных;
      * [DELETE](../../reference/sql_plus_requests/DELETE/DELETE.md) — для удаления данных.
4. Если необходимо, обновите или загрузите другие данные.
   <br>В открытой дельте можно выполнять множество запросов на обновление и загрузку данных, но при этом записи, добавляемые
   в каждую из таблиц, должны иметь разные значения первичного ключа или быть полными дубликатами. Система не гарантирует
   корректную обработку разных записей таблицы с одинаковыми значениями первичного ключа в одной дельте.
5. При обновлении данных логической таблицы: выполните запрос [COMMIT DELTA](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md)
   для сохранения изменений и закрытия дельты.

При успешном выполнении действий состояние данных в логических таблицах обновляется, как описано в разделе 
[Версионирование данных](../data_upload/data_versioning/data_versioning.md).

Пока дельта не закрыта, внесенные в ней изменения данных можно отменить запросом
[ROLLBACK DELTA](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).
{: .note-wrapper}

## Примеры {#examples}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

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