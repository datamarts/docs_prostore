---
layout: default
title: Конфигурация коннекторов
nav_order: 2
parent: Конфигурация
grand_parent: Эксплуатация
has_children: false
---

# Конфигурация коннекторов {#connectors}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Следующие коннекторы требуют настройки конфигурации:
* Kafka-Clickhouse reader connector,
* Kafka-Clickhouse writer connector,
* Kafka-Postgres reader connector,
* Kafka-Postgres writer connector.

Ссылки на репозитории с исходным кодом коннекторов доступны в разделе [Ресурсы](../../../resources/resources.md).
{: .note-wrapper}

Конфигурация коннекторов задается в текстовых YAML-файлах; параметры конфигурации организованы в иерархическую 
древовидную структуру. В разделе приведены примеры файлов конфигурации коннекторов, где перед каждым параметром описано 
его назначение.

## Конфигурация коннектора Kafka-Clickhouse reader {#adqm_reader}

```yaml
# настройки Vertx
vertx:
  # признак кластеризованного режима Vertx
  clustered:true

# настройки журналирования
logging:
  level:
    # уровень важности сообщений, записываемых в лог-файл
    ru.datamart.kafka.clickhouse.reader: ${LOG_LEVEL:DEBUG}

# настройки вертиклов Vertx
verticle:
  worker:
    task-worker:
      # максимальный размер пула подключений веб-клиентов к ADQM
      poolSize: ${TASK_WORKER_POOL_SIZE:12}
      # имя пула подключений веб-клиентов к ADQM
      poolName: ${TASK_WORKER_POOL_NAME:task-worker}
      # внутренний таймаут обработки запросов (в миллисекундах)
      responseTimeoutMs: ${TASK_WORKER_RESPONSE_TIMEOUT_MS:86400000}

# настройки HTTP-подключений
http:
  # порт, на котором работает коннектор
  port: ${SERVER_PORT:8086}

# временная зона коннектора
timezone: ${MPPR_TIME_ZONE:UTC}

# настройки для работы с ADQM
datasource:
  clickhouse:
    # имя базы данных в ADQM
    database: ${CLICKHOUSE_DB_NAME:test1}
    # имя пользователя/логин для авторизации в ADQM
    user: ${CLICKHOUSE_USERNAME:default}
    # пароль для авторизации в ADQM
    password: ${CLICKHOUSE_PASS:}
    # сетевой адрес хоста с ADQM и номер порта на хосте
    hosts: ${CLICKHOUSE_HOSTS:clickhouse.host:8123}
    # максимальный размер результата, возвращаемого по FETCH-запросу к ADQM
    fetchSize: ${CLICKHOUSE_FETCH_SIZE:1000}

# настройки для работы с брокером сообщений Kafka
kafka:
  clickhouse:
    # настройки производителя данных
    producer:
      property:
        # сериализатор строковых ключей   
        key.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
        # сериализатор строковых значений
        value.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
    # настройки кластера
    cluster:
      # сетевой адрес хоста Zookeeper для брокера сообщений Kafka    
      zookeeperHosts: ${ZOOKEEPER_HOSTS:zk-1.dtm.local}
      # корневой путь к хосту Zookeeper для брокера сообщений Kafka   
      rootPath: ${KAFKA_CLUSTER_ROOTPATH:arenadata/cluster/21}
```

## Конфигурация коннектора Kafka-Clickhouse writer {#adqm_writer}

