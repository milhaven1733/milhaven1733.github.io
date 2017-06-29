---
title: LeetCode 182.Duplicate Emails
date: 2017-05-08 11:23:43
tags:
categories: SQL
---

问题描述：

Write a SQL query to find all duplicate emails in a table named `Person`.

```
+----+---------+
| Id | Email   |
+----+---------+
| 1  | a@b.com |
| 2  | c@d.com |
| 3  | a@b.com |
+----+---------+

```

For example, your query should return the following for the above table:

```
+---------+
| Email   |
+---------+
| a@b.com |
+---------+
```

问题求解：

按Email分组查询并筛选出记录数目大于1的分组，选择其Email字段：

```
SELECT Email FROM Person 
GROUP BY Email 
HAVING count(Email)>1;
```