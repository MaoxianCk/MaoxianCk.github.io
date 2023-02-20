---
title: SQL查询表中是否存在符合条件的记录
date: 2020-02-14 16:51:52.0
updated: 2021-01-08 17:01:30.639
url: https://maoxian.fun/archives/sql查询表中是否存在符合条件的记录
categories: 
- 程序
- Sql
tags: 
- 程序
- 代码
- Sql
---

判断记录是否存在，最主要的问题就是性能问题

话不多说，直接上结果

```
-- 存在返回 1, 不存在返回 0
select ifnull((select 1 from tableName where conditions limit 1 ), 0)﻿ as existed
```

ifnull 函数：如果第一个参数值为null，返回第二个参数的值，如果不为 null 则返回第一个参数的值

错误示范：

```
select COUNT(*) from tableName where conditions
```

根据结果的数量进行比较，简单易懂。但是count(*)统计全表数量，性能开销较大