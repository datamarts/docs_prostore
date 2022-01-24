---
layout: default
title: INSERT INTO download_external_table
nav_order: 34
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# INSERT INTO download_external_table
{: .no_toc }

<details markdown="block">
  <summary>
    Содержание раздела
  </summary>
  {: .text-delta }
1. TOC
{:toc}
</details>

Запрос позволяет выгрузить данные, выбранные [SELECT](../SELECT/SELECT.md)-подзапросом 
к [логической базе данных](../../../overview/main_concepts/logical_db/logical_db.md), 
во внешний приемник данных. Данные можно выгружать из [логических таблиц](../../../overview/main_concepts/logical_table/logical_table.md), 
[логических](../../../overview/main_concepts/logical_view/logical_view.md) и 
[материализованных представлений](../../../overview/main_concepts/materialized_view/materialized_view.md).
Запрос обрабатывается в порядке, описанном в разделе
[Порядок обработки запросов на выгрузку данных](../../../overview/interactions/download_processing/download_processing.md).

Для получения небольшого объема данных можно использовать 
[запрос данных](../../../working_with_system/data_reading/data_reading.md).
{: .note-wrapper}

Перед выполнением запроса необходимо создать [внешнюю таблицу](../../../overview/main_concepts/external_table/external_table.md)
с указанием [пути к внешнему приемнику данных](../../path_to_kafka_topic/path_to_kafka_topic.md). Подробнее о порядке 
выполнения действий для выгрузки данных см. в разделе [Выгрузка данных](../../../working_with_system/data_download/data_download.md).
{: .note-wrapper}

В запросе можно указать [СУБД](../../../introduction/supported_DBMS/supported_DBMS.md)
[хранилища](../../../overview/main_concepts/main_concepts.md) для выгрузки данных. Если СУБД не указана, система 
определяет СУБД, [оптимальную для выгрузки](../../../working_with_system/data_reading/routing/routing.md), 
в зависимости от параметров запроса, месторасположения данных и конфигурации системы.

В ответе возвращается:
*   пустой объект ResultSet при успешном выполнении запроса;
*   исключение при неуспешном выполнении запроса.

При успешном выполнении запроса данные выгружаются в том формате и в тот приемник данных, которые были указаны 
[при создании внешней таблицы выгрузки](../CREATE_DOWNLOAD_EXTERNAL_TABLE/CREATE_DOWNLOAD_EXTERNAL_TABLE.md). Формат 
данных соответствует описанному в разделе [Формат выгрузки данных](../../download_format/download_format.md).

## Синтаксис {#syntax}

```sql
INSERT INTO [db_name.]ext_table_name SELECT query
```

Параметры:
*   `db_name` — имя логической базы данных, из которой выгружаются данные. Параметр опционален, если выбрана 
    логическая БД, [используемая по умолчанию](../../../working_with_system/other_features/default_db_set-up/default_db_set-up.md);
*   `ext_table_name` — имя внешней таблицы выгрузки;
*   <a id="param_datasource_type"></a>`query` — [SELECT](../SELECT/SELECT.md)-подзапрос для выбора выгружаемых данных. 
    Если в подзапросе указано ключевое слово `DATASOURCE_TYPE` с псевдонимом СУБД хранилища, данные выгружаются из этой 
    СУБД, иначе — из СУБД, [наиболее оптимальной для исполнения запроса](../../../working_with_system/data_reading/routing/routing.md).

## Ограничения {#restrictions}

* Имена и порядок следования столбцов должны совпадать в SELECT-подзапросе на выгрузку данных и внешней таблице выгрузки.
* Выгрузка данных, выбранных с использованием агрегатных функций, из ADQM дает некорректные результаты. Ограничение 
  связано с тем, что данные из сегментов кластера ADQM выгружаются параллельно и не объединяются.

## Пример {#examples}

### Выгрузка из СУБД, заданной в конфигурации {#default_db_example}

```sql
INSERT INTO sales.sales_ext_download
SELECT * FROM sales.sales WHERE sales.product_units > 2
```

### Выгрузка из указанной СУБД {#defined_db_example}

```sql
INSERT INTO sales.sales_ext_download 
SELECT * FROM sales.sales WHERE description = 'Покупка по акции 1+1' DATASOURCE_TYPE = 'adqm'
```

### Выгрузка из материализованного представления {#matview_example}

```sql
INSERT INTO sales.sales_by_stores_ext_download
SELECT * FROM sales.sales_by_stores WHERE product_code IN ('ABC0002', 'ABC0003', 'ABC0004') DATASOURCE_TYPE = 'adqm'
```