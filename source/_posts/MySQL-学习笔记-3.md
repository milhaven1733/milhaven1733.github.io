---
title: MySQL-学习笔记-3
date: 2017-04-16 09:23:42
tags:
categories: SQL
---

### MySQL约束及修改数据表

#### 外键约束的要求解析

**约束**

![1](MySQL-学习笔记-3\1.png)

根据约束针对字段多少：

针对一个字段——列级约束

针对多个字段——表级约束

<!--more-->

**外键约束**

![](MySQL-学习笔记-3\2.png)

![](MySQL-学习笔记-3\3.png)

注：当外键列无索引时，MySQL自动创建索引，而当参照列无索引时，不会自动创建。

**查看MySQL当前提供引擎**

```
mysql> show engines;
```

**查看MySQL默认存储引擎**

```
mysql> show variables like '%storage_engine%';
+----------------------------------+--------+
| Variable_name                    | Value  |
+----------------------------------+--------+
| default_storage_engine           | InnoDB |
| default_tmp_storage_engine       | InnoDB |
| disabled_storage_engines         |        |
| internal_tmp_disk_storage_engine | InnoDB |
+----------------------------------+--------+
```

**创建父表：**

```
mysql> CREATE TABLE provinces(
    -> id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> pname VARCHAR(20) NOT NULL
    -> );
```

**查看创建父表时的存储引擎**：

