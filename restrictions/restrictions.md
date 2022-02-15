---
layout: default
title: Ограничения Prostore
nav_order: 5.5
has_children: false
has_toc: false
---

# Ограничения Prostore {#prostore_limitations}
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

*   Существует математическая вероятность получения одинаковых контрольных сумм для разных наборов записей, поэтому 
    возможен ложноположительный результат проверки.
*   Максимальное количество записей в дельте, для которых контрольная сумма рассчитывается корректно, 
    пропорционально коэффициенту нормализации. Если коэффициент не указан или равен 1, проверяемая дельта может 
    содержать до `4'294'967'298` записей; при увеличении коэффициента допустимое количество записей также увеличивается.

## [CHECK_SUM](../reference/sql_plus_requests/CHECK_SUM/CHECK_SUM.md#restrictions)

*   Контрольная сумма логической базы данных рассчитывается только по данным логических таблиц и не учитывает данные 
    материализованных представлений.
*   Существует математическая вероятность получения одинаковых контрольных сумм для разных наборов данных.
*   Максимально допустимое количество записей таблицы (или представления) в дельте, для которых контрольная 
    сумма рассчитывается корректно, пропорционально коэффициенту нормализации. Если коэффициент не указан или равен 1,
    каждая из проверяемых таблиц или представлений может содержать до `4'294'967'298` записей в дельте; 
    при увеличении коэффициента допустимое количество записей также увеличивается.

## [COMMIT DELTA](../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md#restrictions)

* Если в запросе указаны дата и время закрытия дельты, они должны быть больше, чем дата и время последней закрытой дельты. Дату и время последней закрытой дельты можно узнать, выполнив запрос 
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
* Имена столбцов должны быть уникальны в рамках представления.
* Столбцы не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`, 
    `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
* Имена, порядок и типы данных столбцов должны совпадать в SELECT-подзапросе и представлении.
* Первичный ключ должен включать все столбцы ключа шардирования.
* Подзапрос может обращаться только к [логическим таблицам](../overview/main_concepts/logical_table/logical_table.md) 
  и только той логической базы данных, в которой находится материализованное представление.
* Подзапрос не может содержать: 
  * ключевое слово [FOR SYSTEM_TIME](../reference/sql_plus_requests/SELECT/SELECT.md#for_system_time),
  * ключевое слово `ORDER BY`,
  * ключевое слово `LIMIT`.

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

## [DELETE](../reference/sql_plus_requests/DELETE/DELETE.md#restrictions)

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)).
* Столбцы в условии запроса не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`,
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`. 
* В условии `WHERE` не допускается использование функций, результаты которых различаются в разных СУБД хранилища. Примерами 
  таких функций служат операции с вещественными числами (числами с плавающей запятой): сравнение с вещественным числом, 
  округление и т.д. 

## [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md#restrictions)

* Выполнение запроса недоступно при наличии другого запрета изменений или незавершенного запроса на создание, удаление или изменение таблицы или представления.

## [DROP DATABASE](../reference/sql_plus_requests/DROP_DATABASE/DROP_DATABASE.md#restrictions)

* Недоступно удаление сервисной базы данных `INFORMATION_SCHEMA`.

## [DROP DOWNLOAD_EXTERNAL_TABLE](../reference/sql_plus_requests/DROP_DOWNLOAD_EXTERNAL_TABLE/DROP_DOWNLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP MATERIALIZED VIEW](../reference/sql_plus_requests/DROP_MATERIALIZED_VIEW/DROP_MATERIALIZED_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP TABLE](../reference/sql_plus_requests/DROP_TABLE/DROP_TABLE.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP UPLOAD EXTERNAL TABLE](../reference/sql_plus_requests/DROP_UPLOAD_EXTERNAL_TABLE/DROP_UPLOAD_EXTERNAL_TABLE.md#restrictions)

* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [DROP_VIEW](../reference/sql_plus_requests/DROP_VIEW/DROP_VIEW.md#restrictions)

* Выполнение запроса недоступно при наличии любого из факторов:
  * горячей [дельты](../overview/main_concepts/delta/delta.md),
  * незавершенного запроса на создание, удаление или изменение таблицы или представления,
  * запрета на изменение сущностей (см. раздел [DENY_CHANGES](../reference/sql_plus_requests/DENY_CHANGES/DENY_CHANGES.md)).
* Выполнение запроса недоступно в сервисной базе данных `INFORMATION_SCHEMA`.

## [INSERT INTO download external table](../reference/sql_plus_requests/INSERT_INTO_download_external_table/INSERT_INTO_download_external_table.md#restrictions)

* Имена и порядок следования столбцов должны совпадать в SELECT-подзапросе на выгрузку данных и внешней таблице выгрузки.
* Выгрузка данных, выбранных с использованием агрегатных функций, из ADQM дает некорректные результаты. Ограничение 
  связано с тем, что данные из сегментов кластера ADQM выгружаются параллельно и не объединяются.

## [INSERT INTO logical table](../reference/sql_plus_requests/INSERT_INTO_logical_table/INSERT_INTO_logical_table.md#restrictions)

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)).

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

## [UPSERT SELECT](../reference/sql_plus_requests/UPSERT_SELECT/UPSERT_SELECT.md#restrictions)

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)).
* Столбцы в запросе не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`,
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
* Типы вставляемых данных должны соответствовать типам данных столбцов целевой логической таблицы.

## [UPSERT VALUES](../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md#restrictions)

* Выполнение запроса возможно только при наличии открытой дельты (см. [BEGIN DELTA](../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md)).
* Столбцы в запросе не могут иметь имена, зарезервированные для служебного использования: `sys_op`, `sys_from`,
  `sys_to`, `sys_close_date`, `bucket_id`, `sign`.
