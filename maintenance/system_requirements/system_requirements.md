---
layout: default
title: Минимальные системные требования
nav_order: 1
parent: Эксплуатация
has_children: false
has_toc: false
---

# Минимальные системные требования {#system_requirements}

## Аппаратные требования системы {#hardware}

Система предъявляет следующие минимальные аппаратные требования:
* ядро системы: 4 CPU, 16 RAM, 20 HDD;
* сервис мониторинга статусов Kafka: 2 CPU, 8 RAM, 20 HDD.

## Программные требования системы {#software}
Система предъявляет следующие минимальные программные требования:
* компоненты Prostore:
  * сервис исполнения запросов (query-execution-core) версии 5.3.0,
  * сервис мониторинга статусов Kafka (status-monitor) версии 5.3.0,
  * JDBC-драйвер версии 5.3.0, 
* ADB версии 6.17.5,
  * коннектор Kafka-Greenplum reader (PXF) версии 1.0,
  * коннектор Kafka-Greenplum writer (FDW) версии 0.10.2,
* ADQM версии 20.4.4.18,
  * коннектор Kafka-Clickhouse reader версии 3.5.6,
  * коннектор Kafka-Clickhouse writer версии 3.5.6,
* ADG версии 2.7.3,
  * коннектор Kafka-Tarantool версии 0.8.5-1,
* PostgreSQL версии 13.5,
  * коннектор Kafka-Postgres reader версии 0.1.7,
  * коннектор Kafka-Postgres writer версии 0.1.7,
* ADS:
  * ADS версии 1.5,
  * Kafka версии 2.4,
  * Zookeeper версии 3.5.6.

Ссылки на репозитории с исходным кодом коннекторов доступны в разделе [Ресурсы](../../resources/resources.md).
{: .note-wrapper}

## Аппаратные и программные требования компонентов {#components}

Компоненты, с которыми работает система, предъявляют минимальные требования, перечисленные 
в таблице ниже.

| Компонент | Системные требования
|:-|:-
| ADB | <https://docs.arenadata.io/adb/requirements/online.html#id2>
| ADQM | <https://docs.arenadata.io/adqm/requirements/index.html#clickhouse>
| ADG (Tarantool) | <https://www.tarantool.io/en/sizing_calculator/>
| PostgreSQL | <https://postgrespro.ru/docs/postgresql/13/install-requirements>
| Kafka | <https://docs.arenadata.io/ads/Requirements/min.html>
| Zookeeper | <https://docs.arenadata.io/ads/Requirements/min.html>