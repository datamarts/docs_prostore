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

Чтобы создать внешнюю readable-таблицу 
в [логической БД](../../../overview/main_concepts/logical_db/logical_db.md), 
выполните запрос [CREATE READABLE EXTERNAL TABLE](../../../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md). 

При успешном выполнении запроса внешняя таблица появляется в 
[логической схеме данных](../../../overview/main_concepts/logical_schema/logical_schema.md). Если в запросе указана
опция `auto.create.table.enable=true`, то создается связанная standalone-таблица в СУБД хранилища.

Для удобства разделения внешних таблиц рекомендуется задавать имя таблицы, указывающее на ее тип 
(например, `payments_ext_read`).
{: .tip-wrapper}

Внешняя readable-таблица представляет собой проекцию таблицы во внешнем источнике данных и не хранит сами данные.
{: .note-wrapper}

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

## Пример {#examples}

```sql
CREATE READABLE EXTERNAL TABLE sales.agreements_ext_read (
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