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

## Текущая версия документации (5.2) {#current}

Изменения:
* добавлена функция обновления данных — альтернатива загрузке в случае небольших объемов данных; описание доступно 
в следующие разделах:
  * [Обновление данных](../working_with_system/data_update/data_update.md);
  * [Порядок обработки запросов на обновление данных](../overview/interactions/llw_processing/llw_processing.md);
  * [UPSERT VALUES](../reference/sql_plus_requests/UPSERT_VALUES/UPSERT_VALUES.md);
  * [UPSERT SELECT](../reference/sql_plus_requests/UPSERT_SELECT/UPSERT_SELECT.md);
  * [DELETE](../reference/sql_plus_requests/DELETE/DELETE.md);
* добавлены новые запросы:
  * [CONFIG_SHOW](../reference/sql_plus_requests/CONFIG_SHOW/CONFIG_SHOW.md);
  * [GET_WRITE_OPERATIONS](../reference/sql_plus_requests/GET_WRITE_OPERATIONS/GET_WRITE_OPERATIONS.md);
  * [RESUME_WRITE_OPERATION](../reference/sql_plus_requests/RESUME_WRITE_OPERATION/RESUME_WRITE_OPERATION.md);
* добавлено ключевое слово [COLLATE](../reference/sql_plus_requests/SELECT/SELECT.md#collate), 
  доступное в [SELECT](../reference/sql_plus_requests/SELECT/SELECT.md)-запросах к ADG;
* добавлена возможность [выгрузки данных](../working_with_system/data_download/data_download.md) 
  из материализованных представлений;
* изменена [маршрутизация SELECT-запросов](../working_with_system/data_reading/routing/routing.md): теперь учитывается 
  не только категория запроса, но и для скольки узлов кластера предназначен запрос;
* добавлен раздел [Разбор ошибок загрузки и обновления данных](../working_with_system/other_features/troubleshooting/troubleshooting.md);
* ограничено исполнение [запросов по управлению схемой данных](../working_with_system/logical_schema_update/logical_schema_update.md) 
  в сервисной базе данных `INFORMATION_SCHEMA`;
* изменен перечень операций, отменяемых запросом [ROLLBACK DELTA](../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md): 
  отменяются все завершенные операции (как операции загрузки данных, так и обновления данных), а также 
  незавершенные операции загрузки данных; незавершенные операции обновления данных не отменяются;
* обновлена [конфигурация системы](../maintenance/configuration/system/system.md):
  * добавлены параметры `AUTO_RESTORE_STATE`, `ADB_MAX_RECONNECTIONS`, `ADB_QUERIES_BY_CONNECT_LIMIT` и 
    `ADB_RECONNECTION_INTERVAL`;
  * добавлена секция параметров `autoSelect` для настройки порядка выбора СУБД в зависимости от категории и
    подкатегории запросов;
  * удален параметр `CORE_TIME_ZONE` (больше не используется);
  * путь к параметру `DTM_METRICS_PORT` изменен с `management.server.port` на `server.port`;
  * путь к параметру `DTM_CORE_METRICS_ENABLED` изменен c `core.metrics.isEnabled` на `core.metrics.еnabled`;
* добавлен раздел [Конфигурация коннекторов](../maintenance/configuration/connectors/connectors.md);
* описание конфигурационных параметров системы перенесено из раздела 
  [Конфигурация](../maintenance/configuration/configuration.md) в раздел 
  [Конфигурация системы](../maintenance/configuration/system/system.md);
* имя системы заменено на Prostore (имя проекта с открытым исходным кодом);
* скорректировано описание служебного поля `sys_op`: поле должно отсутствовать во внешней таблице загрузки и логической 
  таблице и должно присутствовать в загружаемых сообщениях топика Kafka.
  
## Архивные версии документации {#archive}

### Версия 5.1

Версия 5.1 доступна в [архиве](https://arenadata.github.io/docs_prostore_archive/v5-1-0/).

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

Версия 4.1 доступна в [архиве](https://arenadata.github.io/docs_prostore_archive/v4-1-0/).

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

Версия 3.7.3 доступна в [архиве](https://arenadata.github.io/docs_prostore_archive/v3-7-3/).

Изменения:
* обновлена конфигурация:
  * в секцию `vertx.pool` добавлены параметры `DTM_CORE_WORKER_POOL_SIZE` и `DTM_CORE_EVENT_LOOP_POOL_SIZE`;
  * путь к параметру `ADB_MAX_POOL_SIZE` изменился с `adb.maxSize` на `adb.poolSize`;
  * в секцию `adb` добавлен параметр `ADB_EXECUTORS_COUNT`;
* описан запрос ROLLBACK CRASHED_WRITE_OPERATIONS;
* доработаны разделы CHECK_DATA и CHECK_SUM: описаны алгоритм и пример расчета контрольной суммы;
* уточнено описание формата загрузки и формата выгрузки данных;
* в разделе «Минимальные системные требования» версия ADG обновлена до 2.7.2.