---
layout: default
title: История изменений
nav_order: 7
has_children: false
---

# История изменений {#revision_history}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

## Текущая версия документации (5.5) {#current}

Изменения:
* добавлена возможность работать со [standalone-таблицами](../overview/main_concepts/standalone_table/standalone_table.md) — 
  таблицами, которые не относятся к логической и физической схемам данных;
* добавлены новые сущности, предназначенные для работы со standalone-таблицами: 
  [внешняя writable-таблица](../overview/main_concepts/external_table/external_table.md#writable_table) и 
  [внешняя readable-таблица](../overview/main_concepts/external_table/external_table.md#readable_table);
* в запрос [CREATE UPLOAD EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_UPLOAD_EXTERNAL_TABLE/CREATE_UPLOAD_EXTERNAL_TABLE.md) 
  добавлено ключевое слово `OPTIONS`; его значение `auto.create.sys_op.enable=false`
  позволяет создать таблицу, предназначенную для загрузки в standalone-таблицу;
* изменен способ возобновления [операций](../overview/main_concepts/write_operation/write_operation.md) 
  по [обновлению данных](../working_with_system/data_update/data_update.md): 
  теперь нужно повторить исходный запрос, добавив в его начало 
  ключевое слово `RETRY`, а повторение запроса без ключевого слова не возобновляет обработку операции;
* добавлены новые запросы:
  * запросы по управлению внешними writable- и readable-таблицами:
    * [CREATE WRITABLE EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_WRITEABLE_EXTERNAL_TABLE/CREATE_WRITEABLE_EXTERNAL_TABLE.md);
    * [DROP WRITABLE EXTERNAL TABLE](../reference/sql_plus_requests/DROP_WRITEABLE_EXTERNAL_TABLE);
    * [CREATE READABLE EXTERNAL TABLE](../reference/sql_plus_requests/CREATE_READABLE_EXTERNAL_TABLE/CREATE_READABLE_EXTERNAL_TABLE.md);
    * [DROP READABLE EXTERNAL TABLE](../reference/sql_plus_requests/DROP_READABLE_EXTERNAL_TABLE/DROP_READABLE_EXTERNAL_TABLE.md);
  * [ERASE_CHANGE_OPERATION](../reference/sql_plus_requests/ERASE_CHANGE_OPERATION/ERASE_CHANGE_OPERATION.md);
  * [ERASE_WRITE_OPERATION](../reference/sql_plus_requests/ERASE_WRITE_OPERATION/ERASE_WRITE_OPERATION.md);
* запрос `INSERT INTO logical_table` переименован в справочнике запросов в
  [INSERT SELECT FROM upload_external_table](../reference/sql_plus_requests/INSERT_SELECT_FROM_upload_external_table/INSERT_SELECT_FROM_upload_external_table.md):
  переименование связано с тем, что теперь запрос позволяет загружать данные не только в логические таблицы, но и 
  в standalone-таблицы; синтаксис запроса не изменился, изменилось только обозначение запроса в документе;
* в [конфигурацию системы](../maintenance/configuration/system/system.md) добавлены параметры `TARANTOOL_DB_SYNC_BUFFER_SIZE` и 
`ADQM_BUFFER_SIZE`;
* описан [формат служебного топика Kafka](../reference/system_topic_format/system_topic_format.md).
  
## Архивные версии документации {#archive}

### Версия 5.4

Версия 5.4 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v5-4/).

Изменения:
* начало строки подключения изменено с `jdbc:adtm` на `jdbc:prostore`;
* запрос `UPSERT VALUES` переименован в `INSERT VALUES`, запрос `UPSERT SELECT` — в `INSERT SELECT`;
* добавлены новые запросы:
  * `UPSERT VALUES`;
  * `CHECK_SUM_SNAPSHOT`;
* обновлена `конфигурация системы`:
  * добавлен параметр `KAFKA_STATUS_EVENT_TOPIC`;
  * изменены значения по умолчанию:
    * параметр `ADQM_DB_NAME` теперь имеет значение по умолчанию `default`;
    * параметр `ADQM_CLUSTER` теперь имеет значение по умолчанию `default_cluster`;
    * значения параметра `ADQM_SHARDING_EXPR` изменены с `cityHash64` и `intAdd` на `CITY_HASH_64` и `INT_ADD` соответственно;
  * начало путей `io.arenadata` во всех вхождениях заменено на `ru.datamart`;
  * добавлены неучтенные ранее параметры для ADG: `TARANTOOL_VERTX_WORKERS`, `TARANTOOL_DB_SYNC_CONNECTION_TIMEOUT`,
    `TARANTOOL_DB_SYNC_READ_TIMEOUT` и `TARANTOOL_DB_SYNC_REQUEST_TIMEOUT`;
* добавлено ограничение на имена логических сущностей и их столбцов: имя должно начинаться с латинской буквы,
  после первого символа могут следовать латинские буквы, цифры и символы подчеркивания в любом порядке;
* изменен ответ `GET_CHANGES` в случае отсутствия журнала: теперь возвращается пустой объект ResultSet, а не ошибка.

### Версия 5.3

Версия 5.3 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v5-3/).

