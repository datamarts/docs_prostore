---
layout: default
title: Ограничения системы
nav_order: 5.5
has_children: false
has_toc: false
---

# Ограничения системы {#system_limitations}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## [ALLOW_CHANGES](../reference/sql_plus_requests/ALLOW_CHANGES/ALLOW_CHANGES.md#restrictions)

* Выполнение запроса недоступно при наличии незавершенного запроса на создание, удаление или изменение таблицы или представления.

## [ALTER VIEW](../reference/sql_plus_requests/ALTER_VIEW/ALTER_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Подзапрос `query` не может содержать:
  * логические представления,
  * [системные представления](../reference/system_views/system_views.md) INFORMATION_SCHEMA,
  * ключевое слово [FOR SYSTEM_TIME](../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time),
  * ключевое слово [DATASOURCE_TYPE](../reference/sql_plus_requests/SELECT/SELECT.md#param_datasource_type).

## [BEGIN DELTA](../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md#restrictions)

* Выполнение запроса невозможно при наличии незавершенного запроса на создание, удаление или изменение таблицы или представления.
* Если в запросе указан номер открываемой дельты, он должен быть равен номеру последней закрытой дельты + 1.

## [CHECK_DATA](../reference/sql_plus_requests/CHECK_DATA/CHECK_DATA.md#restrictions)

* Существует вероятность совпадения контрольных сумм для разных наборов записей, поэтому
  возможен ложноположительный результат проверки.
* Количество проверяемых записей в одной сущности ограничено и регулируется коэффициентом нормализации. Если количество
  загруженных записей какой-либо сущности в указанной дельте больше `4'294'967'298`, нужно подобрать подходящее значение
  коэффициента нормализации.

## [CHECK_SUM](../reference/sql_plus_requests/CHECK_SUM/CHECK_SUM.md#restrictions)

* Контрольная сумма логической базы данных рассчитывается только по данным логических таблиц и не учитывает данные
  материализованных представлений.
* Существует вероятность совпадения контрольных сумм для разных наборов данных.
* Количество проверяемых записей в одной сущности ограничено и регулируется коэффициентом нормализации. Если количество
  загруженных записей какой-либо сущности в указанной дельте больше `4'294'967'298`, нужно подобрать подходящее значение
  коэффициента нормализации.

## [CHECK_SUM_SNAPSHOT](../reference/sql_plus_requests/CHECK_SUM_SNAPSHOT/CHECK_SUM_SNAPSHOT.md#restrictions)

* Контрольная сумма логической базы данных рассчитывается только по данным логических таблиц и не учитывает данные
  материализованных представлений.
* Существует вероятность совпадения контрольных сумм для разных наборов данных.
* Количество проверяемых записей в одной сущности ограничено и регулируется коэффициентом нормализации. Если количество
  актуальных записей какой-либо сущности в указанной дельте больше `4'294'967'298`, нужно подобрать подходящее значение
  коэффициента нормализации.

## [COMMIT DELTA](../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md#restrictions)

* Если в запросе указаны дата и время закрытия дельты, они должны быть больше, чем дата и время последней закрытой дельты 
  логической базы данных. Дату и время последней закрытой дельты можно узнать, выполнив запрос 
  [GET_DELTA_OK](../reference/sql_plus_requests/GET_DELTA_OK/GET_DELTA_OK.md).

## [CREATE DATABASE](../reference/sql_plus_requests/CREATE_DATABASE/CREATE_DATABASE.md#restrictions)

* Недоступно создание логической базы данных с именем `INFORMATION_SCHEMA`, зарезервированным для сервисной БД.

## [CREATE DOWNLOAD EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [CREATE MATERIALIZED VIEW](../reference/sql_plus_requests/CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Имена представления и его столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md)
  и должны начинаться с латинской буквы. После первого символа могут следовать латинские буквы, цифры и символы
  подчеркивания в любом порядке.
* Столбцы не могут иметь имена, зарезервированные системой для служебного использования: `sys_op`, `sys_from`, 
    `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
* Имена столбцов должны быть уникальны в рамках представления.
* Имена, порядок и типы данных столбцов должны совпадать в SELECT-подзапросе и представлении.
* Первичный ключ должен включать все столбцы ключа шардирования.
* Логические и readable-таблицы, указанные в SELECT-подзапросе, должны принадлежать той же логической базе данных,
  что и материализованное представление.
* Подзапрос не может содержать: 
  * ключевое слово [FOR SYSTEM_TIME](../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time),
  * ключевое слово `ORDER BY`,
  * ключевое слово `LIMIT`.

## [CREATE READABLE EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md#restrictions)

* Имена таблицы и ее столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md) и
  должны начинаться с латинской буквы. После первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания.
* Имена и порядок столбцов должны совпадать во внешней таблице и связанной standalone-таблице.
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [CREATE TABLE](../reference/sql_plus_requests/CREATE_TABLE/CREATE_TABLE.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md), 
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* Имена столбцов должны быть уникальны в рамках логической таблицы.
* Столбцы не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`, `sys_to`, 
  `sys_close_date`, `bucket_id`, `sign`.
* Первичный ключ должен включать все столбцы ключа шардирования.

## [CREATE UPLOAD EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_UPLOAD_EXTERNAL_TABLE/CREATE_UPLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [CREATE VIEW](../reference/sql_plus_requests/CREATE_VIEW/CREATE_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.
* В подзапросе `query` не допускается использование:
  * логических представлений,
  * [системных представлений](../reference/system_views/system_views.md) `INFORMATION_SCHEMA`,
  * ключевого слова [FOR SYSTEM_TIME](../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time).
* Ключевое слово `DATASOURCE_TYPE`, указанное в подзапросе `query`, игнорируется.

## [CREATE WRITABLE EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_WRITEABLE_EXTERNAL_TABLE/CREATE_WRITEABLE_EXTERNAL_TABLE.md#restrictions)

* Имена таблицы и ее столбцов не могут быть из числа [зарезервированных слов](../../reserved_words/reserved_words.md) и
  должны начинаться с латинской буквы. После первого символа могут следовать
  латинские буквы, цифры и символы подчеркивания.
* Имена и порядок столбцов должны совпадать во внешней таблице и связанной standalone-таблице.
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DELETE](../reference/sql_plus_requests/DELETE/DELETE.md#restrictions)

* Удаление записей из логических таблиц возможно только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* В блоке `WHERE` не допускается использование функций, которые приводят к разным результатам в разных СУБД хранилища.
  Примеры таких функций — это операции над числами с плавающей запятой: сравнение с ними, округление и т.д.
* Не допускается параллельное выполнение идентичных запросов.

## [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md#restrictions)

* Выполнение запроса недоступно при наличии другого запрета изменений или незавершенного запроса на создание, удаление или изменение таблицы или представления.

## [DROP DATABASE](../reference/sql_plus_requests/DROP_DATABASE/DROP_DATABASE.md#restrictions)

* Недоступно удаление сервисной базы данных `INFORMATION_SCHEMA`.

## [DROP DOWNLOAD_EXTERNAL_TABLE](../reference/sql_plus_requests/DROP_DOWNLOAD_EXTERNAL_TABLE/DROP_DOWNLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP MATERIALIZED VIEW](../reference/sql_plus_requests/DROP_MATERIALIZED_VIEW/DROP_MATERIALIZED_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей дельты,
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP READABLE EXTERNAL TABLE](../reference/sql_plus_requests/DROP_READABLE_EXTERNAL_TABLE/DROP_READABLE_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP TABLE](../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей дельты,
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP UPLOAD EXTERNAL TABLE](../reference/sql_plus_requests/DROP_UPLOAD_EXTERNAL_TABLE/DROP_UPLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP_VIEW](../reference/sql_plus_requests/DROP_VIEW/DROP_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей дельты,
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP WRITABLE EXTERNAL TABLE](../reference/sql_plus_requests/DROP_WRITEABLE_EXTERNAL_TABLE/DROP_WRITEABLE_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [INSERT INTO download external table](../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md#restrictions)

* Имена и порядок следования столбцов должны совпадать в SELECT-подзапросе на выгрузку данных и внешней таблице выгрузки.
* Выгрузка данных, выбранных с использованием агрегатных функций, из ADQM дает некорректные результаты. Ограничение 
  связано с тем, что данные из сегментов кластера ADQM выгружаются параллельно и не объединяются.

## [INSERT FROM upload_external_table](../reference/sql_plus_requests/INSERT_FROM_upload_external_table/INSERT_FROM_upload_external_table.md#restrictions)

* Загрузка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).

## [INSERT SELECT](../reference/sql_plus_requests/INSERT_SELECT/INSERT_SELECT.md#restrictions)

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Типы вставляемых данных должны соответствовать типам данных столбцов в целевой логической таблице.
* Не допускается параллельное выполнение идентичных запросов.

## [INSERT VALUES](../reference/sql_plus_requests/INSERT_VALUES/INSERT_VALUES.md#restrictions)

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Не допускается параллельное выполнение идентичных запросов.

## [SELECT](../reference/sql_plus_requests/SELECT/SELECT.md#restrictions)

*   Запрос может обращаться либо к логической БД, либо к сервисной БД (см. [SELECT FROM INFORMATION_SCHEMA](../reference/sql_plus_requests/SELECT_FROM_INFORMATION_SCHEMA/SELECT_FROM_INFORMATION_SCHEMA.md)), 
    но не к обеим одновременно.
*   Если ключами соединения в запросе выступают поля типа Nullable, то строки, где хотя бы один из ключей 
    имеет значение NULL, не соединяются.
*   Ключевое слово `ORDER BY` не поддерживается для SELECT-подзапроса в составе запроса 
    [CREATE MATERIALIZED VIEW](../reference/sql_plus_requests/CREATE_MATERIALIZED_VIEW/CREATE_MATERIALIZED_VIEW.md). 

## [SELECT FROM INFORMATION_SCHEMA](../reference/sql_plus_requests/SELECT_FROM_INFORMATION_SCHEMA/SELECT_FROM_INFORMATION_SCHEMA.md#restrictions)

* Не допускается комбинирование подзапросов к `INFORMATION_SCHEMA` с подзапросами к логическим базам данных.

## [TRUNCATE_HISTORY](../reference/sql_plus_requests/TRUNCATE_HISTORY/TRUNCATE_HISTORY.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [UPSERT VALUES](../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md#restrictions)

* Вставка данных в логическую таблицу возможна только при наличии открытой дельты (см. [BEGIN DELTA](../BEGIN_DELTA/BEGIN_DELTA.md)).
* Не допускается параллельное выполнение идентичных запросов.