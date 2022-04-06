---
layout: default
title: Формат сообщений в системном топике Kafka
nav_order: 7
parent: Справочная информация
has_children: false
has_toc: false
---

# Формат сообщений в системном топике Kafka {#system_topic_format}
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Система записывает в топик Kafka сообщения по следующим событиям:
* [изменение логической схемы данных](../../working_with_system/logical_schema_update/logical_schema_update.md),
* [открытие](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md) [дельты](../../overview/main_concepts/delta/delta.md),
* [закрытие дельты](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md),
* [откат дельты](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).

Сообщения записываются в JSON-формате в топик, имя которого задано с помощью параметра 
[конфигурации](../../maintenance/configuration/system/system.md) `KAFKA_STATUS_EVENT_TOPIC`. 
Каждое сообщение состоит из ключа и тела, их форматы описаны ниже.

## Формат ключа сообщения {#key_format}

Ключ сообщения в системном топике Kafka имеет следующий формат:

```json
{
  "datamart": "<logical_db>",
  "datetime": "<event_datetime>",
  "event": "<event_code>",
  "eventLogId": "<event_UUID>"
}
```

Где:
* `logical_db` — имя логической базы данных, в которой произошло событие;
* `event_datetime` — дата и время события;
* `event_code` — код события. Возможные значения: 
  * `DATAMART_SCHEMA_CHANGED` — изменение логической схемы,
  * `DELTA_OPEN` — открытие дельты, 
  * `DELTA_CLOSE` — закрытие дельты,
  * `DELTA_CANCEL` — откат дельты;
* `event_UUID` — уникальный идентификатор события.

Пример ключа в сообщении о закрытии дельты:

```json
{
  "datamart": "marketing", 
  "datetime": "2022-02-28 08:02:40",
  "event": "DELTA_CLOSE",
  "eventLogId": "6b80c392-3198-492f-a631-6a684e521b79"
}
```

## Формат тела сообщения {#body_format}

Тело сообщения в системном топике Kafka содержит параметры события и имеет формат, описанный в таблице ниже.

| Событие | Формат тела сообщения
|:-|:-
| Изменение логической схемы | `{"changeDateTime": "<change_datetime>", "datamart": "<logical_db>"}`
| Открытие дельты | `{"deltaNum": <delta_number>}`
| Закрытие дельты | `{"deltaNum": <delta_number>, "deltaDate": "<delta_commit_datetime>"}`
| Откат дельты | `{"deltaNum": <delta_number>}`

Где:
* `change_datetime` — дата и время изменения в формате `YYYY-MM-DD hh:mm:ss`;
* `logical_db` — имя логической базы данных, к которой относится изменение;
* `delta_number` — номер дельты;
* `delta_commit_datetime` — дата и время закрытия дельты в формате `YYYY-MM-DD hh:mm:ss`.

Пример тела в сообщении об изменении логической схемы:

```json
{
  "changeDateTime": "2022-02-24 15:25:47",
  "datamart": "marketing"
}
```

Пример тела в сообщении о закрытии дельты:

```json
{
  "deltaNum": 8,
  "deltaDate": "2022-02-28 08:02:40"
}
```