```yaml
# настройки HTTP-подключений
http:
  # порт, на котором работает коннектор
  port: ${SERVER_PORT:8090}

# настройки для работы с брокером сообщений Kafka
client:
  kafka:
    # настройки консьюмера (потребителя) Kafka
    consumer:
      # периодичность проверки (в миллисекундах) статуса брокера сообщений Kafka 
      checkingTimeoutMs: ${KAFKA_CHECKING_TIMEOUT_MS:10000}
      # время ожидания (в миллисекундах) ответа брокера сообщений Kafka до тайм-аута
      responseTimeoutMs: ${KAFKA_RESPONSE_TIMEOUT_MS:10000}
      # количество консьюмеров Kafka
      consumerSize: ${KAFKA_CONSUMER_SIZE:10}
      # время ожидания (в миллисекундах) до закрытия соединения с брокером сообщений Kafka   
      closeConsumersTimeout: ${KAFKA_CLOSE_CONSUMER_TIMEOUT:15000}
      # свойства консьюмера в соответствии с конфигурацией консьюмеров Kafka (https://kafka.apache.org/documentation/#consumerconfigs)       
      property:
        # список адресов и портов Bootstrap-серверов брокера сообщений Kafka       
        bootstrap.servers: ${KAFKA_BOOTSTRAP_SERVERS:kafka.host:9092}
        # имя консьюмер-группы для загрузки данных в ADQM
        group.id: ${KAFKA_CONSUMER_GROUP_ID:clickhouse-query-execution}
        # режим обнуления смещения в топиках Kafka
        auto.offset.reset: ${KAFKA_AUTO_OFFSET_RESET:earliest}
        # признак автоматической записи смещения в топиках Kafka   
        enable.auto.commit: ${KAFKA_AUTO_COMMIT:false}
        # периодичность (в миллисекундах) автоматической записи смещения в топиках Kafka       
        auto.commit.interval.ms: ${KAFKA_AUTO_INTERVAL_MS:1000}

# настройки окружения
env:
  # имя окружения
  name: ${ENV:test}

# настройки, связанные с системными полями
datamart:
  # системное поле, хранящее номер операции записи
  hot-delta-field-name: sys_from

# настройки для работы с ADQM
datasource:
  clickhouse:
    # имя базы данных в ADQM
    database: ${CLICKHOUSE_DB_NAME:test1}
    # имя пользователя/логин для авторизации в ADQM    
    user: ${CLICKHOUSE_USERNAME:default}
    # пароль для авторизации в ADQM
    password: ${CLICKHOUSE_PASS:}
    # список хостов ADQM (адрес:порт через запятую)   
    hosts: ${CLICKHOUSE_HOSTS:clockhouse.host:8123}

# настройки VertX
verticle:
  worker:
    # настройки компонента, обрабатывающего запросы
    new-data-worker:
      # максимальный размер пула потоков
      poolSize: ${DATA_WORKER_POOL_SIZE:20}
      # имя пула потоков
      poolName: ${DATA_WORKER_POOL_NAME:new-data-worker}
    # настройки компонента, обрабатывающего полученные данные
    task-worker:
      # максимальный размер пула потоков
      poolSize: ${TASK_WORKER_POOL_SIZE:12}
      # имя пула потоков
      poolName: ${TASK_WORKER_POOL_NAME:task-worker}
      # внутренний таймаут обработки полученных данных (в миллисекундах)
      responseTimeoutMs: ${TASK_WORKER_RESPONSE_TIMEOUT_MS:86400000}
    # настройки компонента, выполняющего вставку данных
    insert-worker:
      # максимальный размер пула потоков
      poolSize: ${INSERT_WORKER_POOL_SIZE:32}
      # имя пула потоков
      poolName: ${INSERT_WORKER_POOL_NAME:insert-worker}
      # внутренний таймаут обработки вставки данных (в миллисекундах)
      responseTimeoutMs: ${INSERT_WORKER_RESPONSE_TIMEOUT_MS:86400000}
      # периодичность вставки данных (в миллисекундах)
      insertPeriodMs: ${INSERT_PERIOD_MS:1000}
      # размер пакета операций при вставке данных 
      batchSize: ${INSERT_BATCH_SIZE:500}
    # настройки компонента, считывающего данные их топиков Kafka  
    kafka-consumer-worker:
      # максимальный размер пула потоков
      poolSize: ${KAFKA_CONSUMER_WORKER_POOL_SIZE:32}
      # имя пула потоков
      poolName: ${KAFKA_CONSUMER_WORKER_POOL_NAME:insert-worker}
      # внутренний таймаут обработки консьюмера данных (в миллисекундах)
      responseTimeoutMs: ${KAFKA_CONSUMER_WORKER_RESPONSE_TIMEOUT_MS:86400000}
      # максимальный размер результата, возвращаемого по FETCH-запросу к ADQM
      maxFetchSize: ${KAFKA_CONSUMER_MAX_FETCH_SIZE:10000}
    # настройки компонента, записывающего смещение в топиках Kafka   
    kafka-commit-worker:
      # максимальный размер пула потоков
      poolSize: ${KAFKA_COMMIT_WORKER_POOL_SIZE:1}
      # имя пула потоков
      poolName: ${KAFKA_COMMIT_WORKER_POOL_NAME:commit-worker}
      # периодичность записи смещения в топиках Kafka (в миллисекундах)
      commitPeriodMs: ${KAFKA_COMMIT_WORKER_COMMIT_PERIOD_MS:1000}

# настройки журналирования
logging:
  level:
    # уровень важности сообщений, записываемых в лог-файл   
    ru.datamart.prostore: DEBUG
    # уровень важности сообщений, записываемых в лог-файл по событиям Kafka     
    org.apache.kafka: INFO
```

