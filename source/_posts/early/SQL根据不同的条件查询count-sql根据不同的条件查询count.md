---
title: SQL根据不同的条件查询count
date: 2020-04-15 16:51:57.0
updated: 2021-01-08 17:05:04.56
url: https://maoxian.fun/archives/sql根据不同的条件查询count
categories: 
- 程序
- Sql
tags: 
- 程序
- 代码
- Sql
---

在一条语句中根据不同的条件count对应的数据。

```sql
select count(if(条件, true, null)), count(if(条件, true, null)) from table;
```

以上语句可以根据两个不同的条件一次计算出对应的count值，很明显的用到的是IF函数。IF( expr1, expr2, expr3)。

> 以下来自[官方文档](https://dev.mysql.com/doc/refman/5.7/en/control-flow-functions.html)的说明
>
> If expr1 is TRUE (expr1 <> 0 and expr1 <> NULL), IF() returns expr2. Otherwise, it returns expr3.

用到count中，即if的条件为真，则if子句返回true，否则返回null。count子句依据该返回值进行计数。

具体的条件依据需求变化，例如需要去重，可以变为count(distinct if(条件, expr2, expr3))

同理，if也可用于sum等其他聚合函数计算。

**注意：**

*第一条sql语句中的返回值不一定为true, null 可依据需求调整，该文章仅作参考。*

*该文章测试环境为* ***Mysql 5.7.25***