``` 
mysql> SHOW CREATE TABLE provinces;
+-----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table     | Create Table                                                                                                                                                                |
+-----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| provinces | CREATE TABLE `provinces` (
  `id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,
  `pname` varchar(20) NOT NULL,
  PRIMARY KEY (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-----------+-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

**创建子表（错误示例）**

``` 
mysql> CREATE TABLE user(
    -> id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> username VARCHAR(10) NOT NULL,
    -> pid BIGINT,
    -> FOREIGN KEY (pid) REFERENCES provinces (id)
    -> );
ERROR 1215 (HY000): Cannot add foreign key constraint
```

可见，外键列必须与主键保持相同数据类型

**创建子表：**

``` 
mysql> CREATE TABLE users(
    -> id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> username VARCHAR(10) NOT NULL,
    -> pid SMALLINT UNSIGNED,
    -> FOREIGN KEY (pid) REFERENCES provinces (id)
    -> );
```

**查看子表创建时的引擎和外键的创建：**

``` 
mysql> SHOW CREATE TABLE users;
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Table | Create Table                        |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| users | CREATE TABLE `users` (
  `id` smallint(5) unsigned NOT NULL AUTO_INCREMENT,
  `username` varchar(10) NOT NULL,
  `pid` smallint(5) unsigned DEFAULT NULL,
  PRIMARY KEY (`id`),
  KEY `pid` (`pid`),
  CONSTRAINT `users_ibfk_1` FOREIGN KEY (`pid`) REFERENCES `provinces` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
+-------+---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
```

**查看父表索引：**

```
mysql> SHOW INDEXES FROM provinces\G
*************************** 1. row ***************************
        Table: provinces
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
```

可见：参照列　‘id’　为主键，已自动创建索引

\G表示以网格形式显示

**查看子表索引**

```
mysql> SHOW INDEXES FROM users\G
*************************** 1. row ***************************
        Table: users
   Non_unique: 0
     Key_name: PRIMARY
 Seq_in_index: 1
  Column_name: id
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: 
   Index_type: BTREE
      Comment: 
Index_comment: 
*************************** 2. row ***************************
        Table: users
   Non_unique: 1
     Key_name: pid
 Seq_in_index: 1
  Column_name: pid
    Collation: A
  Cardinality: 0
     Sub_part: NULL
       Packed: NULL
         Null: YES
   Index_type: BTREE
      Comment: 
Index_comment: 
```

可见，子表存在两个索引：子表主键的索引以及外键的索引

#### 外键约束的参照操作

![](MySQL-学习笔记-3\4.png)

创建子表，指定外键的参照操作：

```
mysql> CREATE TABLE user1(
    -> id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> username VARCHAR(10) NOT NULL,
    -> pid SMALLINT UNSIGNED,
    -> FOREIGN KEY (pid) REFERENCES provinces (id) ON DELETE CASCADE
    -> );
```

**在父表中插入记录**

``` 
mysql> INSERT provinces(pname) VALUE('A');
Query OK, 1 row affected (0.06 sec)

mysql> INSERT provinces(pname) VALUE('B');
Query OK, 1 row affected (0.04 sec)

mysql> INSERT provinces(pname) VALUE('C');
Query OK, 1 row affected (0.09 sec)

mysql> SELECT * FROM provinces;
+----+-------+
| id | pname |
+----+-------+
|  1 | A     |
|  2 | B     |
|  3 | C     |
+----+-------+
```

**在子表中插入记录**

``` 
mysql> INSERT user1(username,pid) VALUES('Tom',3);
Query OK, 1 row affected (0.03 sec)
```

``` 
mysql> INSERT user1(username,pid) VALUES('John',5);
ERROR 1452 (23000): Cannot add or update a child row: a foreign key constraint fails (`lcyDB`.`user1`, CONSTRAINT `user1_ibfk_1` FOREIGN KEY (`pid`) REFERENCES `provinces` (`id`) ON DELETE CASCADE)
```

可见：外键列不能设为父表中参照列不存在的值

```
mysql> INSERT user1(username,pid) VALUES('John',1);
Query OK, 1 row affected (0.05 sec)
mysql> INSERT user1(username,pid) VALUES('Rose',2);
Query OK, 1 row affected (0.05 sec)
mysql> SELECT * FROM user1;
+----+----------+------+
| id | username | pid  |
+----+----------+------+
|  1 | Tom      |    3 |
|  4 | John     |    1 |
|  5 | Rose     |    2 |
+----+----------+------+
```

删除父表中某记录，查看子表相应行是否改变：

``` 
mysql> DELETE FROM provinces WHERE id=3;
Query OK, 1 row affected (0.05 sec)
mysql> SELECT * FROM user1;
+----+----------+------+
| id | username | pid  |
+----+----------+------+
|  4 | John     |    1 |
|  5 | Rose     |    2 |
+----+----------+------+
```

可见：子表已自动删除id为３的记录

**但因为物理的外键约束只有INNODB引擎才支持，在实际的开发过程中，我们很少使用物理的外键约束，大多使用逻辑的外键约束。所以说，我们在实际的项目开发中，一般定义逻辑的外键，指的是在定义两张表结构时，按照存在的某种结构的方式去定义，但是不使用FOREIGN KEY这个关键词**

#### 表级约束和列级约束

![](MySQL-学习笔记-3\5.png)

如之前创建的外键约束为列级约束，可以在列定义时声明

在实际开发中，用列级约束比较多，表级约束很少用。

#### 修改数据表——添加\删除列

**添加单列**

ALTER TABLE table_name ADD [COLUMN] col_name colume_definition [FIRST|AFTER col_name]

如：

``` 
mysql> ALTER TABLE user1 ADD age  TINYINT UNSIGNED NOT NULL DEFAULT 10;
Query OK, 0 rows affected (0.68 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW COLUMNS FROM user1;
+----------+----------------------+------+-----+---------+----------------+
| Field    | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
| username | varchar(10)          | NO   |     | NULL    |                |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |                |
| age      | tinyint(3) unsigned  | NO   |     | 10      |                |
+----------+----------------------+------+-----+---------+----------------+
```

指定AFTER:

```
mysql> ALTER TABLE user1 ADD password VARCHAR(20) NOT NULL AFTER username;
Query OK, 0 rows affected (0.76 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW COLUMNS FROM user1;
+----------+----------------------+------+-----+---------+----------------+
| Field    | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
| username | varchar(10)          | NO   |     | NULL    |                |
| password | varchar(20)          | NO   |     | NULL    |                |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |                |
| age      | tinyint(3) unsigned  | NO   |     | 10      |                |
+----------+----------------------+------+-----+---------+----------------+
5 rows in set (0.00 sec)
```

指定FIRST:

``` 
mysql> ALTER TABLE user1 ADD truename VARCHAR(20) NOT NULL FIRST;
Query OK, 0 rows affected (0.96 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW COLUMNS FROM user1;
+----------+----------------------+------+-----+---------+----------------+
| Field    | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| truename | varchar(20)          | NO   |     | NULL    |                |
| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
| username | varchar(10)          | NO   |     | NULL    |                |
| password | varchar(20)          | NO   |     | NULL    |                |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |                |
| age      | tinyint(3) unsigned  | NO   |     | 10      |                |
+----------+----------------------+------+-----+---------+----------------+
6 rows in set (0.00 sec)
```

添加多列：（不能指定位置关系）

```
ALTER TABLE table_name ADD [COLUMN] (col_name column_definition,...)
```

**删除列**

``` 
ALTER TABLE table_name DROP [COLUMN] (col_name)
```

如：

``` 
mysql> ALTER TABLE user1 DROp truename;
Query OK, 0 rows affected (0.69 sec)
Records: 0  Duplicates: 0  Warnings: 0
mysql> SHOW COLUMNS FROM user1;
+----------+----------------------+------+-----+---------+----------------+
| Field    | Type                 | Null | Key | Default | Extra          |
+----------+----------------------+------+-----+---------+----------------+
| id       | smallint(5) unsigned | NO   | PRI | NULL    | auto_increment |
| username | varchar(10)          | NO   |     | NULL    |                |
| password | varchar(20)          | NO   |     | NULL    |                |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |                |
| age      | tinyint(3) unsigned  | NO   |     | 10      |                |
+----------+----------------------+------+-----+---------+----------------+
```

**删除多列**

``` 
ALTER TABLE table_name DROP col_name1,DROP col_name2,...
```

**删除同时添加**

``` 
ALTER TABLE table_name DROP col_name1,ADD col_name2,...
```
#### 修改数据表——添加\删除约束****

**添加主键约束：**

```
ALTER TABLE table_name ADD[CONSTRAINT[symbol]] PRIMARY KEY [index_type] (index_col_name,...)
```

可选项：CONSTRAINT[symbol] 可以为主键设定名字，index_type指定索引类型

如：

```
mysql> CREATE TABLE user2(
    -> username VARCHAR(10) NOT NULL,
    -> pid SMALLINT UNSIGNED
    -> );
mysql> ALTER TABLE user2 ADD id SMALLINT UNSIGNED;
mysql> ALTER TABLE user2 ADD CONSTRAINT PK_user2_id PRIMARY KEY(id);
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   |     | NULL    |       |
| pid      | smallint(5) unsigned | YES  |     | NULL    |       |
| id       | smallint(5) unsigned | NO   | PRI | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

**添加唯一约束**

``` chainese
ALTER TABLE table_name ADD[CONSTRAINT[symbol]] UNIQUE [INDEX|KEY] [index_name][index_type] (index_col_name,...)
```

如：

```
mysql> ALTER TABLE user2 ADD UNIQUE (username);
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   | UNI | NULL    |       |
| pid      | smallint(5) unsigned | YES  |     | NULL    |       |
| id       | smallint(5) unsigned | NO   | PRI | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

**添加外键约束**

``` 
ALTER TABLE table_name ADD[CONSTRAINT[symbol]] FOREIGN KEY [index_name] (index_col_name,...) reference_definition
```

如：

```
mysql> ALTER TABLE user2 ADD FOREIGN KEY (pid) PEFERENCES provinces(id);
mysql> SHOW CREATE TABLE user2;
| user2 | CREATE TABLE `user2` (
  `username` varchar(10) NOT NULL,
  `pid` smallint(5) unsigned DEFAULT NULL,
  `id` smallint(5) unsigned NOT NULL,
  PRIMARY KEY (`id`),
  UNIQUE KEY `username` (`username`),
  KEY `pid` (`pid`),
  CONSTRAINT `user2_ibfk_1` FOREIGN KEY (`pid`) REFERENCES `provinces` (`id`)
) ENGINE=InnoDB DEFAULT CHARSET=latin1 |
```

**添加/删除默认约束**

``` 
ALTER TABLE table_name ALTER[COLUMN] col_name {SET DEFAULT literal|DROP DEFAULT}
```

如添加约束：

``` 
mysql> ALTER TABLE user2 ADD age TINYINT UNSIGNED NOT NULL;
mysql> ALTER TABLE user2 ALTER age SET DEFAULT 15;
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   | UNI | NULL    |       |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |       |
| id       | smallint(5) unsigned | NO   | PRI | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | 15      |       |
+----------+----------------------+------+-----+---------+-------+
```

删除约束：

``` 
mysql> ALTER TABLE user2 ALTER age DROP DEFAULT;
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   | UNI | NULL    |       |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |       |
| id       | smallint(5) unsigned | NO   | PRI | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

**删除主键约束**

```
ALTER TABLE table_name DROP PRIMARY KEY
```

如：

``` 
mysql> ALTER TABLE user2 DROP PRIMARY KEY;
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   | PRI | NULL    |       |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |       |
| id       | smallint(5) unsigned | NO   |     | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

**删除唯一约束**

```
ALTER TABLE table_name DROP {INDEX|KEY} index_name
```

指定索引名称是因为一张表可以有多个唯一约束，需要删除字段上的约束而非字段本身就要知道约束名称加以限定。

查看索引名称：

``` 
mysql> SHOW INDEXES FROM user2\G
```

得到 唯一约束column的Key_name: username

删除约束：

```
mysql> ALTER TABLE user2 DROP INDEX username;
```

查看数据表：

``` 
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| username | varchar(10)          | NO   |     | NULL    |       |
| pid      | smallint(5) unsigned | YES  | MUL | NULL    |       |
| id       | smallint(5) unsigned | NO   |     | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

唯一约束已删除

**删除外键约束**

``` 
mysql> ALTER TABLE user2 DROP FOREIGN KEY fk_symbol;
```

需查看外键的fk_symbol（系统指定）：

```
mysql> SHOW CREATE TABLE user2;
得到：
KEY `pid` (`pid`),
  CONSTRAINT `user2_ibfk_1` FOREIGN KEY (`pid`) REFERENCES `provinces` (`id`)
```

即：fk_symbol为‘user2\_ibfk_1’

删除约束：

``` 
mysql> ALTER TABLE user2 DROP FOREIGN KEY user2_ibfk_1;
```

还可以继续删除索引：

```
mysql> ALTER TABLE user2 DROP INDEX pid;
mysql> SHOW INDEXES FROM user2\G
Empty set (0.00 sec)
```

#### 修改列定义和更名数据表

**修改列定义**

```
ALTER TABLE table_name MODIFY col_name column_definition [FIRST|AFTER col_name]
```

备选项：可以修改字段位置，如：

```
mysql> ALTER TABLE user2 MODIFY id SMALLINT UNSIGNED NOT NULL FIRST;
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| id       | smallint(5) unsigned | NO   |     | NULL    |       |
| username | varchar(10)          | NO   |     | NULL    |       |
| pid      | smallint(5) unsigned | YES  |     | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

修改字段数据类型：

```
mysql> ALTER TABLE user2 MODIFY id TINYINT UNSIGNED NOT NULL;
mysql> SHOW COLUMNS FROM user2;
+----------+----------------------+------+-----+---------+-------+
| Field    | Type                 | Null | Key | Default | Extra |
+----------+----------------------+------+-----+---------+-------+
| id       | tinyint(3) unsigned  | NO   |     | NULL    |       |
| username | varchar(10)          | NO   |     | NULL    |       |
| pid      | smallint(5) unsigned | YES  |     | NULL    |       |
| age      | tinyint(3) unsigned  | NO   |     | NULL    |       |
+----------+----------------------+------+-----+---------+-------+
```

**但注意：将数据类型修改为更小类型时可能会造成数据丢失。**

**修改列名称：**

``` 
ALTER TABLE table_name CHANGE old_col_name new_col_name column_definition [FIRST|AFTER col_name]
```

即可修改列名称，也可修改列定义。

如：

```
mysql> ALTER TABLE user2 CHANGE pid p_id TINYINT UNSIGNED NOT NULL;
mysql> SHOW COLUMNS FROM user2;
+----------+---------------------+------+-----+---------+-------+
| Field    | Type                | Null | Key | Default | Extra |
+----------+---------------------+------+-----+---------+-------+
| id       | tinyint(3) unsigned | NO   |     | NULL    |       |
| username | varchar(10)         | NO   |     | NULL    |       |
| p_id     | tinyint(3) unsigned | NO   |     | NULL    |       |
| age      | tinyint(3) unsigned | NO   |     | NULL    |       |
+----------+---------------------+------+-----+---------+-------+
```

**数据表更名**

```
ALTER TABLE table_name RENAME[TO|AS] new_table_name
或：
RENAME TABLE table_name TO new_table_name
```

如：

```
mysql> ALTER TABLE user2 RENAME user3;
mysql> SHOW TABLES;
+-----------------+
| Tables_in_lcyDB |
+-----------------+
| provinces       |
| tb1             |
| tb2             |
| tb3             |
| tb4             |
| tb5             |
| tb6             |
| user1           |
| user3           |
| users           |
+-----------------+
mysql> ALTER TABLE user3 RENAME user2;
mysql> SHOW TABLES;
+-----------------+
| Tables_in_lcyDB |
+-----------------+
| provinces       |
| tb1             |
| tb2             |
| tb3             |
| tb4             |
| tb5             |
| tb6             |
| user1           |
| user2           |
| users           |
+-----------------+
```

**注意：**

**要尽量少使用列和表的更名，如果之前创建了索引或视图，引用了表名或列名，修改名称可能会导致视图或存储过程无法正常工作。**