## Конфигурация коннектора Kafka-Postgres reader {#adp_reader}

```yaml
# настройки журналирования
logging:
  level:
    # уровень важности сообщений, записываемых в лог-файл
    ru.datamart.kafka: ${LOG_LEVEL:DEBUG}
    # уровень важности сообщений, записываемых в лог-файл по событиям Kafka 
    org.apache.kafka: ${KAFKA_LOG_LEVEL:INFO}

# настройки HTTP-подключений
http:
  # порт, на котором работает коннектор
  port: ${SERVER_PORT:8094}

# настройки дла работы с Vertx
vertx:
  pools:
    # максимальный размер пула потоков, обрабатывающих события Vertx
    eventLoopPoolSize: ${VERTX_EVENT_LOOP_SIZE:12}
    # максимальный размер пула потоков, выполняющих долгие операции
    workersPoolSize: ${VERTX_WORKERS_POOL_SIZE:32}
  verticle:
    query:
      # количество экземпляров, принимающих запросы
      instances: ${QUERY_VERTICLE_INSTANCES:12}

# настройки для работы с ADP
datasource:
  postgres:
    # имя базы данных ADP
    database: ${POSTGRES_DB_NAME:db}
    # имя пользователя/логин для авторизации в ADP
    user: ${POSTGRES_USERNAME:user}
    # пароль для авторизации в ADP
    password: ${POSTGRES_PASS:password}
    # сетевой адрес хоста с ADP и номер порта на хосте
    hosts: ${POSTGRES_HOSTS:postgres.host:5432}
    # максимальный размер пула потоков
    poolSize: ${POSTGRES_POOLSIZE:10}
    # максимальный размер кэша запроса prepared statement 
    preparedStatementsCacheMaxSize: ${POSTGRES_CACHE_MAX_SIZE:256}
    # максимальный размер запроса prepared statement, который может быть закэширован  
    preparedStatementsCacheSqlLimit: ${POSTGRES_CACHE_SQL_LIMIT:2048}
    # признак кэширования запросов prepared statement
    preparedStatementsCache: ${POSTGRES_CACHE:true}
    # максимальный размер результата, возвращаемого по FETCH-запросу к ADP  
    fetchSize: ${POSTGRES_FETCH_SIZE:1000}

# настройки для работы с брокером сообщений Kafka
kafka:
  client:
    property:
      # сериализатор строковых ключей
      key.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
      # сериализатор строковых значений
      value.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
```

## Конфигурация коннектора Kafka-Postgres writer {#adp_writer}

