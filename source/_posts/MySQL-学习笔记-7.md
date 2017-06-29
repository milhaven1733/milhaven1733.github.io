---
title: MySQL-学习笔记-7
date: 2017-05-06 14:54:18
tags:
categories: SQL
---

### 自定义函数

#### 关于自定义函数

用户自定义函数（user-defined function UDF）是一种对MySQL扩展的途径，其用法和内置函数相同。 

必要条件：

- 参数（可以有零个，一个、多个）
- 返回值（均具有）

<!---more--->

函数可以返回任意类型的值，同样可以接受这些类型的参数

MySQL中的自定义函数最多支持1024个参数。。。

创建自定义函数：

CREATEFUNCTION function_name

RETURNS

{STRING|INTEGER|REAL|DECIMAL}

routine_body  (函数体) 

关于函数体：

- 由合法的SQL语句构成；
- 可以是简单的SELECT或INSERT语句；
- 函数体如果为复合结构则使用BEGIN…END语句；
- 复合结构可以包含声明、循环、控制结构；

#### 创建不带参数的自定义函数

设置客户端编码格式（方便显示汉字）

````
mysql> set NAMES utf8;
````

日期格式转化功能：

```
mysql> SELECT DATE_FORMAT(NOW(),'%Y年%m月%d日 %H点%i分%s秒') DATE;
+-----------------------------------+
| DATE                              |
+-----------------------------------+
| 2017年05月06日 17点09分27秒       |
+-----------------------------------+
```

将以上命令编写成函数：

```
mysql> CREATE FUNCTION f1() RETURNS VARCHAR(30) RETURN DATE_FORMAT(NOW(),'%Y年%m月%d日 %H点%i分%s秒');
Query OK, 0 rows affected (0.01 sec)
mysql> SELECT f1();                                                               
+-----------------------------------+
| f1()                              |
+-----------------------------------+
| 2017年05月06日 17点07分11秒       |
+-----------------------------------+
```

#### 创建带有参数的自定义函数

```
mysql> CREATE FUNCTION f2(num1 SMALLINT UNSIGNED,num2 SMALLINT UNSIGNED)
    -> RETURNS FLOAT(10,2) UNSIGNED
    -> RETURN (num1+num2)/2;
mysql> SELECT f2(10,13);
+-----------+
| f2(10,13) |
+-----------+
|     11.50 |
+-----------+
```

#### 创建带有复合结构的函数体

首先修改SQL命令结束符（使函数定义内部语句可以使用分号但不被认为是命令结束）：

```
mysql> DELIMITER $$
```

结束符被修改为“$$”

函数有多个语句需要执行时，需将函数体包含在BEGIN...END之中：

```
mysql> CREATE FUNCTION adduser(username VARCHAR(20))                                  -> 		-> RETURNS INT UNSIGNED
    -> BEGIN
    -> INSERT test(username) VALUES(username);
    -> RETURN LAST_INSERT_ID();
    -> END$$
```

测试函数功能：

```
mysql> SELECT adduser('Rose')$$                                                   
+-----------------+
| adduser('Rose') |
+-----------------+
|               5 |
+-----------------+
```

```
mysql> SELECT * FROM test$$
+----+----------+
| id | username |
+----+----------+
|  1 | John     |
|  2 | John     |
|  3 | Tom      |
|  5 | Rose     |
+----+----------+
```

