---
title: MySQL-学习笔记-4
date: 2017-04-20 10:34:13
tags:
categories: SQL
---

### 操作数据表中的记录

本节重点：对于数据表中记录的增、删、改、查。

#### 插入记录INSERT

插入记录

```
INSERT [INTO] table_name [(col_name,...)] {VALUES|VALUE} ({expr|DEFAULT},...),(...),...
```

- 列名称省略，代表所有字段依次赋值
- 值可以为表达式或默认值，字段之间逗号分隔
- 可以一次插入多条记录

<!----more---->

对于自动编号的字段，可以用NULL或DEFAULT赋值，实现自动编号，如：

```
mysql> INSERT user_new VALUES(NULL,'John','1111',25,1);
mysql> INSERT user_new VALUES(DEFAULT,'John','1111',25,1);
```

赋值时可以采用表达式，对于包含默认值的字段，可以用DEFAULT使之保持默认值：

```
mysql> INSERT user_new VALUES(DEFAULT,'Tom','1111',3*7-2,1);
mysql> INSERT user_new VALUES(DEFAULT,'Tom','2222',DEFAULT,1);
```

以上指令操作结果：

```
mysql> SELECT * FROM user_new;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  1 | John     | 1111     |  25 |    1 |
|  2 | John     | 1111     |  25 |    1 |
|  3 | Tom      | 1111     |  19 |    1 |
|  4 | Tom      | 2222     |  10 |    1 |
+----+----------+----------+-----+------+
```

可以一次插入多条记录，且值可以返回自函数：

```
mysql> INSERT user_new VALUES(DEFAULT,'Tom','2222',DEFAULT,1),(NULL,'Rose',md5('123'),DEFAULT,0);
mysql> SELECT * FROM user_new;                                                    
+----+----------+----------------------------------+-----+------+
| id | username | password                         | age | sex  |
+----+----------+----------------------------------+-----+------+
|  1 | John     | 1111                             |  25 |    1 |
|  2 | John     | 1111                             |  25 |    1 |
|  3 | Tom      | 1111                             |  19 |    1 |
|  4 | Tom      | 2222                             |  10 |    1 |
|  5 | Tom      | 2222                             |  10 |    1 |
|  6 | Rose     | 202cb962ac59075b964b07152d234b70 |  10 |    0 |
+----+----------+----------------------------------+-----+------+
```

#### 插入记录 INSERT SET/SELECT

插入记录：

```
INSERT [INTO] table_name SET col_name={expr|DEFAULT},...
```

区别：

- 此方法可以引发子查询（SubQuery）

  【由比较运算符引发的子查询，是引发子查询的三种方式之一】

- 只能一次性插入一条记录

例如：

```
mysql> INSERT user_new SET username='Bob',password='456';
```

id字段自动编号，age字段有默认值，sex字段可以为空，可以均不赋值。

插入记录：

```
INSERT [INTO] table_name [(col_name,...)] SELECT...
```

此方法可以将查询结果插入到指定数据表。（详见最后笔记底部）

#### 单表更新记录UPDATE

更新记录（单表更新）

```
UPDATE [LOW_PRIORITY][IGNORE] table_reference SET col_name1={expr1|DEFAULT}[,col_name2={expr2|DEFAULT}]...[WHERE where_condition]
```

- 可选项：降低优先权、处理更新过程中的错误中断等。
- 参照关系：在单表操作中只能为某一张表
- 省略WHERE条件，则所有的记录都被更新

如：

```
mysql> UPDATE user_new set age=age+5;
```

将所有记录的age值加5.

可以同时更新多个字段：

```
mysql> UPDATE user_new set age=age-1,sex=0;
mysql> SELECT * FROM user_new;
+----+----------+----------------------------------+-----+------+
| id | username | password                         | age | sex  |
+----+----------+----------------------------------+-----+------+
|  1 | John     | 1111                             |  29 |    0 |
|  2 | John     | 1111                             |  29 |    0 |
|  3 | Tom      | 1111                             |  23 |    0 |
|  4 | Tom      | 2222                             |  14 |    0 |
|  5 | Tom      | 2222                             |  14 |    0 |
|  6 | Rose     | 202cb962ac59075b964b07152d234b70 |  14 |    0 |
|  7 | Bob      | 456                              |  14 |    0 |
+----+----------+----------------------------------+-----+------+
```

