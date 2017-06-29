---
title: LeetCode 175.Combine Two Tables
date: 2017-05-08 11:35:44
tags:
categories: SQL
---

问题描述：

Table: `Person`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| PersonId    | int     |
| FirstName   | varchar |
| LastName    | varchar |
+-------------+---------+
PersonId is the primary key column for this table.
```

Table: `Address`

```
+-------------+---------+
| Column Name | Type    |
+-------------+---------+
| AddressId   | int     |
| PersonId    | int     |
| City        | varchar |
| State       | varchar |
+-------------+---------+
AddressId is the primary key column for this table.

```

Write a SQL query for a report that provides the following information for each person in the Person table, regardless if there is an address for each of those people:

```
FirstName, LastName, City, State
```

问题求解：

由于`regardless if there is an address for each of those people`  使用 `Person` 表左连接 `Address`表，查询指定字段：

```
SELECT P.FirstName,P.LastName,A.City,A.State 
FROM Person as P left join Address as A 
on P.PersonId=A.PersonId
```