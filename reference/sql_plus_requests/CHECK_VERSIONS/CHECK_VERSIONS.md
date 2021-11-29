---
layout: default
title: CHECK_VERSIONS
nav_order: 9
parent: Запросы SQL+
grand_parent: Справочная информация
has_children: false
has_toc: false
---

# CHECK_VERSIONS

Запрос позволяет получить информацию о версиях следующих программных компонентов:
*   [компонентов системы](../../../overview/components/components.md),
*   компонентов, с которыми работает система.

В ответе возвращается:
*   объект ResultSet с записями, содержащими информацию об именах и версиях компонентах, при успешном 
    выполнении запроса (см. рисунок ниже);
*   исключение при неуспешном выполнении запроса.

![](check_versions.png)
{: .figure-center}
*Ответ CHECK_VERSIONS*
{: .figure-caption-center}

## Синтаксис {#syntax}

```sql
CHECK_VERSIONS()
```