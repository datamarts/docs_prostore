﻿---
layout: default
title: DROP READABLE EXTERNAL TABLE
nav_order: 24.5
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# DROP READABLE EXTERNAL TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос удаляет [внешнюю readable-таблицу](../../../overview/main_concepts/external_table/external_table.md#readable_table).

По умолчанию система удаляет внешнюю таблицу из [логической базы данных](../../../overview/main_concepts/logical_db/logical_db.md) 
и не удаляет связанную с ней
[standalone-таблицу](../../../overview/main_concepts/standalone_table/standalone_table.md) в СУБД хранилища. 
Чтобы standalone-таблица автоматически удалилась при удалении внешней таблицы,
укажите в запросе ключевое слово `OPTIONS` со значением `auto.drop.table.enable=true`.

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

## Синтаксис {#syntax}

```sql
DROP READABLE EXTERNAL TABLE [db_name.]ext_table_name
[OPTIONS ('option_list')]
```

**Параметры:**

`db_name`

: Имя логической базы данных, из которой удаляется внешняя таблица. Опционально, если выбрана
  логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`ext_table_name`

: Имя удаляемой внешней таблицы.

`option_list`

: Список дополнительных параметров и их значений в формате `option1=value1;option2=value2...`.
  Возможные параметры:
  * `auto.drop.table.enable` — признак удаления связанной standalone-таблицы, возможные значения: `true` — удалить таблицу,
    `false` (по умолчанию) — не удалять таблицу.

## Ограничения {#restrictions}

Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Примеры {#examples}

### Удаление внешней таблицы с удалением standalone-таблицы {#example_with_options}

```sql
DROP READABLE EXTERNAL TABLE marketing.agreements_ext_read_adp
OPTIONS ('auto.drop.table.enable=true')
```

### Удаление внешней таблицы без удаления standalone-таблицы {#example_without_options}

```sql
DROP READABLE EXTERNAL TABLE marketing.payments_ext_read_adg
```