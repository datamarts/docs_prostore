---
layout: default
title: CREATE WRITABLE EXTERNAL TABLE
nav_order: 19.5
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE WRITABLE EXTERNAL TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет создать [внешнюю writable-таблицу](../../../overview/main_concepts/external_table/external_table.md#writable_table).

По умолчанию система создает внешнюю таблицу в [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md) 
и не создает связанную с ней
[standalone-таблицу](../../../overview/main_concepts/standalone_table/standalone_table.md) в СУБД хранилища. 
Это может быть полезно, если standalone-таблица уже существует в СУБД.
Чтобы standalone-таблица автоматически создалась при создании внешней таблицы,
укажите в запросе ключевое слово `OPTIONS` со значением `auto.create.table.enable=true`.

Внешняя таблица должна содержать все физические поля standalone-таблицы. Например, внешняя таблица, связанная с таблицей
ADG, должна содержать поле `bucket_id` с типом `INT` и ограничением `NOT NULL` (см. пример [ниже](#adg_without_options)).
{: .note-wrapper}

В ответе возвращается:
* пустой объект ResultSet при успешном выполнении запроса;
* исключение при неуспешном выполнении запроса.

Наличие внешней таблицы можно проверить, как описано в разделе 
[Проверка наличия логической сущности](../../../working_with_system/logical_schema_update/entity_presence_check/entity_presence_check.md).

Изменение внешней таблицы недоступно. Для замены таблицы необходимо удалить ее и создать новую.
{: .note-wrapper}

## Синтаксис {#syntax}

```sql
CREATE WRITABLE EXTERNAL TABLE [db_name.]ext_table_name_ext (
  column_name_1 datatype_1 [NULL | NOT NULL],
  column_name_2 datatype_2 [NULL | NOT NULL],
  column_name_3 datatype_3 [NULL | NOT NULL],
  [PRIMARY KEY (column_list_1)]
) 
[DISTRIBUTED BY (column_list_2)]
LOCATION 'core:datasource_alias://path_to_table'
[OPTIONS ('option_list')]
```

**Параметры:**

`db_name`

: Имя логической базы данных, в которой создается внешняя таблица. Опционально, если выбрана
  логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

`ext_table_name`

: Имя создаваемой внешней таблицы. Должно быть уникально среди всех логических
  сущностей логической базы данных, а также должно удовлетворять другим условиям, перечисленным в секции [Ограничения](#restrictions). 
  <br>Чтобы быстро различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип 
  таблицы, например `agreements_ext_write` или `agreements_ext_write_adp`.
  При необходимости типы writable- и readable-таблиц можно проверить в системном представлении 
  [tables](../../system_views/system_views.md#tables).

`column_name_N`

: Имя столбца таблицы. Должно удовлетворять условиям, перечисленным в секции [Ограничения](#restrictions).

`datatype_N`

: Тип данных столбца `column_name_N`. Возможные значения см.
  в разделе [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md).

`column_list_1`

: Список столбцов, входящих в первичный ключ таблицы. Опционален, если используется существующая 
  standalone-таблица. Рекомендуется заполнять в соответствии с первичным ключом standalone-таблицы.

`column_list_2`

: Список столбцов, входящих в ключ шардирования таблицы. Опционален, если используется существующая
  standalone-таблица. Рекомендуется заполнять в соответствии с ключом шардирования standalone-таблицы.

`datasource_alias`

: Псевдоним СУБД хранилища, в которой уже размещена или нужно разместить standalone-таблицу, связанную
  с внешней таблицей. Возможные значения: `adb`, `adqm`, `adg`, `adp`.

`path_to_table`

: Путь к связанной standalone-таблице. Состоит из имени схемы (если применимо для СУБД) и имени таблицы,
  указанных через точку. Примеры: `dtm__sales.agreements` (adqm), `sales.agreements` (adp, adb), `dtm__sales__agreements` (adg).

`option_list`

: Список дополнительных параметров и их значений в формате `option1=value1;option2=value2...`.
  Возможные параметры:
  * `auto.create.table.enable` — признак создания связанной standalone-таблицы, возможные значения: `true` — создать таблицу,
    `false` (по умолчанию) — не создавать таблицу.

## Ограничения {#restrictions}

* Имена таблицы и ее столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md) и
  должны начинаться с латинской буквы. После первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания.
* Имена и порядок столбцов должны совпадать во внешней таблице и связанной standalone-таблице.
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## Примеры {#examples}

### Создание таблицы с ключами и параметрами (ADP) {#adp_with_options}

```sql
CREATE WRITABLE EXTERNAL TABLE sales.agreements_ext_write_adp (
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
CREATE WRITABLE EXTERNAL TABLE sales.payments_ext_write_adg (
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