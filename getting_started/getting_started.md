---
layout: default
title: Сборка и развёртывание
nav_order: 2.5
has_children: false
has_toc: false
---

# Сборка и развёртывание {#build_and_deploy}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

В этом разделе описаны шаги по развёртыванию среды в конфигурации, предполагающей единственное хранилище — СУБД PostgreSQL. 
Дополнительная информация приведена в разделе [Схемы развёртывания](../maintenance/deployment_diagrams/deployment_diagrams.md).

## Предустановленные программные средства {#preinstalled_software}
*   OC Centos 7;
*   yum-utils;
*   curl;
*   git;
*   wget;
*   OpenJDK 8;
*   Apache Maven 3.6.3;
*   СУБД PostgreSQL 13;
*   Apache Zookeeper;
*   Apache Kafka (например, в каталоге /opt/kafka);
*   SQL-клиент, например DBeaver;
*   docker;
*   Браузер топиков Kafka с возможностью загрузки бинарных данных, например kafkacat.

## Сборка Prostore {#prostore_build}

```shell
# клонирование репозитория Prostore
git clone https://github.com/datamarts/prostore ~/prostore
# запуск сборки Prostore средствами Apache Maven
cd ~/prostore
mvn clean install -DskipTests=true
# создание символической ссылки на файл конфигурации
sudo ln -s ~/prostore/dtm-query-execution-core/config/application.yml ~/prostore/dtm-query-execution-core/target/application.yml
# приведение конфигурационного файла к виду, показанному ниже
sudo nano ~/prostore/dtm-query-execution-core/config/application.yml
```
<details markdown="block">
  <summary>
    конфигурационный файл Prostore `application.yml`
  </summary>
  {: .text-delta }
