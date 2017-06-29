---
title: MySQL-学习笔记-2
date: 2017-04-13 18:12:57
tags:
categories: SQL
---

## 数据类型及数据表操作

### 数据类型

数据类型决定存储格式

要根据实际应用来选择最合适的数据类型

<!--more-->

#### 整型

![](MySQL-学习笔记-2\6.png)

#### 浮点型

![](MySQL-学习笔记-2\7.png)

#### 日期时间型

![](MySQL-学习笔记-2\8.png)

通常用数字类型取代日期时间或进行时间戳转换

#### 字符型

![](MySQL-学习笔记-2\9.png)

注：

CHAR(M)为定长类型

VARCHAR 为变长类型

L+1或L+2里多出来的字节用来保存数据值长度

ENUM表示从枚举的值中选择其一

SET表示从成员中进行排列组合形成其值

### 数据表操作

#### 创建数据表

数据表是数据库中其他对象的基础

关系性数据库——二维表格——数据表

行——记录 列——字段

#### 打开数据库

> USE 数据库名称;

> SELECT DATABASE()； 显示当前使用的数据库

#### 创建数据表

> CREATE TABLE table_name(
>
> ​	column_name data_type,
>
> ​	……
>
>  )
>
> e.g.
>
> ```
> mysql> CREATE TABLE tb1(
> -> username VARCHAR(20),
> -> age TINYINT UNSIGNED,
> -> salary FLOAT(8,2) UNSIGNED                                                     
> -> );
> ```
>
> Query OK, 0 rows affected (0.35 sec)

#### 查看数据表

> ****SHOW TABLES[FROM db_name]
>
> [LIKE 'pattern'| WHERE expr]

注：此命令可查询其他数据库中的表且不改变当前所处数据库，如：

> mysql> show tables from mysql;
>
> +---------------------------+
>
> | Tables_in_mysql           |
> +---------------------------+
> | columns_priv              |
> | db                        |
> | engine_cost               |
> | event                     |

第二行涉及通配符，暂略

#### 查看数据表结构

> SHOW COLUMNS FROM table_name;

如：

> mysql> show columns from tb1;
> +----------+---------------------+------+-----+---------+-------+
> | Field    | Type                | Null | Key | Default | Extra |
> +----------+---------------------+------+-----+---------+-------+
> | username | varchar(20)         | YES  |     | NULL    |       |
> | age      | tinyint(3) unsigned | YES  |     | NULL    |       |
> | salary   | float(8,2) unsigned | YES  |     | NULL    |       |
> +----------+---------------------+------+-----+---------+-------+
> 3 rows in set (0.01 sec)

#### 记录插入与查找

插入记录

> INSERT [INTO] tb1_name [(col_name,...)] VALUES(val,...)

如：

> mysql> INSERT tb1 VALUES('TOM',25,7825.37);
> Query OK, 1 row affected (0.07 sec)
>
> mysql> INSERT tb1(username,salary)  VALUES('Jphn',6825.37);
> Query OK, 1 row affected (0.04 sec)

查找记录

> SELECT expr,... FROM table_name

如：

> mysql> SELECT * FROM tb1;
> +----------+------+---------+
> | username | age  | salary  |
> +----------+------+---------+
> | TOM      |   25 | 7825.37 |
> | Jphn     | NULL | 6825.37 |
> +----------+------+---------+

注：*是对字段的过滤而非对记录的过滤

#### 空值与非空

创建表格时指定某字段的值是否可以为空：

NULL,字段值可以为空

NOT NULL，字段值禁止为空

> mysql> CREATE TABLE tb2(
> -> username VARCHAR(20) NOT NULL,
> -> age TINYINT UNSIGNED NULL
> -> );

> mysql> SHOW COLUMNS FROM tb2;
> +----------+---------------------+------+-----+---------+-------+
> | Field    | Type                | Null | Key | Default | Extra |
> +----------+---------------------+------+-----+---------+-------+
> | username | varchar(20)         | NO   |     | NULL    |       |
> | age      | tinyint(3) unsigned | YES  |     | NULL    |       |
> +----------+---------------------+------+-----+---------+-------+

> mysql> INSERT tb2 VALUES(NULL,26);
> ERROR 1048 (23000): Column 'username' cannot be null

#### 自动编号

保证某条记录的唯一性——记录自动编号

![](MySQL-学习笔记-2\12.png)

即自动编号字段必须定义为数值类型，且定义为主键

如设计错误的定义：

> mysql> CREATE TABLE tb3(                                                              -> ->id SMALLINT UNSIGNED AUTO_INCREMENT,                                           -> username VARCHAR(30) NOT NULL
>
> ->);
>
> ERROR 1075 (42000): Incorrect table definition; there can be only one auto column and it must be defined as a key

#### 初涉主键约束

