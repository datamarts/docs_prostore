﻿---
layout: default
title: Создание внешней writable-таблицы
nav_order: 12.5
grand_parent: Работа с системой
parent: Управление схемой данных
has_children: false
has_toc: false
---

# Создание внешней writable-таблицы {#create_writable_table}

Чтобы создать [внешнюю writable-таблицу](../../../overview/main_concepts/external_table/external_table.md#writable_table) 
в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE WRITABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_WRITABLE_EXTERNAL_TABLE/CREATE_WRITABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица появляется в 
[логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана
опция `auto.create.table.enable=true`, в СУБД хранилища создается связанная 
[standalone-таблица](../../../overview/main_concepts/standalone_table/standalone_table.md).

Чтобы быстро различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип
таблицы, например `payments_ext_write` или `payments_ext_write_adg`. При необходимости типы writable- и readable-таблиц 
можно проверить в системном представлении [tables](../../../reference/system_views/system_views.md#tables).
{: .tip-wrapper}

Внешняя writable-таблица указывает на standalone-таблицу и не хранит сами данные.
{: .note-wrapper}

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Примеры {#examples}

### Создание таблицы с ключами и параметрами (ADP) {#adp_with_options}

```sql
CREATE WRITABLE EXTERNAL TABLE marketing.agreements_ext_write_adp (
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
LOCATION 'core:adp://marketing.agreements'
OPTIONS ('auto.create.table.enable=true')
```

### Создание таблицы без ключей и параметров (ADG) {#adg_without_options}

```sql
CREATE WRITABLE EXTERNAL TABLE marketing.payments_ext_write_adg (
  id INT NOT NULL,
  agreement_id INT,
  code VARCHAR(16),
  amount DOUBLE,
  currency_code VARCHAR(3),
  description VARCHAR,
  bucket_id INT NOT NULL
)
LOCATION 'core:adg://dtm__marketing__payments'
```