```yml
#
# Copyright © 2021 ProStore
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#    http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.
#

logging:
  level:
    io.arenadata.dtm.query.execution: ${DTM_LOGGING_LEVEL:TRACE}

server:
  port: ${DTM_METRICS_PORT:8080}

management:
  endpoints:
    enabled-by-default: ${DTM_METRICS_ENABLED:true}
    web:
      exposure:
        include: ${DTM_METRICS_SCOPE:info, health, requests}

core:
  plugins:
    active: ${CORE_PLUGINS_ACTIVE:ADP}

  http:
    port: ${DTM_CORE_HTTP_PORT:9090}
    tcpNoDelay: ${DTM_CORE_HTTP_TCP_NO_DELAY:true}
    tcpFastOpen: ${DTM_CORE_HTTP_TCP_FAST_OPEN:true}
    tcpQuickAck: ${DTM_CORE_HTTP_TCP_QUICK_ACK:true}

  env:
    name: ${DTM_NAME:test}

  restoration:
    autoRestoreState: ${AUTO_RESTORE_STATE:true}

  matviewsync:
    periodMs: ${MATERIALIZED_VIEWS_SYNC_PERIOD_MS:5000}
    retryCount: ${MATERIALIZED_VIEWS_RETRY_COUNT:10}
    maxConcurrent: ${MATERIALIZED_VIEWS_CONCURRENT:2}

  metrics:
    enabled: ${DTM_CORE_METRICS_ENABLED:true}

  datasource:
    edml:
      defaultChunkSize: ${EDML_DEFAULT_CHUNK_SIZE:1000}
      pluginStatusCheckPeriodMs: ${EDML_STATUS_CHECK_PERIOD_MS:1000}
      firstOffsetTimeoutMs: ${EDML_FIRST_OFFSET_TIMEOUT_MS:15000}
      changeOffsetTimeoutMs: ${EDML_CHANGE_OFFSET_TIMEOUT_MS:10000}
    zookeeper:
      connection-string: ${ZOOKEEPER_DS_ADDRESS:localhost}
      connection-timeout-ms: ${ZOOKEEPER_DS_CONNECTION_TIMEOUT_MS:30000}
      session-timeout-ms: ${ZOOKEEPER_DS_SESSION_TIMEOUT_MS:86400000}
      chroot: ${ZOOKEEPER_DS_CHROOT:/adtm}

  kafka:
    producer:
      property:
        key.serializer: org.apache.kafka.common.serialization.StringSerializer
        value.serializer: org.apache.kafka.common.serialization.StringSerializer
    cluster:
      zookeeper:
        connection-string: ${ZOOKEEPER_KAFKA_ADDRESS:localhost}
        connection-timeout-ms: ${ZOOKEEPER_KAFKA_CONNECTION_TIMEOUT_MS:30000}
        session-timeout-ms: ${ZOOKEEPER_KAFKA_SESSION_TIMEOUT_MS:86400000}
        chroot: ${ZOOKEEPER_KAFKA_CHROOT:}
    admin:
      inputStreamTimeoutMs: ${KAFKA_INPUT_STREAM_TIMEOUT_MS:2000}
    status.event.publish:
      enabled: ${KAFKA_STATUS_EVENT_ENABLED:false}
    statusMonitor:
      statusUrl: ${STATUS_MONITOR_URL:http://localhost:9095/status}
      versionUrl: ${STATUS_MONITOR_VERSION_URL:http://localhost:9095/versions}

  vertx:
    blocking-stacktrace-time: ${DTM_VERTX_BLOCKING_STACKTRACE_TIME:1}
    pool:
      worker-pool: ${DTM_CORE_WORKER_POOL_SIZE:20}
      event-loop-pool: ${DTM_CORE_EVENT_LOOP_POOL_SIZE:20}
      task-pool: ${DTM_CORE_TASK_POOL_SIZE:20}
      task-timeout: ${DTM_CORE_TASK_TIMEOUT:86400000}

  cache:
    initialCapacity: ${CACHE_INITIAL_CAPACITY:100000}
    maximumSize: ${CACHE_MAXIMUM_SIZE:100000}
    expireAfterAccessMinutes: ${CACHE_EXPIRE_AFTER_ACCESS_MINUTES:99960}

  delta:
    rollback-status-calls-ms: ${DELTA_ROLLBACK_STATUS_CALLS_MS:2000}

adp:
  datasource:
    user: ${ADP_USERNAME:dtm}
    password: ${ADP_PASS:dtm}
    host: ${ADP_HOST:localhost}
    port: ${ADP_PORT:5432}
    poolSize: ${ADP_MAX_POOL_SIZE:3}
    executorsCount: ${ADP_EXECUTORS_COUNT:3}
    fetchSize: ${ADP_FETCH_SIZE:1000}
    preparedStatementsCacheMaxSize: ${ADP_PREPARED_CACHE_MAX_SIZE:256}
    preparedStatementsCacheSqlLimit: ${ADP_PREPARED_CACHE_SQL_LIMIT:2048}
    preparedStatementsCache: ${ADP_PREPARED_CACHE:true}

  mppw:
    restStartLoadUrl: ${ADP_REST_START_LOAD_URL:http://localhost:8096/newdata/start}
    restStopLoadUrl: ${ADP_REST_STOP_LOAD_URL:http://localhost:8096/newdata/stop}
    restVersionUrl: ${ADP_MPPW_CONNECTOR_VERSION_URL:http://localhost:8096/versions}
    kafkaConsumerGroup: ${ADP_KAFKA_CONSUMER_GROUP:adp-load}

  mppr:
    restLoadUrl: ${ADP_MPPR_QUERY_URL:http://localhost:8094/query}
    restVersionUrl: ${ADP_MPPR_CONNECTOR_VERSION_URL:http://localhost:8094/versions}
```
</details>

Далее конфигурационный файл `application.yml` обозначается термином "конфигурация Prostore".


## Настройка СУБД Postgres {#postgres_setup}

