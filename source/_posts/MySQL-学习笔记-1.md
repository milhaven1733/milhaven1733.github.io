---
title: MySQL-学习笔记-1
date: 2017-04-11 00:17:20
tags:
categories: SQL
---

### MySQL自启动

**sudo chkconfig --add mysql        添加服务**

**netstat -na | grep 3306  看到有监听说明服务启动**

或：**ps -ef | grep mysqld 检查MySQL服务器是否启动**

另：

**sudo /etc/init.d/mysql start** 	启动MySQL 服务器

**sudo /etc/init.d/mysql stop** 或

**./mysqladmin -u root -p shutdown**  关闭目前运行的 MySQL 服务器（/usr/bin 目录下）

<!--more-->

### MySQL添加用户、删除用户与授权

**登录MYSQL：**

\>mysql -u root -p

**创建用户：**

\>CREATE USER 'username'@'localhost' IDENTIFIED BY 'password';

或 

> INSERT INTO user (host, user,ssl_cipher,x509_issuer,x509_subject)  VALUES ('localhost', 'test','','','');
>
> use mysql;
>
>  UPDATE user SET authentication_string=PASSWORD('***') WHERE user='test';
>
> FLUSH PRIVILEGES;



**为用户授权**:

**以ROOT身份登录**

\>mysql -u root -p

**为用户创建一个数据库:**

\>create database ***DB;

**授权新用户拥有\*\*\*DB数据库的所有权限:**

\>grant all privileges on \*\*\*DB.* to \*\*\*@localhost identified by '***(password)';

**刷新系统权限表:**

\>flush privileges;

**授权新用户拥有所有数据库的某些权限： **

\>grant select,delete,update,create,drop on *.* to \*\*\*@localhost identified by "***";

**查看MYSQL数据库中所有用户**:

\> SELECT DISTINCT CONCAT('User: ''',user,'''@''',host,''';') AS query FROM mysql.user;

**查看数据库中具体某个用户的权限:**

\>show grants for '***'@'localhost';  

### 修改MySQL提示符：

**登录时：**

mysql -u -p --prompt 提示符

**已连接客户端时:**

\>prompt 提示符;

部分参数：

![](MySQL-学习笔记-1\1.png)

### 常用命令及语法规范

![](MySQL-学习笔记-1\2.png)

![](MySQL-学习笔记-1\3.png)

### 操作数据库

**创建数据库：**

\>CREATE DATABASE db_name;

全部参数：

![](MySQL-学习笔记-1\4.png)

存在warning信息时查看：

\>SHOW WARNINGS;

查看创建时编码方式：

\>SHOW CREATE DATABASE db_name;

**查看（当前服务器下）数据库列表：**

\>SHOW DATABASES;

**修改数据库：**

![](MySQL-学习笔记-1\5.png)

**删除数据库：**

\>DROP DATABASE db_name;