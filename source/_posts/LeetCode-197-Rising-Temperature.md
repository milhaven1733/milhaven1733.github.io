---
title: LeetCode 197.Rising Temperature
date: 2017-05-08 12:06:04
tags:
categories: SQL
---

问题描述：

Given a `Weather` table, write a SQL query to find all dates' Ids with higher temperature compared to its previous (yesterday's) dates.

```
+---------+------------+------------------+
| Id(INT) | Date(DATE) | Temperature(INT) |
+---------+------------+------------------+
|       1 | 2015-01-01 |               10 |
|       2 | 2015-01-02 |               25 |
|       3 | 2015-01-03 |               20 |
|       4 | 2015-01-04 |               30 |
+---------+------------+------------------+

```

For example, return the following Ids for the above Weather table:

```
+----+
| Id |
+----+
|  2 |
|  4 |
+----+
```

问题求解：

可以对 `Weather` 表相邻两天的记录进行连接，筛选出前一天气温低于后一天的记录

```
SELECT b.Id FROM Weather AS a 
join Weather AS b 
ON TO_DAYS(a.DATE)=TO_DAYS(b.DATE)-1 
WHERE a.Temperature<b.Temperature;
```

可以在连接时加以限定：

```
SELECT b.Id FROM Weather a 
join Weather b 
ON TO_DAYS(a.DATE)=TO_DAYS(b.DATE)-1 
AND a.Temperature<b.Temperature;
```

也可以不连接直接进行多表查询：

```
SELECT b.Id FROM Weather a ,Weather b 
WHERE TO_DAYS(a.DATE)=TO_DAYS(b.DATE)-1 
AND a.Temperature<b.Temperature;
```