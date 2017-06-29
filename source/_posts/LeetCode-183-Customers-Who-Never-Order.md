---
title: LeetCode 183.Customers Who Never Order
date: 2017-05-08 11:50:39
tags:
categories: SQL
---

问题描述：

Suppose that a website contains two tables, the `Customers` table and the `Orders` table. Write a SQL query to find all customers who never order anything.

Table: `Customers`.

```
+----+-------+
| Id | Name  |
+----+-------+
| 1  | Joe   |
| 2  | Henry |
| 3  | Sam   |
| 4  | Max   |
+----+-------+

```

Table: `Orders`.

```
+----+------------+
| Id | CustomerId |
+----+------------+
| 1  | 3          |
| 2  | 1          |
+----+------------+

```

Using the above tables as example, return the following:

```
+-----------+
| Customers |
+-----------+
| Henry     |
| Max       |
+-----------+
```

问题求解：

可以用`Customers`表左外连接 `Orders` 表，然后使用条件语句查询连接结果中O.Id字段值为NULL的记录：

```
SELECT C.Name as Customers FROM Customers C 
left Join Orders O 
on C.Id=O.CustomerId 
WHERE O.Id IS NULL;
```

也可以使用NOT IN:

```
SELECT C.Name as Customers FROM Customers C 
WHERE C.Id NOT IN (SELECT CustomerId FROM Orders);
```