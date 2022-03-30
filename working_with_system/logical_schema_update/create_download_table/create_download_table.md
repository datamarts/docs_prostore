---
layout: default
title: Создание внешней таблицы выгрузки
nav_order: 12
parent: Управление схемой данных
grand_parent: Работа с системой
has_children: false
---

# Создание внешней таблицы выгрузки {#create_download_table}

Чтобы создать [внешнюю таблицу выгрузки](../../../overview/main_concepts/external_table/external_table.md) 
в [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE DOWNLOAD EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md). 
При успешном выполнении запроса внешняя таблица загрузки появляется 
в [логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md).

Для удобства разделения таблиц выгрузки и загрузки рекомендуется задавать имя таблицы, 
указывающее на ее тип (например, `transactions_ext_download` или `transactions_ext_upload`).
{: .tip-wrapper}

Внешняя таблица представляет собой декларацию приемника данных и формата выгрузки данных и 
не хранит сами данные.
{: .note-wrapper}

Наличие внешней таблицы можно проверить, как описано в разделе [Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
-- выбор базы данных marketing по умолчанию
USE marketing;

-- создание внешней таблицы выгрузки
CREATE DOWNLOAD EXTERNAL TABLE marketing.sales_ext_download (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales_out'
FORMAT 'AVRO'
CHUNK_SIZE 1000;
```