```yaml
# настройки журналирования
logging:
  level:
    # уровень важности сообщений, записываемых в лог-файл
    ru.datamart.kafka: ${LOG_LEVEL:DEBUG}
    # уровень важности сообщений, записываемых в лог-файл по событиям Kafka   
    org.apache.kafka: ${KAFKA_LOG_LEVEL:INFO}

# настройки HTTP-подключений
http:
  # порт, на котором работает коннектор
  port: ${SERVER_PORT:8096}

# настройки для работы с Vertx
vertx:
  pools:
    # максимальный размер пула потоков, обрабатывающих события Vertx 
    eventLoopPoolSize: ${VERTX_EVENT_LOOP_SIZE:12}
    # максимальный размер пула потоков, выполняющих долгие операции 
    workersPoolSize: ${VERTX_WORKERS_POOL_SIZE:32}
  verticle:
    query:
      # количество экземпляров вертиклов, обрабатывающих запросы
      instances: ${QUERY_VERTICLE_INSTANCES:12}
    insert:
      # максимальный размер пула потоков, вставляющих данные 
      poolSize: ${INSERT_WORKER_POOL_SIZE:32}
      # периодичность вставки новых данных (в миллисекундах)
      insertPeriodMs: ${INSERT_PERIOD_MS:1000}
      # размер пакета операций при вставке данных
      batchSize: ${INSERT_BATCH_SIZE:500}
    consumer:
      # максимальный размер пула потоков, считывающих данные        
      poolSize: ${KAFKA_CONSUMER_WORKER_POOL_SIZE:32}
      # максимальный размер результата, возвращаемого по FETCH-запросу к ADP
      maxFetchSize: ${KAFKA_CONSUMER_MAX_FETCH_SIZE:10000}
    commit:
      # размер пула потоков, записывающих смещение в топиках Kafka
      poolSize: ${KAFKA_COMMIT_WORKER_POOL_SIZE:1}
      # периодичность записи смещения в топиках Kafka (в миллисекундах)
      commitPeriodMs: ${KAFKA_COMMIT_WORKER_COMMIT_PERIOD_MS:1000}

# настройки для работы с брокером сообщений Kafka
client:
  kafka:
    # настройки консьюмера (потребителя) Kafka
    consumer:
      # периодичность проверки (в миллисекундах) статуса брокера сообщений Kafka  
      checkingTimeoutMs: ${KAFKA_CHECKING_TIMEOUT_MS:10000}
      # время ожидания (в миллисекундах) ответа от брокера сообщений Kafka до тайм-аута    
      responseTimeoutMs: ${KAFKA_RESPONSE_TIMEOUT_MS:10000}
      # количество консьюмеров Kafka
      consumerSize: ${KAFKA_CONSUMER_SIZE:10}
      # время ожидания (в миллисекундах) до закрытия соединения с брокером сообщений Kafka    
      closeConsumersTimeout: ${KAFKA_CLOSE_CONSUMER_TIMEOUT:15000}
      # свойства консьюмера в соответствии с конфигурацией консьюмеров Kafka (https://kafka.apache.org/documentation/#consumerconfigs)  
      property:
        # список адресов и портов Bootstrap-серверов брокера сообщений Kafka
        bootstrap.servers: ${KAFKA_BOOTSTRAP_SERVERS:kafka.host:9092}
        # имя консьюмер-группы для загрузки данных в ADP
        group.id: ${KAFKA_CONSUMER_GROUP_ID:postgres-query-execution}
        # режим обнуления смещения в топиках Kafka
        auto.offset.reset: ${KAFKA_AUTO_OFFSET_RESET:earliest}
        # признак автоматической записи смещения в топиках Kafka
        enable.auto.commit: ${KAFKA_AUTO_COMMIT:false}
        # периодичность (в миллисекундах) автоматической записи смещения в топиках Kafka
        auto.commit.interval.ms: ${KAFKA_AUTO_INTERVAL_MS:1000}

# настройки для работы с ADP
datasource:
  postgres:
    # имя базы данных ADP
    database: ${POSTGRES_DB_NAME:db}
    # имя пользователя/логин для авторизации в ADP
    user: ${POSTGRES_USERNAME:user}
    # пароль для авторизации в ADP
    password: ${POSTGRES_PASS:password}
    # сетевой адрес хоста с ADP и номер порта на хосте   
    hosts: ${POSTGRES_HOSTS:postgres.host:5432}
    # максимальный размер пула потоков
    poolSize: ${POSTGRES_POOLSIZE:10}
    # максимальный размер кэша запроса prepared statement   
    preparedStatementsCacheMaxSize: ${POSTGRES_CACHE_MAX_SIZE:256}
    # максимальный размер запроса prepared statement, который может быть закэширован     
    preparedStatementsCacheSqlLimit: ${POSTGRES_CACHE_SQL_LIMIT:2048}
    # признак кэширования запросов prepared statement
    preparedStatementsCache: ${POSTGRES_CACHE:true}
```