```shell
# создание в СУБД Postgres SUPERUSER-пользователя c именем и паролем,
# указанными в конфигурации Prostore
# (значения параметров (adp:datasource:user) и (adp:datasource:password) соответственно)
cd /
sudo -u postgres psql -c 'CREATE ROLE dtm WITH LOGIN SUPERUSER'
sudo -u postgres psql -c "ALTER ROLE dtm WITH PASSWORD 'dtm'"
# создание базы данных с именем test, указанным в конфигурации Prostore (env: name)
sudo -u postgres psql -c 'CREATE DATABASE test'
# перезапуск сервиса Postgresql
sudo systemctl reload postgresql-13
```

## Сборка и установка коннектора Kafka-Postgres {#kafka_postgres_connector_build_deploy}

```shell
# клонирование репозитория kafka-postgres-connector
git clone https://github.com/arenadata/kafka-postgres-connector ~/kafka-postgres-connector
# запуск сборки коннектора kafka-postgres средствами Apache Maven
cd ~/kafka-postgres-connector
mvn clean install -DskipTests=true
# приведение конфигурационных файлов kafka-postgres-writer и kafka-postgres-reader к виду,
# показанному ниже, чтобы значения параметров совпадали со значениями соответствующих параметров конфигурации Prostore
# datasource: postgres: database ~ env: name,
# datasource: postgres: user     ~ adp: datasource: user,
# datasource: postgres: password ~ adp: datasource: password,
# datasource: postgres: hosts    ~ adp: datasource: host, adp: datasource: port
sudo nano ~/kafka-postgres-connector/kafka-postgres-writer/src/main/resources/application.yml
sudo nano ~/kafka-postgres-connector/kafka-postgres-reader/src/main/resources/application.yml
# создание символических ссылок на файлы конфигурации
sudo ln -s ~/kafka-postgres-connector/kafka-postrges-writer/src/main/resources/application.yml ~/kafka-postgres-connector/kafka-postrges-writer/target/application.yml
sudo ln -s ~/kafka-postgres-connector/kafka-postrges-reader/src/main/resources/application.yml ~/kafka-postgres-connector/kafka-postrges-reader/target/application.yml
```
<details markdown="block">
  <summary>
    конфигурационный файл kafka-postgres-writer `application.yml`
  </summary>
  {: .text-delta }
```yml
logging:
  level:
    io.arenadata.kafka: ${LOG_LEVEL:DEBUG}
    org.apache.kafka: ${KAFKA_LOG_LEVEL:INFO}

http:
  port: ${SERVER_PORT:8096}

vertx:
  pools:
    eventLoopPoolSize: ${VERTX_EVENT_LOOP_SIZE:12}
    workersPoolSize: ${VERTX_WORKERS_POOL_SIZE:32}
  verticle:
    query:
      instances: ${QUERY_VERTICLE_INSTANCES:12}
    insert:
      poolSize: ${INSERT_WORKER_POOL_SIZE:32}
      insertPeriodMs: ${INSERT_PERIOD_MS:1000}
      batchSize: ${INSERT_BATCH_SIZE:500}
    consumer:
      poolSize: ${KAFKA_CONSUMER_WORKER_POOL_SIZE:32}
      maxFetchSize: ${KAFKA_CONSUMER_MAX_FETCH_SIZE:10000}
    commit:
      poolSize: ${KAFKA_COMMIT_WORKER_POOL_SIZE:1}
      commitPeriodMs: ${KAFKA_COMMIT_WORKER_COMMIT_PERIOD_MS:1000}

client:
  kafka:
    consumer:
      checkingTimeoutMs: ${KAFKA_CHECKING_TIMEOUT_MS:10000}
      responseTimeoutMs: ${KAFKA_RESPONSE_TIMEOUT_MS:10000}
      consumerSize: ${KAFKA_CONSUMER_SIZE:10}
      closeConsumersTimeout: ${KAFKA_CLOSE_CONSUMER_TIMEOUT:15000}
      property:
        bootstrap.servers: ${KAFKA_BOOTSTRAP_SERVERS:kafka.host:9092}
        group.id: ${KAFKA_CONSUMER_GROUP_ID:postgres-query-execution}
        auto.offset.reset: ${KAFKA_AUTO_OFFSET_RESET:earliest}
        enable.auto.commit: ${KAFKA_AUTO_COMMIT:false}
        auto.commit.interval.ms: ${KAFKA_AUTO_INTERVAL_MS:1000}

datasource:
  postgres:
    database: ${POSTGRES_DB_NAME:test}
    user: ${POSTGRES_USERNAME:dtm}
    password: ${POSTGRES_PASS:dtm}
    hosts: ${POSTGRES_HOSTS:localhost:5432}
    poolSize: ${POSTGRES_POOLSIZE:10}
    preparedStatementsCacheMaxSize: ${POSTGRES_CACHE_MAX_SIZE:256}
    preparedStatementsCacheSqlLimit: ${POSTGRES_CACHE_SQL_LIMIT:2048}
    preparedStatementsCache: ${POSTGRES_CACHE:true}
```
</details>

