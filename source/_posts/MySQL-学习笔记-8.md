---
title: MySQL-学习笔记-8
date: 2017-05-11 10:37:11
tags:
categories: SQL
---

### MySQL存储过程

#### 简介

SQL命令执行流程：

![](\MySQL-学习笔记-8\1.png)

如果能省略其中语法分析和编译的过程，执行效率就能大大提高——存储过程

关于存储过程：

是SQL语句和控制语句的预编译集合，以一个名称存储并作为一个单元处理。

存储在数据库内，可以有应用程序调用执行，允许用户声明变量、进行流程控制。

可以接受参数并返回多个返回值。

执行效率高：只在第一次执行时进行语法分析和编译，再次调用时直接运用编译的结果

<!---more--->

存储过程优点：

- 增强SQL语句功能和灵活性（过程内可写控制语句，有很强灵活性，可以完成较复杂的判断、运算等）
- 实现较快执行速度（将编译结果储存在内存中）
- 减少网络流量（提交数据量减少）

#### 存储过程语法结构解析

创建存储过程：

CREATE

[DEFINER = { user|CURRENT_USER}]

PROCEDURE sp_name ([proc_parameter[,...]])

[characteristic ...]routine_body



proc_parameter:

[IN|OUT|INOUT]param_name type

参数类型：

IN：表示该参数的值必须在调用存储过程时指定
OUT：表示该参数的值可以被存储过程改变，并且可以返回
INOUT：表示该参数的值在调用时指定，并且可以被改变和返回

[characteristic ...]特性类型：

CONTAINS SQL 表示包含SQL语句但不包含读或写数据的语句

NO SQL 表示不包含SQL语句

READS SQL DATA 表示包含读数据的语句，但不包含写数据的语句

MODIFIES SQL DATA   表示包含写数据的语句

默认的是CONTAINS SQL

SQL SECURITY特征可以用来指定存储过程该用创建者的许可来执行，还是使用调用者的许可来执行。默认值是DEFINER

过程体：

- 由合法SQL语句构成
- 可以为任意增删改查或多表连接等SQL语句
- 复合结构要包含在BEGIN...END语句中

#### 创建不带参数的存储过程

创建查看当前版本信息的存储过程：

```
mysql> CREATE PROCEDURE sp1() SELECT VERSION();
```

调用存储过程：

CALL sp_name([proc_parameter[,...]])

CALL sp_name[()]

存储过程在封装时如果不带有参数，调用时可以加也可以不加小括号，但如果有参数，则调用时一定要加括号和参数,如：

```
+-----------+
| VERSION() |
+-----------+
| 5.7.15-1  |
+-----------+

mysql> CALL sp1();
+-----------+
| VERSION() |
+-----------+
| 5.7.15-1  |
+-----------+
```

#### 创建带有IN类型参数的存储过程

错误示范：

```
CREATE PROCEDURE removeUserById(IN id INT UNSIGNED)
    -> BEGIN
    -> DELETE FROM users WHERE id = id;
    -> END
    -> $$
```

使用该存储过程从某张表中删除记录会导致所有记录被删除，原因是参数名称不能和记录表中的字段名称相同。

修改存储过程可以修改注释，内容类型但无法修改过程体。

正确创建过程：

```
mysql> DELIMITER $$                                                               mysql> CREATE PROCEDURE removeUser(IN p_id INT UNSIGNED)
    -> BEGIN
    -> DELETE FROM users WHERE id=p_id;
    -> END
    -> $$
```

删除一条记录:

```
mysql> CALL removeUser(7); 
Query OK, 0 rows affected (0.00 sec)
```

查看记录是否被删除

```
mysql> SELECT * FROM users WHERE id=7;
Empty set (0.00 sec)
```

#### 创建带有IN和OUT类型参数的存储过程

对上一个存储过程添加功能：删除记录并返回剩余记录数：

```
mysql> CREATE PROCEDURE RUARU(IN p_id INT UNSIGNED,OUT userNums INT UNSIGNED)     
	-> BEGIN                                                                         
	-> DELETE FROM users WHERE id=P_id;
    -> SELECT count(id) FROM users INTO userNums;
    -> END                                                                            
    -> $$
```

调用存储过程，并创建一个变量来接收OUT值：

```
mysql> CALL RUARU(12,@nums);
Query OK, 1 row affected (0.04 sec)
mysql> SELECT @nums;
+-------+
| @nums |
+-------+
|    16 |
+-------+
```

在过程体或函数体中声明的变量为局部变量

而通过SELECT、SELECT...INTO、SET语句声明的变量为用户变量，可对当前客户端生效。

#### 创建带有多个OUT类型参数的存储过程

创建根据年龄删除用户的存储过程，返回被删除的记录数和剩余的记录数。

row_count()函数：返回插入、删除、更新（即被影响）的记录总数。

创建存储过程：

```
mysql> DELIMITER //
mysql> CREATE PROCEDURE RUARI(IN p_age SMALLINT UNSIGNED,OUT deleteUsers SMALLINT UNSIGNED,OUT userCount SMALLINT UNSIGNED)
    -> BEGIN
    -> DELETE FROM users WHERE age=p_age;
    -> SELECT ROW_COUNT() INTO deleteUsers;
    -> SELECT COUNT(id) FROM users INTO userCount;
    -> END
    -> //
```

删除年龄为23的记录:

```
mysql> SELECT count(id) FROM users WHERE age=23;
+-----------+
| count(id) |
+-----------+
|         5 |
+-----------+
mysql> CALL RUARI(23,@a,@b);
Query OK, 1 row affected (0.04 sec)
mysql> SELECT count(id) FROM users WHERE age=23;
+-----------+
| count(id) |
+-----------+
|         0 |
+-----------+
```

查看返回的结果：

```
mysql> SELECT @a,@b;
+------+------+
| @a   | @b   |
+------+------+
|    5 |    8 |
+------+------+
```