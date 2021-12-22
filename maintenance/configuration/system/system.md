---
layout: default
title: Конфигурация системы
nav_order: 1
parent: Конфигурация
grand_parent: Эксплуатация
has_children: false
---

# Конфигурация системы {#system}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Конфигурация системы задается в текстовом YAML-файле. Параметры конфигурации организованы 
в иерархическую древовидную структуру.

В разделе представлены примеры файлов конфигурации системы: [конфигурации сервиса исполнения 
запросов](#core_configuration) и [конфигурации сервиса мониторинга статусов Kafka](#status_monitor_configuration). 
Перед каждым параметром указан комментарий, поясняющий назначение этого параметра. Для наглядности конфигурация 
сервиса исполнения запросов разделена на отдельные секции.

## Конфигурация сервиса исполнения запросов {#core_configuration}

### Настройки журналирования {#logging_parameters}
```yaml
# настройки журналирования
logging:
# уровень важности сообщений, записываемых в лог-файл
  level:
    io.arenadata.dtm.query.execution: ${DTM_LOGGING_LEVEL:TRACE}
```

### Настройки управления Prostore {#prostore_parameters}
```yaml
# настройки сервера Prostore
server:
# номер порта сервиса метрик  
  port: ${DTM_METRICS_PORT:8080}
# настройки управления Prostore
management:
# настройки конечных точек Prostore
  endpoints:
# признак генерации метрик со стороны Prostore
    enabled-by-default: ${DTM_METRICS_ENABLED:true}
# настройки видимости метрик через веб-соединения
    web:
      exposure:
# состав метрик, видимых через веб-соединения
        include: ${DTM_METRICS_SCOPE:info, health, requests}
```

### Настройки сервиса исполнения запросов {#core_parameters}
```yaml
# настройки сервиса исполнения запросов
core:
# настройки плагинов
  plugins:
# список используемых СУБД
    active: ${CORE_PLUGINS_ACTIVE:ADG, ADB, ADP, ADQM}
# порядок выбора СУБД хранилища для исполнения запросов на чтение и выгрузку данных
    category:
# порядок выбора СУБД в зависимости от категории запроса; настройки используются, если отсутствует секция plugins.category.autoSelect      
      mapping:
# порядок для общих реляционных запросов
        RELATIONAL: ${DTM_CORE_PLUGINS_RELATIONAL:ADB, ADP, ADQM, ADG}
# порядок для аналитических запросов
        ANALYTICAL: ${DTM_CORE_PLUGINS_ANALYTICAL:ADQM, ADB, ADP, ADG}
# порядок для запросов ключ-значение
        DICTIONARY: ${DTM_CORE_PLUGINS_DICTIONARY:ADG, ADB, ADP, ADQM}
# порядок для других категорий запросов
        UNDEFINED:  ${DTM_CORE_PLUGINS_UNDEFINED:ADB, ADP, ADQM, ADG}        
# порядок выбора СУБД в зависимости от категории и подкатегории запроса   
      autoSelect:
# порядок для общих реляционных запросов        
        RELATIONAL:
# порядок для запросов, предназначенных для одного узла кластера        
          SHARD_ONE: ${DTM_CORE_PLUGINS_AUTOSELECT_RELATIONAL_SHARDONE:ADB, ADP, ADQM, ADG}
# порядок для запросов, предназначенных для нескольких узлов кластера (от 1 до всех)
          SHARD_SET: ${DTM_CORE_PLUGINS_AUTOSELECT_RELATIONAL_SHARDSET:ADB, ADP, ADQM, ADG}
# порядок для запросов, предназначенных для всех узлов кластера
          SHARD_ALL: ${DTM_CORE_PLUGINS_AUTOSELECT_RELATIONAL_SHARDALL:ADB, ADP, ADQM, ADG}
# порядок для аналитических запросов          
        ANALYTICAL:
# порядок для запросов, предназначенных для одного узла кластера 
          SHARD_ONE: ${DTM_CORE_PLUGINS_AUTOSELECT_ANALYTICAL_SHARDONE:ADQM, ADB, ADP, ADG}
# порядок для запросов, предназначенных для нескольких узлов кластера (от 1 до всех)
          SHARD_SET: ${DTM_CORE_PLUGINS_AUTOSELECT_ANALYTICAL_SHARDSET:ADQM, ADB, ADP, ADG}
# порядок для запросов, предназначенных для всех узлов кластера
          SHARD_ALL: ${DTM_CORE_PLUGINS_AUTOSELECT_ANALYTICAL_SHARDALL:ADQM, ADB, ADP, ADG}
# порядок для запросов ключ-значение          
        DICTIONARY:
# порядок для запросов, предназначенных для одного узла кластера 
          SHARD_ONE: ${DTM_CORE_PLUGINS_AUTOSELECT_DICTIONARY_SHARDONE:ADG, ADB, ADP, ADQM}
# порядок для запросов, предназначенных для нескольких узлов кластера (от 1 до всех)
          SHARD_SET: ${DTM_CORE_PLUGINS_AUTOSELECT_DICTIONARY_SHARDSET:ADG, ADB, ADP, ADQM}
# порядок для запросов, предназначенных для всех узлов кластера
          SHARD_ALL: ${DTM_CORE_PLUGINS_AUTOSELECT_DICTIONARY_SHARDALL:ADG, ADB, ADP, ADQM}
# порядок для других категорий запросов          
        UNDEFINED:
# порядок для запросов, предназначенных для одного узла кластера 
          SHARD_ONE: ${DTM_CORE_PLUGINS_AUTOSELECT_UNDEFINED_SHARDONE:ADB, ADP, ADQM, ADG}
# порядок для запросов, предназначенных для нескольких узлов кластера (от 1 до всех)
          SHARD_SET: ${DTM_CORE_PLUGINS_AUTOSELECT_UNDEFINED_SHARDSET:ADB, ADP, ADQM, ADG}
# порядок для запросов, предназначенных для всех узлов кластера
          SHARD_ALL: ${DTM_CORE_PLUGINS_AUTOSELECT_UNDEFINED_SHARDALL:ADB, ADP, ADQM, ADG}
  # настройки сетевых подключений через HTTP-протокол
  http:
# номер порта сервиса исполнения запросов
    port: ${DTM_CORE_HTTP_PORT:9090}
# режим оптимизации работы сокета TCP_NODELAY
    tcpNoDelay: ${DTM_CORE_HTTP_TCP_NO_DELAY:true}
# режим TCP FAST_OPEN
    tcpFastOpen: ${DTM_CORE_HTTP_TCP_FAST_OPEN:true}
# режим оптимизации работы сокета TCP_QUICKACK
    tcpQuickAck: ${DTM_CORE_HTTP_TCP_QUICK_ACK:true}
# настройки окружения
  env:
# имя окружения для формирования полных имен логических БД
    name: ${DTM_NAME:test}
# настройки восстановления состояния при запуске и перезапуске системы
  restoration:
# признак восстановления состояния при запуске и перезапуске системы   
    autoRestoreState: ${AUTO_RESTORE_STATE:true}    
# настройки синхронизации материализованных представлений
  matviewsync:
# периодичность запуска синхронизации в миллисекундах; если значение равно 0, синхронизация отключена
    periodMs: ${MATERIALIZED_VIEWS_SYNC_PERIOD_MS:5000}
# максимальное количество попыток синхронизации представления, после перезапуска системы счетчик обнуляется
    retryCount: ${MATERIALIZED_VIEWS_RETRY_COUNT:10}
# максимальное количество представлений, синхронизируемых одновременно
    maxConcurrent: ${MATERIALIZED_VIEWS_CONCURRENT:2}
# настройки генерации метрики сервиса исполнения запросов
  metrics:
# признак генерации метрик сервиса исполнения запросов
    enabled: ${DTM_CORE_METRICS_ENABLED:true}
# настройки источника данных
  datasource:
# настройки для EDML-операторов
    edml:
# количество записей, по умолчанию выгружаемых в одном сообщении топика Каfka
      defaultChunkSize: ${EDML_DEFAULT_CHUNK_SIZE:1000}
# период проверки статуса плагина в миллисекундах
      pluginStatusCheckPeriodMs: ${EDML_STATUS_CHECK_PERIOD_MS:1000}
# время ожидания (в миллисекундах) до тайм-аута при работе с первым смещением в топике Kafka
      firstOffsetTimeoutMs: ${EDML_FIRST_OFFSET_TIMEOUT_MS:15000}
# время ожидания (в миллисекундах) до тайм-аута при ожидании смены смещения в топике Kafka
      changeOffsetTimeoutMs: ${EDML_CHANGE_OFFSET_TIMEOUT_MS:10000}
# настройки Zookeeper
    zookeeper:
# сетевой адрес хоста Zookeeper для сервисной БД
      connection-string: ${ZOOKEEPER_DS_ADDRESS:localhost}
# время ожидания (в миллисекундах) соединения с хостом Zookeeper для сервисной БД до достижения тайм-аута
      connection-timeout-ms: ${ZOOKEEPER_DS_CONNECTION_TIMEOUT_MS:30000}
# время бездействия (в миллисекундах) в сессии хоста Zookeeper для сервисной БД до достижения тайм-аута
      session-timeout-ms: ${ZOOKEEPER_DS_SESSION_TIMEOUT_MS:86400000}
# корневой путь к хосту Zookeeper для сервисной БД
      chroot: ${ZOOKEEPER_DS_CHROOT:/adtm}
# настройки взаимодействия сервиса исполнения запросов с брокером сообщений Kafka
  kafka:
    producer:
      property:
# сериализатор строковых ключей
        key.serializer: org.apache.kafka.common.serialization.StringSerializer
# сериализатор строковых значений
        value.serializer: org.apache.kafka.common.serialization.StringSerializer
# настройки кластера Zookeeper для взаимодействия с брокером сообщений Kafka
    cluster:
      zookeeper:
# сетевой адрес хоста Zookeeper для брокера сообщений Kafka
        connection-string: ${ZOOKEEPER_KAFKA_ADDRESS:localhost}
# время ожидания (в миллисекундах) соединения с хостом Zookeeper для брокера сообщений Kafka до достижения тайм-аута
        connection-timeout-ms: ${ZOOKEEPER_KAFKA_CONNECTION_TIMEOUT_MS:30000}
# время бездействия (в миллисекундах) в сессии хоста Zookeeper для брокера сообщений Kafka до достижения тайм-аута
        session-timeout-ms: ${ZOOKEEPER_KAFKA_SESSION_TIMEOUT_MS:86400000}
# корневой путь к хосту Zookeeper для брокера сообщений Kafka
        chroot: ${ZOOKEEPER_KAFKA_CHROOT:}
# настройки администратора Kafka
    admin:
# время ожидания (в миллисекундах) входного потока данных для брокера сообщений Kafka до достижения тайм-аута
      inputStreamTimeoutMs: ${KAFKA_INPUT_STREAM_TIMEOUT_MS:2000}
# настройки статусов публикации событий брокером сообщений Kafka
    status.event.publish:
# разрешение на публикацию событий
      enabled: ${KAFKA_STATUS_EVENT_ENABLED:false}
# имя топика Kafka, в который публикуются события
      topic: ${KAFKA_STATUS_EVENT_TOPIC:status.event}
# настройки подключения к сервису мониторинга статусов Kafka 
    statusMonitor:
# сетевой адрес и путь для получения информации о статусе сервиса
      statusUrl: ${STATUS_MONITOR_URL:http://localhost:9095/status}
# сетевой адрес и путь для получения информации о версии сервиса
      versionUrl: ${STATUS_MONITOR_VERSION_URL:http://localhost:9095/versions}
# настройки при использовании фреймворка vertx
  vertx:
# время в (секундах), после которого заблокированный поток пишет stacktrace
    blocking-stacktrace-time: ${DTM_VERTX_BLOCKING_STACKTRACE_TIME:1}
    pool:
# максимальный размер пула потоков, выполняющих долгие операции
      worker-pool: ${DTM_CORE_WORKER_POOL_SIZE:20}
# максимальный размер пула потоков, обрабатывающих события vertx
      event-loop-pool: ${DTM_CORE_EVENT_LOOP_POOL_SIZE:20}
# максимальный размер пула задач в сервисе исполнения запросов
      task-pool: ${DTM_CORE_TASK_POOL_SIZE:20}
# время (в миллисекундах) завершения задачи, выполняемой в сервисе исполнения запросов
      task-timeout: ${DTM_CORE_TASK_TIMEOUT:86400000}
# настройки кэширования запросов
  cache:
# начальная емкость кэша
    initialCapacity: ${CACHE_INITIAL_CAPACITY:100000}
# максимальный размер кэша
    maximumSize: ${CACHE_MAXIMUM_SIZE:100000}
# время (в минутах) устаревания кэша после последнего обращения к нему
    expireAfterAccessMinutes: ${CACHE_EXPIRE_AFTER_ACCESS_MINUTES:99960}
# настройки отката дельты
  delta:
# периодичность (в миллисекундах) проверки операций записи, требующих остановки
    rollback-status-calls-ms: ${DELTA_ROLLBACK_STATUS_CALLS_MS:2000}
```

### Настройки СУБД ADB {#adb_parameters}
```yaml
# настройки ADB
adb:
# настройки источника данных ADB
  datasource:
# имя пользователя/логин для авторизации в ADB
    user: ${ADB_USERNAME:dtm}
# пароль для авторизации в ADB
    password: ${ADB_PASS:dtm}
# сетевой адрес хоста с ADB
    host: ${ADB_HOST:localhost}
# сетевой адрес порта на хосте с ADB
    port: ${ADB_PORT:5432}
# лимит подключений к ADB в одном потоке; лимит по всем потокам равен произведению poolSize и executorsCount
    poolSize: ${ADB_MAX_POOL_SIZE:3}
# количество одновременных потоков, исполняющих запросы к ADB
    executorsCount: ${ADB_EXECUTORS_COUNT:3}
# максимальный размер результата, возвращаемого по FETCH-запросу к ADB
    fetchSize: ${ADB_FETCH_SIZE:1000}
# максимальный размер кэша запроса prepared statement
    preparedStatementsCacheMaxSize: ${ADB_PREPARED_CACHE_MAX_SIZE:256}
# максимальный размер запроса prepared statement, который может быть закэширован
    preparedStatementsCacheSqlLimit: ${ADB_PREPARED_CACHE_SQL_LIMIT:2048}
# признак кэширования запросов prepared statement
    preparedStatementsCache: ${ADB_PREPARED_CACHE:true}
# максимальное количество попыток восстановить соединение (0 — неограниченно)
    maxReconnections: ${ADB_MAX_RECONNECTIONS:0}
# максимальное количество запросов, проходящих через соединение перед его переподключением (0 - неограниченно)
    queriesByConnectLimit: ${ADB_QUERIES_BY_CONNECT_LIMIT:1000}
# интервал (в миллисекундах) между попытками восстановить соединение (0 — без интервала)
    reconnectionInterval: ${ADB_RECONNECTION_INTERVAL:5000}
# настройки механизма загрузки данных в ADB
  mppw:
# признак использования улучшенного PXF-коннектора; при значении false данные загружаются с помощью FDW-коннектора
    usePxfConnector: ${ADB_MPPW_USE_ADVANCED_CONNECTOR:false}
# имя консьюмер-группы ADB для взаимодействия с брокером сообщений Kafka
    consumerGroup: ${ADB_LOAD_GROUP:adb-emulator-load-adb}
# максимальный размер пула подключений к ADB для операций загрузки данных
    poolSize: ${ADB_MPPW_POOL_SIZE:2}
# время ожидания (в миллисекундах) до остановки загрузки
    stopTimeoutMs: ${ADB_MPPW_STOP_TIMEOUT_MS:86400000}
# максимальное количество сообщений для операций загрузки данных в ADB
    defaultMessageLimit: ${ADB_MPPW_DEFAULT_MESSAGE_LIMIT:100}
# время ожидания (в миллисекундах) для FDW-коннектора ADB
    fdwTimeoutMs: ${ADB_MPPW_FDW_TIMEOUT_MS:1000}
# признак использования исторических таблиц
  with-history-table: ${ADB_WITH_HISTORY_TABLE:false}
```

### Настройки СУБД ADG {#adg_parameters}
```yaml
# настройки ADG
adg:
  tarantool:
    db:
# сетевой адрес хоста с ADG
      host: ${TARANTOOL_DB_HOST:localhost}
# сетевой адрес порта на хосте с ADG
      port: ${TARANTOOL_DB_PORT:3306}
# имя пользователя/логин для авторизации в ADG
      user: ${TARANTOOL_DB_USER:admin}
# пароль для авторизации в ADG
      password: ${TARANTOOL_DB_PASS:memstorage-cluster-cookie}
# время ожидания (в миллисекундах) выполнения операции ADG
      operationTimeout: ${TARANTOOL_DB_OPER_TIMEOUT:60000}
# максимальное количество повторных попыток выполнения операции
      retryCount: ${TARANTOOL_DB_RETRY_COUNT:0}
# движок ADG
      engine: ${TARANTOOL_DEFAULT_ENGINE:MEMTX}
# настройки картриджа Tatantool
    cartridge:
# сетевой путь и порт к картриджу Tarantool
      url: ${TARANTOOL_CATRIDGE_URL:http://localhost:8086}
# настройки механизма загрузки данных
  mppw:
# имя консьюмер-группы ADG для взаимодействия с брокером сообщений Kafka
    consumerGroup: ${ADG_CONSUMER_GROUP:tarantool-group-csv}
# максимальное количество сообщений в топике Kafka на раздел ADG
    maxNumberOfMessagesPerPartition: ${ADG_MAX_MSG_PER_PARTITION:200}
# время простоя (в секундах) callback-функции
    callbackFunctionSecIdle: ${ADG_CB_FUNC_IDLE:100}
# настройки отката операции
  rollback:
# размер пакета операций при откате
    eraseOperationBatchSize: ${ADG_ROLLBACK_OPERATION_BATCH_SIZE:300}
# настройки отказоустойчивости ADG по паттерну circuitbreaker
  circuitbreaker:
# максимальное количество отказов ADG
    maxFailures: ${ADG_CIRCUIT_BREAKER_MAX_FAILURES:5}
# время ожидания (в миллисекундах) отклика ADG до фиксации отказа
    timeout: ${ADG_CIRCUIT_BREAKER_TIMEOUT:30000}
# использование паттерна fallback при отказе
    fallbackOnFailure: ${ADG_CIRCUIT_BREAKER_FALLBACK_ON_FAILURE:false}
# время ожидания (в миллисекундах) до сброса по паттерну timeout
    resetTimeout: ${ADG_CIRCUIT_BREAKER_RESET_TIMEOUT:10000}
# настройки для подключений веб-клиентов
  web-client:
# максимальный размер пула подключений веб-клиентов к ADG
    max-pool-size: ${ADG_WEB_CLIENT_MAX_POOL_SIZE:100}
```

### Настройки СУБД ADQM {#adqm_parameters}
```yaml
# настройки ADQM
adqm:
# настройки источника данных ADQM
  datasource:
# имя ADQM
    database: ${ADQM_DB_NAME:upload}
# имя пользователя/логин для авторизации в ADQM
    user: ${ADQM_USERNAME:}
# пароль для авторизации в ADQM
    password: ${ADQM_PASS:}
# сетевой адрес хоста с ADQM и номер порта на хосте
    hosts: ${ADQM_HOSTS:localhost:8123}
# время ожидания (в миллисекундах) отклика соединения с ADQM до тайм-аута
    socketTimeout: ${ADQM_SOCKET_TIMEOUT:30000}
# время ожидания (в миллисекундах) завершения обмена данными с ADQM до тайм-аута
    dataTransferTimeout: ${ADQM_DATA_TRANSFER_TIMEOUT:10000}
# настройки DDL-операторов
  ddl:
# имя кластера ADQM
    cluster: ${ADQM_CLUSTER:test_arenadata}
# алгоритм шардирования данных; 
# возможные значения: cityHash64 (значение по умолчанию в версии 5.3 и выше), intAdd (значение по умолчанию в версии 5.2 и ниже)
    shardingKeyExpr: ${ADQM_SHARDING_EXPR:cityHash64}
# настройки механизма выгрузки данных из ADQM
  mppr:
# сетевой адрес и путь для запросов на выгрузку данных
    loadingUrl: ${ADQM_MPPR_CONNECTOR_URL:http://localhost:8086/query}
# сетевой адрес и путь для получения информации о версии коннектора
    versionUrl: ${ADQM_MPPR_CONNECTOR_VERSION_URL:http://localhost:8086/versions}
# настройки механизма загрузки данных в ADQM
  mppw:
# имя консьюмер-группы для загрузки данных в ADQM
# не используется
    consumerGroup: ${ADQM_CONSUMER_GROUP:adqm}
# сетевой адрес брокера сообщений Kafka
    kafkaBrokers: ${ADQM_BROKERS:localhost:9092}
# тип интерфейса для загрузки данных в ADQM
    loadType: ${ADQM_MPPW_LOAD_TYPE:REST}
# сетевой адрес и путь к REST-интерфейсу для загрузки новых данных в ADQM
    restStartLoadUrl: ${ADQM_REST_START_LOAD_URL:http://localhost:8090/newdata/start}
# сетевой адрес и путь к REST-интерфейсу для остановки загрузки данных в ADQM
    restStopLoadUrl: ${ADQM_REST_STOP_LOAD_URL:http://localhost:8090/newdata/stop}
# сетевой адрес и путь для получения информации о версии коннектора
    versionUrl: ${ADQM_MPPW_CONNECTOR_VERSION_URL:http://localhost:8090/versions}
# имя коньсюмер-группы для загрузки данных в ADQM через REST API
    restLoadConsumerGroup: ${ADQM_REST_LOAD_GROUP:adb-emulator-load-adqm}
# настройки подключений веб-клиентов
  web-client:
# максимальный размер пула подключений веб-клиентов к ADQM
    max-pool-size: ${ADQM_WEB_CLIENT_MAX_POOL_SIZE:100}
```

### Настройки СУБД ADP {#adp_parameters}
```yaml
# настройки ADP
adp:
# настройки источника данных ADP
  datasource:
# имя пользователя/логин для авторизации в ADP
    user: ${ADP_USERNAME:dtm}
# пароль для авторизации в ADP
    password: ${ADP_PASS:dtm}
# сетевой адрес хоста с ADP
    host: ${ADP_HOST:localhost}
# сетевой адрес порта на хосте с ADP
    port: ${ADP_PORT:5432}
# лимит подключений к ADP в одном потоке; лимит по всем потокам равен произведению poolSize и executorsCount
    poolSize: ${ADP_MAX_POOL_SIZE:3}
# количество одновременных потоков, исполняющих запросы к ADP
    executorsCount: ${ADP_EXECUTORS_COUNT:3}
# максимальный размер результата, возвращаемого по FETCH-запросу к ADP
    fetchSize: ${ADP_FETCH_SIZE:1000}
# максимальный размер кэша запроса prepared statement
    preparedStatementsCacheMaxSize: ${ADP_PREPARED_CACHE_MAX_SIZE:256}
# максимальный размер запроса prepared statement, который может быть закэширован
    preparedStatementsCacheSqlLimit: ${ADP_PREPARED_CACHE_SQL_LIMIT:2048}
# признак кэширования запросов prepared statement
    preparedStatementsCache: ${ADP_PREPARED_CACHE:true}
# настройки механизма загрузки данных в ADP
  mppw:
# сетевой адрес и путь к REST-интерфейсу для загрузки данных в ADP
    restStartLoadUrl: ${ADP_REST_START_LOAD_URL:http://localhost:8096/newdata/start}
# сетевой адрес и путь к REST-интерфейсу для остановки загрузки данных в ADP
    restStopLoadUrl: ${ADP_REST_STOP_LOAD_URL:http://localhost:8096/newdata/stop}
# сетевой адрес и путь для получения информации о версии коннектора
    restVersionUrl: ${ADP_MPPW_CONNECTOR_VERSION_URL:http://localhost:8096/versions}
# имя коньсюмер-группы для загрузки данных в ADP через REST API
    kafkaConsumerGroup: ${ADP_KAFKA_CONSUMER_GROUP:adp-load}
# настройки механизма выгрузки данных из ADP
  mppr:
# сетевой адрес и путь для запросов на выгрузку данных из ADP
    restLoadUrl: ${ADP_MPPR_QUERY_URL:http://localhost:8094/query}
# сетевой адрес и путь для получения информации о версии коннектора
    restVersionUrl: ${ADP_MPPR_CONNECTOR_VERSION_URL:http://localhost:8094/versions}
```

## Конфигурация сервиса мониторинга статусов Kafka {#status_monitor_configuration}

```yaml
# настройки cервиса мониторинга статусов Kafka
monitor:
# список адресов брокеров сообщений Kafka
brokersList: ${STATUS_MONITOR_BROKERS:localhost:9092}
# количество потребителей (консьюмеров) cервиса мониторинга Kafka
consumersCount: ${STATUS_MONITOR_CONSUMERS:8}
```