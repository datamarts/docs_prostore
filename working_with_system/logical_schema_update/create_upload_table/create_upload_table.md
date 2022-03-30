---
layout: default
title: Создание внешней таблицы загрузки
nav_order: 10
grand_parent: Работа с системой
parent: Управление схемой данных
has_children: false
has_toc: false
---

# Создание внешней таблицы загрузки {#create_upload_table}

Чтобы создать [внешнюю таблицу загрузки](../../../overview/main_concepts/external_table/external_table.md) 
в [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE UPLOAD EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_UPLOAD_EXTERNAL_TABLE/CREATE_UPLOAD_EXTERNAL_TABLE.md). 
При успешном выполнении запроса внешняя таблица загрузки появляется в 
[логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md). 

Для удобства разделения таблиц загрузки и выгрузки рекомендуется задавать имя таблицы, 
указывающее на ее тип (например, `transactions_ext_upload` или `transactions_ext_download`).
{: .tip-wrapper}

Внешняя таблица представляет собой декларацию источника данных и формата загрузки данных и 
не хранит сами данные.
{: .note-wrapper}

Наличие внешней таблицы можно проверить, как описано в разделе [Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
-- выбор базы данных marketing по умолчанию
USE marketing;

-- создание внешней таблицы загрузки
CREATE UPLOAD EXTERNAL TABLE sales_ext_upload (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://zk1:2181,zk2:2181,zk3:2181/sales'
FORMAT 'AVRO'
MESSAGE_LIMIT 1000;
```