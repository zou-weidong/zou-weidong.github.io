---
title: "Postgresql"
linkTitle: "Postgresql"
weight: 1
description: >
  Postgresql
---

{{% pageinfo %}}
Postgresql
{{% /pageinfo %}}

##  使用
查询sql跟mysql一样

```sql
-- 查库
SELECT datname FROM pg_database;

-- 查表
SELECT * FROM pg_tables WHERE schemaname = 'public';
```