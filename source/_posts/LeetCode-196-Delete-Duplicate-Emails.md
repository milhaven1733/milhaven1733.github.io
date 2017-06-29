---
title: LeetCode 196.Delete Duplicate Emails
date: 2017-05-08 12:19:29
tags:
categories: SQL
---

问题描述：

Write a SQL query to delete all duplicate email entries in a table named `Person`, keeping only unique emails based on its *smallest* **Id**.

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
| 3  | john@example.com |
+----+------------------+
Id is the primary key column for this table.

```

For example, after running your query, the above `Person` table should have the following rows:

```
+----+------------------+
| Id | Email            |
+----+------------------+
| 1  | john@example.com |
| 2  | bob@example.com  |
+----+------------------+
```

问题求解：

要求在原表中对Email重复的记录进行删除，仅保留Id值最小的一条记录。

涉及到多表删除。

先根据Email进行分组，筛选出组内记录数大于1（即包含重复记录）的分组，选取组内最小Id值和对应Email字段作为一张新表。（则新表包含唯一Email记录）

对照新表删去原表中Email字段相同但ID号大于新表的记录（即得到与新表相同的结果）

```
DELETE t1 FROM Person as t1 ,
(SELECT MIN(Id) as ID,Email FROM Person GROUP BY Email HAVING count(Email)>1) as t2 
WHERE t1.Email=t2.Email 
AND t1.ID>t2.ID;
```

也可以连接后从原表删除指定记录：

```
DELETE t1 FROM Person as t1 
left join (SELECT MIN(Id) as ID,Email FROM Person GROUP BY Email HAVING count(Email)>1) as t2
on t1.Email=t2.Email 
WHERE t1.ID>t2.ID;
```