可以在WHERE之后加入条件，如：

```
mysql> UPDATE user_new set age=age+5 WHERE id%2=0;
```

**注意：判断取余结果要用“=” 不是“==”**

可以看到，id为偶数的记录被更新了。

```
mysql> SELECT * FROM user_new;
+----+----------+----------------------------------+-----+------+
| id | username | password                         | age | sex  |
+----+----------+----------------------------------+-----+------+
|  1 | John     | 1111                             |  29 |    0 |
|  2 | John     | 1111                             |  34 |    0 |
|  3 | Tom      | 1111                             |  23 |    0 |
|  4 | Tom      | 2222                             |  19 |    0 |
|  5 | Tom      | 2222                             |  14 |    0 |
|  6 | Rose     | 202cb962ac59075b964b07152d234b70 |  19 |    0 |
|  7 | Bob      | 456                              |  14 |    0 |
+----+----------+----------------------------------+-----+------+

```

#### 单表删除记录DELETE

删除记录（单表删除）

```
DELETE FROM table_name [WHERE where_condition]
```

如：

```
mysql> DELETE FROM user_new WHERE id=6;
```

id为６的记录被删除。但需要注意，当此时再插入一条新纪录，id为当前最大id值加１，而非补充被删去的６：

```
mysql> INSERT user_new VALUES(NULL,'Tom','2222',33,NULL);
mysql> SELECT * FROM user_new;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  1 | John     | 1111     |  29 |    0 |
|  2 | John     | 1111     |  34 |    0 |
|  3 | Tom      | 1111     |  23 |    0 |
|  4 | Tom      | 2222     |  19 |    0 |
|  5 | Tom      | 2222     |  14 |    0 |
|  7 | Bob      | 456      |  14 |    0 |
|  8 | Tom      | 2222     |  33 | NULL |
+----+----------+----------+-----+------+
```

#### 查询表达式解析

SELECT语句在针对表操作的语句中使用率非常高。

SELECT语句语法：

![](MySQL-学习笔记-4\1.png)

-    SELECT语句可以只书写表达式，如：

     ```
     SELECT VERSION();    SELECT NOW();  不依附于任何一张表	
     ```

     SELECE语句查询表达式：

![](MySQL-学习笔记-4\2.png)

**SELECT 查询表达式顺序影响结果顺序：**

```
mysql> SELECT id,username FROM user_new;
+----+----------+
| id | username |
+----+----------+
|  1 | John     |
|  2 | John     |
|  3 | Tom      |
|  4 | Tom      |
|  5 | Tom      |
|  7 | Bob      |
|  8 | Tom      |
+----+----------+
mysql> SELECT username,id FROM user_new;
+----------+----+
| username | id |
+----------+----+
| John     |  1 |
| John     |  2 |
| Tom      |  3 |
| Tom      |  4 |
| Tom      |  5 |
| Bob      |  7 |
| Tom      |  8 |
+----------+----+
```

table_name.colume_name表示某表的某列，table_name.*表示该表所有列（多表连接操作中需要明确某列隶属于某表）。

可以使用 as alias_name 使用别名，如：（as可以书写，也可以不写）

```
mysql> SELECT id AS userid,username AS uname FROM user_new;
+--------+-------+
| userid | uname |
+--------+-------+
|      1 | John  |
|      2 | John  |
|      3 | Tom   |
|      4 | Tom   |
|      5 | Tom   |
|      7 | Bob   |
|      8 | Tom   |
+--------+-------+
```

**字段的别名将影响结果集。**

#### where语句进行条件查询

条件表达式：

对记录进行过滤，如果没有指定where语句，则显示所有记录（同理，在UPDATE语句和DELETE语句中将更新和删除所有记录）

在WHERE表达式中，可以使用MySQL支持的函数（如数学、字符函数）和运算符。

#### group by语句对查询结果分组

查询结果分组：

