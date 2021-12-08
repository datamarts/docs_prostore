---
layout: default
title: Хранилище данных
nav_order: 8
parent: Основные понятия
grand_parent: Обзор понятий, компонентов и связей
has_children: false
has_toc: false
---

# Хранилище данных {#data_storage}

_Физическое хранилище данных (далее — хранилище)_ — совокупность 
[СУБД различных типов](../../../introduction/supported_DBMS/supported_DBMS.md), 
в которых хранятся данные [логических таблиц](../logical_table/logical_table.md) и 
[материализованных представлений](../materialized_view/materialized_view.md). 
Данные в хранилище хранятся в соответствии с [физической схемой данных](../physical_schema/physical_schema.md).

Система предоставляет единый интерфейс взаимодействия с хранилищем в виде [логической базы данных](../logical_db/logical_db.md).