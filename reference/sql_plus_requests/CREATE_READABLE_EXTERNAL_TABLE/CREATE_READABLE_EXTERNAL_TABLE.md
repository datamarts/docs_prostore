---
layout: default
title: CREATE READABLE EXTERNAL TABLE
nav_order: 16.5
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CREATE READABLE EXTERNAL TABLE
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет создать [внешнюю readable-таблицу](../../../overview/main_concepts/external_table/external_table.md#readable_table).

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

Внешние таблицы не отображаются в [системных представлениях](../../system_views/system_views.md) `INFORMATION_SCHEMA`.
Наличие внешней таблицы можно проверить, как описано в разделе
[Проверка наличия внешней таблицы](../entity_presence_check/entity_presence_check.md#ext_table_check).

Изменение внешней таблицы недоступно. Для замены таблицы необходимо удалить ее и создать новую.
{: .note-wrapper}

## Синтаксис {#syntax}

```sql
CREATE READABLE EXTERNAL TABLE [db_name.]ext_table_name_ext (
  column_name_1 datatype_1 [NULL | NOT NULL],
  column_name_2 datatype_2 [NULL | NOT NULL],
  column_name_3 datatype_3 [NULL | NOT NULL],
  [PRIMARY KEY (column_list_1)]
) 
[DISTRIBUTED BY (column_list_2)]
LOCATION 'core:<datasource_alias>://<path_to_table>'
[OPTIONS ('<option_list>')]
```

db_name
: Имя логической базы данных, в которой создается внешняя таблица. Опционально, если выбрана
  логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md).

ext_table_name
: Имя создаваемой внешней таблицы. Должно быть уникально среди всех сущностей
  логической базы данных, а также должно удовлетворять другим условиям, перечисленным в секции [Ограничения](#restrictions).
  Чтобы можно было различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип
  таблицы, например `agreements_ext_read_adp`.

column_name_N
: Имя столбца таблицы. Должно удовлетворять условиям, перечисленным в секции [Ограничения](#restrictions).

datatype_N
: Тип данных столбца `column_name_N`. Возможные значения см.
  в разделе [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md).

column_list_1
: Список столбцов, входящих в первичный ключ таблицы. Опционален, если используется существующая
  standalone-таблица. Рекомендуется заполнять в соответствии с первичным ключом standalone-таблицы.

column_list_2
: Список столбцов, входящих в ключ шардирования таблицы. Опционален, если используется существующая
  standalone-таблица. Рекомендуется заполнять в соответствии с ключом шардирования standalone-таблицы.

datasource_alias
: Псевдоним СУБД хранилища, в которой уже размещена или нужно разместить standalone-таблицу, связанную
  с внешней таблицей. Возможные значения: `adb`, `adqm`, `adg`, `adp`.

path_to_table
: Путь к связанной standalone-таблице. Состоит из имени схемы (если применимо для СУБД) и имени таблицы,
  указанных через точку. Примеры: `dtm__sales.agreements` (adqm), `sales.agreements` (adp, adb), `dtm__sales__agreements` (adg).

option_list
: Список дополнительных параметров и их значений в формате `option1=value1;option2=value2...`.
  Возможные параметры:
  * `auto.create.table.enable` — признак создания связанной standalone-таблицы, возможные значения: `true` — создать таблицу,
    `false` (по умолчанию) — пропустить создание таблицы.


Параметры:
* `db_name` — имя логической базы данных, в которой создается внешняя таблица. Опционально, если выбрана 
  логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
* `ext_table_name` — имя создаваемой внешней таблицы. Должно быть уникально среди всех сущностей 
  логической базы данных, а также должно удовлетворять другим условиям, перечисленным в секции [Ограничения](#restrictions). 
  Чтобы можно было различать разные типы внешних таблиц между собой, рекомендуется давать им имена, указывающие на тип 
  таблицы, например `agreements_ext_read_adp`;
* `column_name_N` — имя столбца таблицы. Должно удовлетворять условиям, перечисленным в секции [Ограничения](#restrictions);
* `datatype_N` — тип данных столбца `column_name_N`. Возможные значения см. 
  в разделе [Логические типы данных](../../supported_data_types/logical_data_types/logical_data_types.md);
* `column_list_1` — список столбцов, входящих в первичный ключ таблицы. Опционален, если используется существующая
  standalone-таблица. Рекомендуется заполнять в соответствии с первичным ключом standalone-таблицы;
* `column_list_2` — список столбцов, входящих в ключ шардирования таблицы. Опционален, если используется существующая
  standalone-таблица. Рекомендуется заполнять в соответствии с ключом шардирования standalone-таблицы;
* `datasource_alias` — псевдоним СУБД хранилища, в которой уже размещена или нужно разместить standalone-таблицу, связанную 
  с внешней таблицей. Возможные значения: `adb`, `adqm`, `adg`, `adp`;
* `path_to_table` — путь к связанной standalone-таблице. Состоит из имени схемы (если применимо для СУБД) и имени таблицы, 
  указанных через точку. Примеры: `dtm__sales.agreements` (adqm), `sales.agreements` (adp, adb), `dtm__sales__agreements` (adg);
* `option_list` — список дополнительных параметров и их значений в формате `option1=value1;option2=value2...`. 
  Возможные параметры: 
  * `auto.create.table.enable` — признак создания связанной standalone-таблицы, возможные значения: `true` — создать таблицу, 
    `false` (по умолчанию) — пропустить создание таблицы.

## Ограничения {#restrictions}

* Имена таблицы и ее столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md) и 
  должны начинаться с латинской буквы. После первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания.
* Имена и порядок столбцов должны совпадать во внешней таблице и связанной standalone-таблице.
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

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