```
[GROUP BY {cool_name|position} [ASC|DESC],...]
```

ASC表示分组结果升序排列（默认），DESC表示分组结果降序排列。

如：

```
mysql> SELECT sex FROM user_new GROUP BY sex;
+------+
| sex  |
+------+
| NULL |
|    0 |
+------+
mysql> SELECT sex FROM user_new GROUP BY sex DESC;
+------+
| sex  |
+------+
|    0 |
| NULL |
+------+
```

#### having语句设置分组筛选条件

分组条件

```
[HAVING where_condition]
```

用HAVING字句做分组筛选的条件时，HAVING后的分组条件需要为聚合函数或条件中字段出现在SELECT之后

聚合函数：永远只有一个返回结果的函数，如常规的min、max、avg、sum、count等

如：

```
mysql> SELECT sex FROM user_new GROUP BY sex HAVING count(id)>2;
+------+
| sex  |
+------+
|    0 |
+------+
```

分组筛选的结果剔除了SEX是NULL的结果，其原因是SEX=NULL的结果只有一个，不满足count(id)>2的条件。

#### order by语句对查询结果排序

对查询结果进行排序：

```
[ORDER BY {col_name|expr|position}[ASC|DESC],...]
```

可以添加多个排序条件，当按照第一个字段排列后有重复值，再按照第二个字段排列，以此类推。如：

```
mysql> SELECT * FROM user_new ORDER BY age;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  5 | Tom      | 2222     |  14 |    0 |
|  7 | Bob      | 456      |  14 |    0 |
|  4 | Tom      | 2222     |  19 |    0 |
|  3 | Tom      | 1111     |  23 |    0 |
|  1 | John     | 1111     |  29 |    0 |
|  8 | Tom      | 2222     |  33 | NULL |
|  2 | John     | 1111     |  34 |    0 |
+----+----------+----------+-----+------+
mysql> SELECT * FROM user_new ORDER BY age,id DESC;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  7 | Bob      | 456      |  14 |    0 |
|  5 | Tom      | 2222     |  14 |    0 |
|  4 | Tom      | 2222     |  19 |    0 |
|  3 | Tom      | 1111     |  23 |    0 |
|  1 | John     | 1111     |  29 |    0 |
|  8 | Tom      | 2222     |  33 | NULL |
|  2 | John     | 1111     |  34 |    0 |
+----+----------+----------+-----+------+
```

#### limit语句限制查询数量

限制查询结果返回的数量

```
[LIMIT {[offset,]row_count|row_count OFFSET offset}]
```

返回查询结果中的前两条：

```
mysql> SELECT * FROM user_new LIMIT 2;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  1 | John     | 1111     |  29 |    0 |
|  2 | John     | 1111     |  34 |    0 |
+----+----------+----------+-----+------+
```

返回从第2条起的2条记录：（记录从0开始编号）

```
mysql> SELECT * FROM user_new LIMIT 2,2;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  3 | Tom      | 1111     |  23 |    0 |
|  4 | Tom      | 2222     |  19 |    0 |
+----+----------+----------+-----+------+
```

还要注意，id号和结果中的排号无联系，如：

```
mysql> SELECT * FROM user_new ORDER BY id DESC LIMIT 2;
+----+----------+----------+-----+------+
| id | username | password | age | sex  |
+----+----------+----------+-----+------+
|  8 | Tom      | 2222     |  33 | NULL |
|  7 | Bob      | 456      |  14 |    0 |
+----+----------+----------+-----+------+
```

#### 将查询结果插入到表中

可以讲查询返回的结果插入到指定表中，如：

新建一张表：

```
mysql> CREATE TABLE test(
    -> id TINYINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> username VARCHAR(20)                                                           -> );
```

将查询到的字段插入到新建表中：

```
mysql> INSERT test(username) SELECT username FROM user_new WHERE age>=25;
Query OK, 3 rows affected (0.04 sec)
```

查看:

```
mysql> SELECT * FROM test;
+----+----------+
| id | username |
+----+----------+
|  1 | John     |
|  2 | John     |
|  3 | Tom      |
+----+----------+
```