<details markdown="block">
  <summary>
    конфигурационный файл kafka-postgres-reader `application.yml`
  </summary>
  {: .text-delta }
```yml
logging:
  level:
    io.arenadata.kafka: ${LOG_LEVEL:DEBUG}
    org.apache.kafka: ${KAFKA_LOG_LEVEL:INFO}

http:
  port: ${SERVER_PORT:8094}

vertx:
  pools:
    eventLoopPoolSize: ${VERTX_EVENT_LOOP_SIZE:12}
    workersPoolSize: ${VERTX_WORKERS_POOL_SIZE:32}
  verticle:
    query:
      instances: ${QUERY_VERTICLE_INSTANCES:12}

datasource:
  postgres:
    database: ${POSTGRES_DB_NAME:test}
    user: ${POSTGRES_USERNAME:dtm}
    password: ${POSTGRES_PASS:dtm}
    hosts: ${POSTGRES_HOSTS:localhost:5432}
    poolSize: ${POSTGRES_POOLSIZE:10}
    preparedStatementsCacheMaxSize: ${POSTGRES_CACHE_MAX_SIZE:256}
    preparedStatementsCacheSqlLimit: ${POSTGRES_CACHE_SQL_LIMIT:2048}
    preparedStatementsCache: ${POSTGRES_CACHE:true}
    fetchSize: ${POSTGRES_FETCH_SIZE:1000}

kafka:
  client:
    property:
      key.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
      value.serializer: org.apache.kafka.common.serialization.ByteArraySerializer
```
</details>
## Запуск сервисов Apache Zookeeper и Apache Kafka {#zookeeper_kafka_execution}

```shell
# запуск одного экземпляра сервера ZooKeeper, если он еще не запущен
sudo systemctl start zookeeper
# запуск сервера Kafka и проверка его состояния
sudo systemctl start kafka
sudo systemctl status kafka
```

## Запуск коннектора Kafka-Postgres {#kafka_postgres_execution}

```shell
# запуск kafka-postgres-writer в отдельном окне терминала 
cd ~/kafka-postgres-connector/kafka-postgres-writer/target
java -jar kafka-postgres-writer-<version>.jar
# запуск kafka-postgres-reader в отдельном окне терминала
cd ~/kafka-postgres-connector/kafka-postgres-reader/target
java -jar kafka-postgres-reader-<version>.jar
```

## Запуск службы dtm-status-monitor {#dtm_status_monitor_execution}

```shell
# создание символической ссылки на файл конфигурации dtm-status-monitor
sudo ln -s ~/prostore/dtm-status-monitor/src/main/resources/application.yml ~/prostore/dtm-status-monitor/target/application.yml
# запуск dtm-status-monitor в отдельном окне терминала с указанием порта, заданного в конфигурации Prostore (core:kafka:statusMonitor)
cd ~/prostore/dtm-status-monitor/target
java -Dserver.port=9095 -jar dtm-status-monitor-<version>.jar
```
***Примечание:*** Запуск службы dtm-status-monitor без указания порта `-Dserver.port`
 приведёт к конкуренции с сервисом Prostore за использование последним порта по умолчанию `8080`.

