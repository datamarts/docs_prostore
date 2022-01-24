---
layout: default
title: Обновление данных
nav_order: 4
parent: Работа с системой
has_children: false
has_toc: false
---

# Обновление данных {#data_update}

Система позволяет обновлять небольшие объемы данных, добавляя новые и удаляя устаревшие данные. Обновлять данные 
можно только в [логических таблицах](../../overview/main_concepts/logical_table/logical_table.md).
Обновление данных в [логических](../../overview/main_concepts/logical_view/logical_view.md)
и [материализованных представлениях](../../overview/main_concepts/materialized_view/materialized_view.md)
недоступно.

Под небольшим объемом данных подразумеваются десятки записей.
Для обновления большого объема данных следует использовать [загрузку данных](../data_upload/data_upload.md).
{: .note-wrapper}

Обновление данных в ADG не поддерживается.
{: .note-wrapper}

Чтобы обновить данные в логической таблице:
2.  [Создайте](../../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md)
    логическую таблицу, если она еще не создана.
4.  Выполните запрос [BEGIN DELTA](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)
    на открытие [дельты](../../overview/main_concepts/delta/delta.md),
    если она еще не открыта.
5.  Выполните запрос на обновление данных:
      * [UPSERT VALUES](../../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md) или 
        [UPSERT SELECT](../../reference/sql_plus_requests/UPSERT_SELECT/UPSERT_SELECT.md) — 
        для добавления новых или изменения актуальных данных;
      * [DELETE](../../reference/sql_plus_requests/DELETE/DELETE.md) — для удаления актуальных данных.
6.  Если необходимо, обновите или загрузите другие данные. 
    <br>В открытой дельте можно выполнить произвольное количество запросов на обновление и загрузку данных, 
    а также [отменить изменения](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md). 
    При этом не допускается загрузка различных состояний объекта (то есть различных записей с одинаковым первичным ключом) 
    в одной дельте.
7.  Выполните запрос [COMMIT DELTA](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md)
    для сохранения изменений и закрытия дельты.

При успешном выполнении действий состояние данных системы обновляется, как описано в разделе 
[Версионирование данных](../data_upload/data_versioning/data_versioning.md).

Пока дельта не закрыта, изменения по всем ее завершенным [операциям записи](../../overview/main_concepts/write_operation/write_operation.md) 
можно отменить с помощью запроса [ROLLBACK DELTA](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md). 
Изменения по незавершенным операциям отменить невозможно.
{: .note-wrapper}

## Примеры {#examples}

```sql
-- выбор логической базы данных sales в качестве базы данных по умолчанию
USE sales;

-- открытие новой (горячей) дельты
BEGIN DELTA;

-- вставка трех записей в логическую таблицу sales
UPSERT INTO sales 
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
UPSERT INTO sales_july_2021 
SELECT * FROM sales WHERE CAST(EXTRACT(MONTH FROM transaction_date) AS INT) = 7 AND 
  CAST(EXTRACT(YEAR FROM transaction_date) AS INT) = 2021 DATASOURCE_TYPE = 'adb';

-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```