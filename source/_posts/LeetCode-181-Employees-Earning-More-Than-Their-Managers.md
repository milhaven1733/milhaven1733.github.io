---
title: LeetCode 181.Employees Earning More Than Their Managers
date: 2017-05-08 11:41:49
tags:
categories: SQL
---

问题描述：

The `Employee` table holds all employees including their managers. Every employee has an Id, and there is also a column for the manager Id.

```
+----+-------+--------+-----------+
| Id | Name  | Salary | ManagerId |
+----+-------+--------+-----------+
| 1  | Joe   | 70000  | 3         |
| 2  | Henry | 80000  | 4         |
| 3  | Sam   | 60000  | NULL      |
| 4  | Max   | 90000  | NULL      |
+----+-------+--------+-----------+

```

Given the `Employee` table, write a SQL query that finds out employees who earn more than their managers. For the above table, Joe is the only employee who earns more than his manager.

```
+----------+
| Employee |
+----------+
| Joe      |
+----------+
```

问题求解:

可以对` Employee` 表自身进行连接，连接后选取满足限定条件“employee who earns more than his manager”的记录的Name字段值。

注意：一定按照题目要求为查询结果指定别名。

```
SELECT e.Name as Employee FROM Employee as e 
join Employee as m 
on e.ManagerId=m.ID 
WHERE e.Salary>m.Salary;
```