## Запуск Prostore {#prostore_execution}

Запуск со значением по умолчанию (8080) для порта (server:port) в конфигурации Prostore:

```shell
# запуск файла dtm-query-execution-core-<version>.jar (например, dtm-query-execution-core-5.1.0.jar)
cd ~/prostore/dtm-query-execution-core/target
java -jar dtm-query-execution-core-<version>.jar
```

Запуск с иным заданным значением осуществляется путём изменения параметра (server:port) в конфигурации Prostore или задании переменной окружения <DTM_METRICS_PORT>.

## Подключение к Prostore с помощью SQL-клиента {#sql_client_connection}

Порядок подключения описан в разделе [Подключение с помощью SQL-клиента](../working_with_system/connection/connection_via_sql_client/connection_via_sql_client.md).

## Демонстрационный сценарий {#demo_scenario}

### Создание необходимых логических сущностей {#logical_entities_creation}

```sql
-- создание логической базы данных
CREATE DATABASE sales;
-- выбор логической БД по умолчанию
USE sales;
-- создание логической таблицы в БД sales
CREATE TABLE sales (
  id INT NOT NULL,
  transaction_date TIMESTAMP NOT NULL,
  product_code VARCHAR(256) NOT NULL,
  product_units INT NOT NULL,
  store_id INT NOT NULL,
  description VARCHAR(256),
  PRIMARY KEY (id)
)
DISTRIBUTED BY (id);
-- создание внешней таблицы загрузки
CREATE UPLOAD EXTERNAL TABLE sales_ext_upload (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://localhost:2181/salesTopic'
FORMAT 'AVRO'
MESSAGE_LIMIT 1000;
-- создание логического представления stores_by_sold_products
CREATE VIEW stores_by_sold_products AS
  SELECT store_id, SUM(product_units) AS product_amount
  FROM sales.sales
  GROUP BY store_id
  ORDER BY product_amount DESC
  LIMIT 30;
-- создание внешней таблицы выгрузки в топик Kafka "salesTopicOut"
CREATE DOWNLOAD EXTERNAL TABLE sales.sales_ext_download (
  id INT,
  transaction_date TIMESTAMP,
  product_code VARCHAR(256),
  product_units INT,
  store_id INT,
  description VARCHAR(256)
)
LOCATION  'kafka://localhost:2181/salesTopicOut'
FORMAT 'AVRO'
CHUNK_SIZE 1000;
```

### Создание топика Kafka для последующей загрузки данных {#upload_kafka_topic_creation}

Создание топика Kafka "salesTopic" в терминале:

```shell
cd /opt/kafka/bin
bash kafka-topics.sh --create --replication-factor 1 --partitions 1 --topic salesTopic --zookeeper localhost:2181
```

### Создание бинарного avro-файла kafka_upload_sales.avro из avro-схемы и данных {#avro_file_creation}

<details markdown="block">
  <summary>
    JSON-файл avro-схемы `kafka_upload_sales.avsc`
  </summary>
  {: .text-delta }
```json
{
  "name": "sales",
  "namespace": "sales",
  "type": "record",
  "fields": [
    {
	  "name": "id",
	  "type": "long"
    },
    {
      "name": "transaction_date",
	  "type": {
	    "type": "long",
	    "logicalType": "timestamp-micros"
	  }	
    },
    {
	  "name": "product_code",
	  "type": "string"
    },
    {
	  "name": "product_units",
	  "type": "long"
    },
    {
	  "name": "store_id",
	  "type": "long"
    },
    {
	  "name": "description",
	  "type": "string"
    },
    {
	  "name": "sys_op",
	  "type": "int"
    }
  ]
}
```
</details>

<details markdown="block">
  <summary>
    JSON-файл данных `kafka_upload_sales.json`
  </summary>
  {: .text-delta }
