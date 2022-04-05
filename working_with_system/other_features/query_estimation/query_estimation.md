---
layout: default
title: Получение информации о SELECT-запросе
nav_order: 4
parent: Другие действия
grand_parent: Работа с системой
has_children: false
has_toc: false
---

# Получение информации о SELECT-запросе {#query_estimation}

По [SELECT](../../../reference/sql_plus_requests/SELECT/SELECT.md)-запросу можно получить информацию 
о его выполнении без фактического выполнения в [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md) 
[хранилища](../../../overview/main_concepts/data_storage/data_storage.md). Доступна следующая информация:
* имя СУБД хранилища, в которой предполагается выполнение запроса;
* план выполнения запроса (только для ADB и ADP);
* обогащенный запрос, подготовленный системой на основе исходного запроса с учетом специфики СУБД хранилища.

Чтобы получить информацию о SELECT-запросе, выполните его с ключевым словом `ESTIMATE_ONLY` (см. пример [ниже](#examples)).

Подробнее о формате и содержимом ответа см. в секции 
[Ключевое слово ESTIMATE_ONLY](../../../reference/sql_plus_requests/SELECT/SELECT.md#estimate)
раздела [SELECT](../../../reference/sql_plus_requests/SELECT/SELECT.md), о возможных ключевых словах SELECT-запроса 
— в секции [Поддерживаемые ключевые слова](../../../reference/sql_plus_requests/SELECT/SELECT.md#key_words) 
того же раздела.

## Пример {#examples}

```sql
SELECT s.store_id, SUM(s.product_units) AS product_amount
FROM marketing.sales AS s
GROUP BY (s.store_id)
ORDER BY product_amount DESC
ESTIMATE_ONLY
```