Изменения:
* добавлено создание материализованных представлений в ADQM на основе данных ADB;
* добавлена вставка данных из ADB в ADQM с помощью UPSERT SELECT;
* удалено требование на целочисленные ключи шардирования в логических таблицах: теперь ключ может содержать столбцы
  с любыми типами данных;
* добавлены новые запросы:
  * CHECK_MATERIALIZED_VIEW;
  * DENY_CHANGES;
  * ALLOW_CHANGES;
  * GET_CHANGES;
  * GET_ENTITY_DDL;
* в системное представление `tables` добавлен новый тип сущности — `MATERIALIZED VIEW`;
* добавлен перезапуск незавершенной операции по обновлению данных;
* добавлено автоматическое ведение журнала — списка операций по изменению логических сущностей;
* для выгрузки данных добавлен выбор оптимальной СУБД хранилища,
  аналогичный выбору СУБД для SELECT-запросов;
* из описания запроса `CREATE TABLE` удалено неподдерживаемое ключевое слово `DEFAULT`;
* добавлен раздел «Зарезервированные слова» со словами, которые не могут использоваться как имена сущностей
  и имена полей;
* добавлен раздел «Ограничения системы» со списком всех ограничений, имеющихся в запросах системы;
* обновлена конфигурация системы:
  * добавлен параметр `ADQM_SHARDING_EXPR`;
  * добавлен параметр `ADB_MPPW_USE_ADVANCED_CONNECTOR`;
  * удален параметр `EDML_DATASOURCE`;
  * исправлен путь до параметра `ADB_WITH_HISTORY_TABLE` с `adb:mppw:with-history-table` на `adb:with-history-table`;
  * исправлен путь до параметров `ADG_MAX_MSG_PER_PARTITION` и `ADG_CB_FUNC_IDLE`: из пути удален параметр `kafka`;
* изменена терминология: архивация актуальных записей теперь называется удалением, а удаление записей
  с помощью запроса TRUNCATE HISTORY — удалением записей с историей.

### Версия 5.2

Версия 5.2 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v5-2/).

Изменения:
* добавлена функция обновления данных — альтернатива загрузке в случае небольших объемов данных; описание доступно
  в следующие разделах:
  * «Обновление данных»;
  * «Порядок обработки запросов на обновление данных»;
  * UPSERT VALUES;
  * UPSERT SELECT;
  * DELETE;
* добавлены новые запросы:
  * `CONFIG_SHOW`;
  * `GET_WRITE_OPERATIONS`;
  * `RESUME_WRITE_OPERATION`;
* добавлено ключевое слово `COLLATE`, доступное в SELECT-запросах к ADG;
* добавлена возможность выгрузки данных из материализованных представлений;
* изменена маршрутизация SELECT-запросов: теперь учитывается не только категория запроса, но и набор узлов кластера, 
  для которых предназначен запрос;
* добавлен раздел «Разбор ошибок загрузки и обновления данных»;
* ограничено исполнение запросов по управлению схемой данных в сервисной базе данных `INFORMATION_SCHEMA`;
* изменен перечень операций, отменяемых запросом `ROLLBACK DELTA`: отменяются все завершенные операции (как операции 
  загрузки данных, так и обновления данных), а также незавершенные операции загрузки данных; незавершенные операции 
  обновления данных не отменяются;
* обновлена конфигурация системы:
  * добавлены параметры `AUTO_RESTORE_STATE`, `ADB_MAX_RECONNECTIONS`, `ADB_QUERIES_BY_CONNECT_LIMIT` и
    `ADB_RECONNECTION_INTERVAL`;
  * добавлена секция параметров `autoSelect` для настройки порядка выбора СУБД в зависимости от категории и
    подкатегории запросов;
  * удален параметр `CORE_TIME_ZONE` (больше не используется);
  * путь к параметру `DTM_METRICS_PORT` изменен с `management.server.port` на `server.port`;
  * путь к параметру `DTM_CORE_METRICS_ENABLED` изменен c `core.metrics.isEnabled` на `core.metrics.еnabled`;
* добавлен раздел «Конфигурация коннекторов»;
* описание конфигурационных параметров системы перенесено из раздела «Конфигурация» в раздел
  «Конфигурация системы»;
* имя системы заменено на `Prostore` (в соответствии с именем проекта с открытым исходным кодом);
* скорректировано описание служебного поля `sys_op`: поле должно отсутствовать во внешней таблице загрузки и логической
  таблице и должно присутствовать в загружаемых сообщениях топика Kafka.

### Версия 5.1

Версия 5.1 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v5-1-0/).