![](MySQL-学习笔记-2\13.png)

如修改上述记录：

```
mysql> CREATE TABLE tb3(
-> id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
-> username VARCHAR(30) NOT NULL                          -> ); 
```

> mysql> SHOW COLUMNS FROM tb3;
> +----------+----------------------+------+-----+---------+----------------+
> | Field    | Type                 | Null | Key | Default | Extra          |
> +----------+----------------------+------+-----+---------+----------------+
> | id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
> | username | varchar(30)          | NO   |     | NULL    |                |
> +----------+----------------------+------+-----+---------+----------------+

自动编号示例：

> mysql> INSERT tb3(username) VALUES('XueXue');
> Query OK, 1 row affected (0.05 sec)
>
> mysql> INSERT tb3(username) VALUES('TuTu');
> Query OK, 1 row affected (0.06 sec)
>
> mysql> INSERT tb3(username) VALUES('JingJing');
> Query OK, 1 row affected (0.02 sec)
>
> mysql> INSERT tb3(username) VALUES('GaiGai');
> Query OK, 1 row affected (0.05 sec)
>
> mysql> SELECT * FROM tb3;
> +----+----------+
> | id | username |
> +----+----------+
> |  1 | XueXue   |
> |  2 | TuTu     |
> |  3 | JingJing |
> |  4 | GaiGai   |
> +----+----------+

主键不一定要定义为自动编号，如：

> mysql> CREATE TABLE tb4(
> -> id SMALLINT UNSIGNED PRIMARY KEY,
> -> username VARCHAR(20) NOT NULL
> -> );
> Query OK, 0 rows affected (0.36 sec)
>
> mysql> SHOW COLUMNS FROM tb4;
> +----------+----------------------+------+-----+---------+-------+
> | Field    | Type                 | Null | Key | Default | Extra |
> +----------+----------------------+------+-----+---------+-------+
> | id       | smallint(5) unsigned | NO   | PRI | NULL    |       |
> | username | varchar(20)          | NO   |     | NULL    |       |
> +----------+----------------------+------+-----+---------+-------+

主键字段具有唯一性：

> mysql> INSERT tb4 VALUES(22,'Tom');
> Query OK, 1 row affected (0.05 sec)
>
> mysql> INSERT tb4 VALUES(22,'John');
> ERROR 1062 (23000): Duplicate entry '22' for key 'PRIMARY'

#### 初涉唯一约束

![](MySQL-学习笔记-2\14.png)



第二三条看似相悖，实际上一张表只允许存在一个值为空的唯一约束字段

创建示例：

```
mysql> CREATE TABLE tb5(
-> id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
-> username VARCHAR(20) NOT NULL UNIQUE KEY,
-> age TINYINT UNSIGNED
-> );
```

> mysql> SHOW COLUMNS FROM tb5;
> +----------+----------------------+------+-----+---------+----------------+
> | Field    | Type                 | Null | Key | Default | Extra          |
> +----------+----------------------+------+-----+---------+----------------+
> | id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
> | username | varchar(20)          | NO   | UNI | NULL    |                |
> | age      | tinyint(3) unsigned  | YES  |     | NULL    |                |
> +----------+----------------------+------+-----+---------+----------------+

验证唯一性：

> mysql> INSERT tb5(username,age) VALUES('John',22);
> Query OK, 1 row affected (0.05 sec)
>
> mysql> INSERT tb5(username,age) VALUES('John',22);
> ERROR 1062 (23000): Duplicate entry 'John' for key 'username'

#### 初涉默认约束

DEFAULT

创建：

> mysql> CREATE TABLE tb6(
> -> id SMALLINT UNSIGNED AUTO_INCREMENT PRIMARY KEY,
> -> username VARCHAR(20) NOT NULL UNIQUE KEY,
> -> sex ENUM('1','2','3') DEFAULT '3'
> -> );

查看：

> mysql> SHOW COLUMNS FROM tb6;
> +----------+----------------------+------+-----+---------+----------------+
> | Field    | Type                 | Null | Key | Default | Extra          |
> +----------+----------------------+------+-----+---------+----------------+
> | id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
>
> | username | varchar(20) | NO   | UNI  | NULL |      |
> | -------- | ----------- | ---- | ---- | ---- | ---- |
> |          |             |      |      |      |      |
>
> | sex      | enum('1','2','3')    | YES  |     | 3       |                |
> +----------+----------------------+------+-----+---------+----------------+ 

验证默认约束：

> mysql> INSERT  tb6(username) VALUES('Tom');
> Query OK, 1 row affected (0.05 sec)
>
> mysql> SELECT * FROM tb6;
> +----+----------+------+
> | id | username | sex  |
> +----+----------+------+
> |  1 | Tom      | 3    |
> +----+----------+------+