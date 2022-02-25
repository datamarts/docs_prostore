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

Система записывает служебные сообщения по следующим событиям:
* [изменение логической схемы данных](../../working_with_system/logical_schema_update/logical_schema_update.md),
* [открытие](../../reference/sql_plus_requests/BEGIN_DELTA/BEGIN_DELTA.md) [дельты](../../overview/main_concepts/delta/delta.md),
* [закрытие дельты](../../reference/sql_plus_requests/COMMIT_DELTA/COMMIT_DELTA.md),
* [откат дельты](../../reference/sql_plus_requests/ROLLBACK_DELTA/ROLLBACK_DELTA.md).

Сообщения записываются в топик Kafka, имя которого 
задано в [конфигурации системы](../../maintenance/configuration/system/system.md) с помощью параметра 
`KAFKA_STATUS_EVENT_TOPIC`. Каждое служебное сообщение состоит из ключа и тела. Форматы ключа и тела 
описаны ниже.

## Формат ключа сообщения {#key_format}

Ключ сообщения в системном топике Kafka имеет следующий формат:

```json
{
  datamart: '<logical_db_name>',
  datetime: '<event_datetime>',
  event: '<event_code>',
  eventLogId: <event_UUID>
}
```

Где:
* `logical_db_name` — имя логической базы данных, в которой произошло событие;
* `event_datetime` — дата и время события;
* `event_code` — код события. Возможные значения: 
  * `DATAMART_SCHEMA_CHANGED` — изменение логической схемы,
  * `DELTA_OPEN` — открытие дельты, 
  * `DELTA_CLOSE` — закрытие дельты,
  * `DELTA_CANCEL` — откат дельты;
* `event_UUID` — уникальный идентификатор события

## Формат тела сообщения {#body_format}

Тело сообщения в системном топике Kafka содержит параметры события и имеет формат, описанный в таблице ниже.

| Событие | Формат тела сообщения
|:-|:-
| Изменение логической схемы | `{"changeDateTime":<change_datetime>,"datamart":"<logical_db_name>"}`
| Открытие дельты | `{"deltaNum":<delta_number>}`
| Закрытие дельты | `{"deltaNum":<delta_number>,"deltaDate":"<delta_commit_datetime>"}`
| Откат дельты | `{"deltaNum":<delta_number>}`

Где:
* `change_datetime` — дата и время изменения в формате `YYYY-MM-DD hh:mm:ss`;
* `logical_db_name` — имя логической базы данных, к которой относится изменение;
* `delta_number` — номер дельты;
* `delta_commit_datetime` — дата и время закрытия дельты в формате `YYYY-MM-DD hh:mm:ss`.

Пример тела сообщения об изменении логической схемы:

```json
{"changeDateTime":"2022-02-24 15:25:47","datamart":"sales"}
```

Пример тела сообщения о закрытии дельты:

```json
{"deltaNum":8,"deltaDate":"2021-10-22 17:54:00"}
```