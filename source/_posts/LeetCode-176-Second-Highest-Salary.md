---
title: LeetCode 176.Second Highest Salary
date: 2017-05-08 12:40:30
tags:
categories: SQL
---

问题描述：

Write a SQL query to get the second highest salary from the `Employee` table.

```
+----+--------+
| Id | Salary |
+----+--------+
| 1  | 100    |
| 2  | 200    |
| 3  | 300    |
+----+--------+

```

For example, given the above Employee table, the second highest salary is `200`. If there is no second highest salary, then the query should return `null`.

问题求解：

可以先查询出最高的Salary，再从原表中选取Salary低于此最高值的记录作为结果，最后再利用max函数选取结果中的最大值，即第二高的Salary。

注意提交结果时使用题目指定的别名。

```
SELECT MAX(Salary) AS SecondHighestSalary 
FROM Employee 
WHERE Salary<(SELECT MAX(Salary) FROM Employee);
```