```json
{
  "id": 1000111,
  "transaction_date": 1614269474000000,
  "product_code": "ABC102101",
  "product_units": 2,
  "store_id": 1000012345,
  "description": "Покупка по акции 1+1",
  "sys_op": 0
}
{
  "id": 1000112,
  "transaction_date": 1614334214000000,
  "product_code": "ABC102001",
  "product_units": 1,
  "store_id": 1000000123,
  "description": "Покупка без акций",
  "sys_op": 0
}
{
  "id": 1000020,
  "transaction_date": 1614636614000000,
  "product_code": "ABC102010",
  "product_units": 4,
  "store_id": 1000000123,
  "description": "Покупка по акции 1+1",
  "sys_op": 0
}
```
</details>

<details markdown="block">
  <summary>
    бинарный AVRO-файл `kafka_upload_sales.avro`
  </summary>
  {: .text-delta } 
[Сохранить](./kafka_upload_sales.avro) бинарный файл
</details>

### Загрузка avro-файла kafka_upload_sales.avro {#avro_file_upload}

Загрузка avro-файла kafka_upload_sales.avro в топик Kafka "salesTopic" через терминал с помощью kafkacat:
```shell
#получение docker-образа kafkacat
sudo docker pull edenhill/kcat:1.7.0
#запуск docker-образа kafkacat для загрузки в топик salesTopic
#avro-файла /opt/kafka/sales/kafka_upload_sales.avro
sudo docker run -it --network host \
--volume /opt/kafka/sales/kafka_upload_sales.avro:/data/kafka_upload_sales.avro \
edenhill/kcat:1.7.0 -b localhost:9092 -t salesTopic -P /data/kafka_upload_sales.avro
 ```
 
### Загрузка данных {#data_upload}

```sql
-- открытие новой (горячей) дельты
BEGIN DELTA;
-- запуск загрузки данных в логическую таблицу sales
INSERT INTO sales SELECT * FROM sales.sales_ext_upload;
-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Вставка данных {#data_upsert}

```sql
-- открытие новой (горячей) дельты
BEGIN DELTA;
-- запуск вставки данных в логическую таблицу sales
INSERT INTO sales.sales
(id, transaction_date, product_code, product_units, store_id, description)
VALUES
(2000111, '2020-05-01 13:14:16', 'ABC202010', 7, 1000000123, 'Покупка без акций'),
(2000112, '2020-05-02 16:13:17', 'ABC202011', 11, 1000000456, 'Покупка без акций'),
(2000113, '2020-05-03 21:15:17', 'ABC202012', 5, 1000000789, 'Покупка без акций'),
(2000114, '2020-05-04 23:03:13', 'ABC202013', 7, 1000000123, 'Покупка без акций'),
(2000115, '2020-05-05 14:10:21', 'ABC202014', 21, 1000000623, 'Покупка без акций'),
(2000116, '2020-06-12 08:43:56', 'ABC202015', 32, 1000000987, 'Покупка без акций');
-- закрытие дельты (фиксация изменений)
COMMIT DELTA;
```

### Выборка данных {#data_selection}

```sql
-- запрос с неявным указанием столбцов и ключевым словом WHERE
SELECT * FROM sales.sales
WHERE store_id = 1000000123;
-- запрос с агрегацией, группировкой и сортировкой данных, а также выбором первых 5 строк
SELECT s.store_id, SUM(s.product_units) AS product_amount
FROM sales.sales AS s
GROUP BY (s.store_id)
ORDER BY product_amount DESC
LIMIT 5;
-- запрос к логическому представлению stores_by_sold_products
SELECT * from stores_by_sold_products;
```

### Выгрузка в топик Kafka {#kafka_topic_download}

```sql
-- запуск выгрузки данных из логической таблицы sales
INSERT INTO sales_ext_download 
SELECT * FROM sales WHERE product_units > 2;
```

### Удаление логических сущностей {#logical_entities_drop}

```sql
-- удаление внешней таблицы загрузки
DROP UPLOAD EXTERNAL TABLE sales_ext_upload;
-- удаление внешней таблицы выгрузки
DROP DOWNLOAD EXTERNAL TABLE sales_ext_download;
```
