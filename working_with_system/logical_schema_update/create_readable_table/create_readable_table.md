---
layout: default
title: Создание внешней readable-таблицы
nav_order: 12.3
grand_parent: Работа с системой
parent: Управление схемой данных
has_children: false
has_toc: false
---

# Создание внешней readable-таблицы {#create_readable_table}

Чтобы создать [внешнюю readable-таблицу](../../../overview/main_concepts/external_table/external_table.md#readable_table) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE READABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица появляется в 
[логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана
опция `auto.create.table.enable=true`, в СУБД хранилища создается связанная 
[standalone-таблица](../../../overview/main_concepts/standalone_table/standalone_table.md).

Чтобы можно было различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип
таблицы, например `payments_ext_read` или `payments_ext_read_adg`.
{: .tip-wrapper}

Внешняя readable-таблица указывает на standalone-таблицу и не хранит сами данные.
{: .note-wrapper}

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Примеры {#examples}

### Создание таблицы с ключами и параметрами (ADP) {#adp_with_options}

```sql
CREATE READABLE EXTERNAL TABLE sales.agreements_ext_read_adp (
  id INT NOT NULL,
  client_id INT NOT NULL,
  number VARCHAR NOT NULL,
  signature_date DATE,
  effective_date DATE,
  closing_date DATE,
  description VARCHAR,
  PRIMARY KEY(id)
)
DISTRIBUTED BY (id)
LOCATION 'core:adp://sales.agreements'
OPTIONS ('auto.create.table.enable=true')
```

### Создание таблицы без ключей и параметров (ADG) {#adg_without_options}

```sql
CREATE READABLE EXTERNAL TABLE sales.payments_ext_read_adg (
  id INT NOT NULL,
  agreement_id INT,
  code VARCHAR(16),
  amount DOUBLE,
  currency_code VARCHAR(3),
  description VARCHAR,
  bucket_id INT NOT NULL
)
LOCATION 'core:adg://dtm__sales__payments'
```