Изменения:
* добавлено ключевое слово `ESTIMATE_ONLY`, доступное в SELECT-запросах;
* добавлено ключевое слово `LOGICAL_ONLY`, доступное в запросах на создание и удаление логической БД,
  логической таблицы и материализованного представления;
* обновлено описание запросов `CHECK_DATA` и `CHECK_SUM`:
  * добавлен коэффициент нормализации, повышающий максимально допустимое количество записей в
    проверяемых дельтах;
  * изменен расчет контрольных сумм: теперь они считаются по дельтам, а не отдельным операциям записи;
* обновлено описание запроса `CHECK_SUM`:
  * изменен расчет контрольной суммы по таблице/представлению: теперь расчет аналогичен тому, который
    выполняется для CHECK_DATA;
  * изменен расчет контрольной суммы по логической БД: теперь контрольные суммы таблиц складываются,
    а не проходят дополнительный этап хеширования;
* в конфигурацию добавлен параметр `DTM_VERTX_BLOCKING_STACKTRACE_TIME`;
* добавлена глава «Сборка и развертывание»;
* в главу «Работа с системой» добавлены разделы «Получение информации о SELECT-запросе» и «Проверка месторасположения 
  логической сущности»;
* в главу «Эксплуатация» добавлен раздел «Часовые пояса системы и компонентов».

### Версия 5.0

Изменения:
* добавлена СУБД хранилища нового типа — ADP — на основе PostgreSQL;
* добавлена выгрузка данных из СУБД хранилища, указанной в запросе `INSERT INTO download_external_table`;
* в системное представление `tables` добавлен столбец `table_datasource_type`;
* обновлено описание запроса `CHECK_SUM`: теперь запрос поддерживает расчет контрольной суммы по материализованному 
  представлению;
* обновлена конфигурация:
  * добавлены параметры для управления СУБД ADP;
  * добавлены параметры запроса prepared statement для ADB: `ADB_PREPARED_CACHE_MAX_SIZE`, `ADB_PREPARED_CACHE_SQL_LIMIT`
    и `ADB_PREPARED_CACHE`;
  * значения следующих параметров расширены новой СУБД ADP: `CORE_PLUGINS_ACTIVE`, `DTM_CORE_PLUGINS_RELATIONAL`,
    `DTM_CORE_PLUGINS_ANALYTICAL`, `DTM_CORE_PLUGINS_DICTIONARY`, `DTM_CORE_PLUGINS_UNDEFINED`;
  * добавлен параметр `DTM_LOGGING_LEVEL` для управления уровнем логирования;
  * конкретные IP-адреса заменены на `localhost`;
* добавлен раздел «Схемы развертывания».

### Версия 4.1

Версия 4.1 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v4-1-0/).

Изменения:
* добавлено ключевое слово `OFFSET`, доступное в SELECT-запросах;
* добавлено ключевое слово `FETCH NEXT <N> ROWS ONLY` как полная альтернатива ключевому слову `LIMIT <N>`
  в SELECT-запросах;
* обновлено описание запроса ROLLBACK DELTA: теперь запрос отменяет как завершенные, так и выполняемые операции записи;
* обновлена конфигурация:
  * значение параметра `ADB_EXECUTORS_COUNT` изменено с 20 на 3;
  * значение параметра `ADB_MAX_POOL_SIZE` изменено с 5 на 3;
  * добавлен новый параметр `DELTA_ROLLBACK_STATUS_CALLS_MS`.

### Версия 4.0

Изменения:
* описаны материализованные представления;
* описаны возможные форматы даты и времени в запросах;
* добавлен раздел «Проверка наличия логической сущности»;
* добавлен раздел «Настройка JSON-логов»;
* в конфигурацию добавлены параметры по управлению материализованными представлениями:
  * `MATERIALIZED_VIEWS_SYNC_PERIOD_MS`,
  * `MATERIALIZED_VIEWS_RETRY_COUNT`,
  * `MATERIALIZED_VIEWS_RETRY_COUNT`.

### Версия 3.7.3

Версия 3.7.3 доступна в [архиве](https://datamarts.github.io/docs_prostore_archive/v3-7-3/).

Изменения:
* обновлена конфигурация:
  * в секцию `vertx.pool` добавлены параметры `DTM_CORE_WORKER_POOL_SIZE` и `DTM_CORE_EVENT_LOOP_POOL_SIZE`;
  * путь к параметру `ADB_MAX_POOL_SIZE` изменился с `adb.maxSize` на `adb.poolSize`;
  * в секцию `adb` добавлен параметр `ADB_EXECUTORS_COUNT`;
* описан запрос ROLLBACK CRASHED_WRITE_OPERATIONS;
* доработаны разделы CHECK_DATA и CHECK_SUM: описаны алгоритм и пример расчета контрольной суммы;
* уточнено описание формата загрузки и формата выгрузки данных;
* в разделе «Минимальные системные требования» версия ADG обновлена до 2.7.2.
