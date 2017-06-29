---
title: MySQL-学习笔记-5
date: 2017-04-21 10:47:43
tags:
categories: SQL
---

### 子查询与连接

#### 数据的准备

创建商城信息数据库：

```
mysql> CREATE TABLE IF NOT EXISTS tdb_goods(
    ->     goods_id    SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    ->     goods_name  VARCHAR(150) NOT NULL,
    ->     goods_cate  VARCHAR(40)  NOT NULL,
    ->     brand_name  VARCHAR(40)  NOT NULL,
    ->     goods_price DECIMAL(15,3) UNSIGNED NOT NULL DEFAULT 0,
    ->     is_show     BOOLEAN NOT NULL DEFAULT 1,
    ->     is_saleoff  BOOLEAN NOT NULL DEFAULT 0
    ->   );
```

字段说明：

goods_id 商品id 主键	goods_cate 商品分类

is_show  是否上架 布尔型	is_saleoff 是否售空 布尔型

<!---more--->

录入数据：

由于名称、品牌等含有中文字符，需要更改表的编码格式：

```
mysql> ALTER TABLE tdb_goods CONVERT TO CHARACTER SET utf8;
```

录入20余条商品记录，如：

```
INSERT tdb_goods (goods_name,goods_cate,brand_name,goods_price,is_show,is_saleoff) VALUES('商务双肩背包','笔记本配件','索尼','99',DEFAULT,DEFAULT);
```

以网格形式显示表内容：

```
mysql> SELECT * FROM tdb_goods\G
*************************** 1. row ***************************
   goods_id: 1
 goods_name: R510VC 15.6英寸笔记本
 goods_cate: 笔记本
 brand_name: 华硕
goods_price: 3399.000
    is_show: 1
 is_saleoff: 0
*************************** 2. row ***************************
   goods_id: 2
 goods_name: Y400N 14.0英寸笔记本电脑
 goods_cate: 笔记本
 brand_name: 联想
goods_price: 4899.000
    is_show: 1
 is_saleoff: 0
```

#### 子查询

子查询是指出现在其他SQL语句内的SELECE子句

如：

```
SELECT * FROM t1 WHERE col1=(SELECT col2 FROM t2);
```

其中SELECE * FROM t1称为Outer Quert(外层查询)

SELECT col2 FROM t2，称为SubQuery(子查询)

注意几点：

- 子查询嵌套在查询内部,且必须始终出现在圆括号内。

- 子查询可以包含多个关键字或条件，如DISTINCT、GROUP BY、LIMIT、函数等。

- 子查询的外层查询可以是：SELECT,INSERT,UPDATE,SET,DO等

  >强调：这里的外层查询不是指狭义的“查找”，而是指所有SQL 命令的统称，SQL——结构化查询语言。

子查询返回值：

子查询可以返回标量、一行、一列或子查询。

#### 由比较运算符引发的子查询（第一类）

使用比较运算符的子查询：

=、>、<、>=、<=、<>、!=、<=>

语法结构

operand comparison_operator subquery

示例一：

查询商品平均价格：（使用聚合函数AVG（））

```
mysql> SELECT AVG(goods_price) FROM tdb_goods;
+------------------+
| AVG(goods_price) |
+------------------+
|     5636.3636364 |
+------------------+
```

四舍五入到百分位：（ROUND函数）

```
mysql> SELECT ROUND(AVG(goods_price),2) FROM tdb_goods;
+---------------------------+
| ROUND(AVG(goods_price),2) |
+---------------------------+
|                   5636.36 |
+---------------------------+
```

查找表中价格大于平均价格的商品信息：

```
mysql> SELECT goods_id,goods_name,goods_price FROM tdb_goods WHERE goods_price>5636.36;
+----------+-----------------------------------------+-------------+
| goods_id | goods_name                              | goods_price |
+----------+-----------------------------------------+-------------+
|        3 | G150TH 15.6英寸游戏本                   |    8499.000 |
|        7 | SVP13226SCB 13.3英寸触控超极本          |    7999.000 |
|       13 | iMac ME086CH/A 21.5英寸一体电脑         |    9188.000 |
|       17 | Mac Pro MD878CH/A 专业级台式电脑        |   28888.000 |
|       18 |  HMZ-T3W 头戴显示设备                   |    6999.000 |
|       20 | X3250 M4机架式服务器 2583i14            |    6888.000 |
|       21 |  HMZ-T3W 头戴显示设备                   |    6999.000 |
+----------+-----------------------------------------+-------------+
```

使用子查询得到相同结果：

```
mysql> SELECT goods_id,goods_name,goods_price FROM tdb_goods WHERE goods_price>(SELECT ROUND(AVG(goods_price),2) FROM tdb_goods);
+----------+-----------------------------------------+-------------+
| goods_id | goods_name                              | goods_price |
+----------+-----------------------------------------+-------------+
|        3 | G150TH 15.6英寸游戏本                   |    8499.000 |
|        7 | SVP13226SCB 13.3英寸触控超极本          |    7999.000 |
|       13 | iMac ME086CH/A 21.5英寸一体电脑         |    9188.000 |
|       17 | Mac Pro MD878CH/A 专业级台式电脑        |   28888.000 |
|       18 |  HMZ-T3W 头戴显示设备                   |    6999.000 |
|       20 | X3250 M4机架式服务器 2583i14            |    6888.000 |
|       21 |  HMZ-T3W 头戴显示设备                   |    6999.000 |
+----------+-----------------------------------------+-------------+
```

示例二：

查找商品中类型为“超级本”的商品价格：

```
mysql> SELECT goods_price FROM tdb_goods WHERE goods_cate="超级本";
+-------------+
| goods_price |
+-------------+
|    4999.000 |
|    4299.000 |
|    7999.000 |
+-------------+
```

在表中查找商品价格大于等于超级本价格的商品：

```
mysql> SELECT goods_id,goods_name,goods_price FROM tdb_goods WHERE goods_price>=(SELECT goods_price FROM tdb_goods WHERE goods_cate="超级本");
ERROR 1242 (21000): Subquery returns more than 1 row
```

报错:子查询返回结果多于一行，即没有指定对于一个还是多个结果进行比较。

用ANY、SOME、ALL、修饰比较运算符：

ANY和SOME等价，表示符合一个即可，ALL表示需要符合所有。返回值原则如下：

|       | ANY  | SOME | ALL  |
| ----- | ---- | ---- | ---- |
| \>、>= | 最小值  | 最小值  | 最大值  |
| <、<=  | 最大值  | 最大值  | 最小值  |
| =     | 任意值  | 任意值  |      |
| <>、!= |      |      | 任意值  |

如查询大于等于最大值的：

```
mysql> SELECT goods_id,goods_name,goods_price FROM tdb_goods WHERE goods_price>=ALL(SELECT goods_price FROM tdb_goods WHERE goods_cate="超级本");
+----------+-----------------------------------------+-------------+
| goods_id | goods_name                              | goods_price |
+----------+-----------------------------------------+-------------+
|        3 | G150TH 15.6英寸游戏本                   |    8499.000 |
|        7 | SVP13226SCB 13.3英寸触控超极本          |    7999.000 |
|       13 | iMac ME086CH/A 21.5英寸一体电脑         |    9188.000 |
|       17 | Mac Pro MD878CH/A 专业级台式电脑        |   28888.000 |
+----------+-----------------------------------------+-------------+
```

#### 由[NOT] IN/EXISTS 引发的子查询（第二、三类）

语法结构

operand comparison_operator [NOT] IN subquery

=ANY 运算符与IN等效。

!=ALL或<>ALL运算符与NOT IN等效。

如：

```
mysql> SELECT goods_id,goods_name,goods_price FROM tdb_goods WHERE goods_price IN  (SELECT goods_price FROM tdb_goods WHERE goods_cate="超级本");
+----------+---------------------------------------+-------------+
| goods_id | goods_name                            | goods_price |
+----------+---------------------------------------+-------------+
|        5 | X240(20ALA0EYCD) 12.5英寸超极本       |    4999.000 |
|        6 | U330P 13.3英寸超极本                  |    4299.000 |
|        7 | SVP13226SCB 13.3英寸触控超极本        |    7999.000 |
+----------+---------------------------------------+-------------+
```

使用[NOT] EXISTS 的子查询

如果子查询返回任何行，EXISTS将返回TRUE；否则为FALSE。（不常用）

#### 使用INSERT...SELECT插入记录

将查询结果写入数据表：

```
INSERT [INTO] table_name [(col_name,...)] SELECT...
```

之前使用的表中goods_cate和brand_name字段重复信息很多，如果商品信息增多，会造成数据表体积过于庞大，故该字段适合用外键存储，需要另建商品类型表及品牌表，以创建商品类型表为例：

```
mysql> CREATE TABLE tdb_goods_cates(
    -> cate_id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> cate_name VARCHAR(40) NOT NULL
    -> );
```

把分类信息插入新建表：

对商品信息进行分组：

```
mysql> SELECT goods_cate FROM tdb_goods GROUP BY goods_cate;
+---------------------+
| goods_cate          |
+---------------------+
| 台式机              |
| 平板电脑            |
| 服务器/工作站       |
| 游戏本              |
| 笔记本              |
| 笔记本配件          |
| 超级本              |
+---------------------+
```

将查询结果写入数据表：（含有中文字符需修改字符编码）

```
mysql> INSERT tdb_goods_cates(cate_name) SELECT goods_cate FROM tdb_goods GROUP BY goods_cate;
```

查看结果：

```
mysql> SELECT * FROM tdb_goods_cates;
+---------+---------------------+
| cate_id | cate_name           |
+---------+---------------------+
|       1 | 台式机              |
|       2 | 平板电脑            |
|       3 | 服务器/工作站       |
|       4 | 游戏本              |
|       5 | 笔记本              |
|       6 | 笔记本配件          |
|       7 | 超级本              |
+---------+---------------------+
```

接下来，如果想使用外键表示商品分类，需要参照分类表去更新商品表，涉及下一节——多表更新。

#### 多表更新

```
UPDATE table_references SET col_name1={expr1|DEFAULT}[,col_name2={expr2|DEFAULT}]... [WHERE where_condition]
```

多表更新重点：表的参照关系，即：table_references

语法结构：

```
table_references
{[INNER|CROSS]JOIN|{LEFT|RIGHT}[OUTER]JOIN}
table_references
ON conditional_expr
```

连接类型：

```
inner join,内连接

在mysql，join,cross join和inner join 是等价的

left [outer] join,左外连接

right [outer] join,右外连接
```

使用内连接更新商品表：

```
mysql> UPDATE tdb_goods INNER JOIN tdb_goods_cates ON goods_cate=cate_name SET goods_cate=cate_id;
```

查看结果：

```
mysql> SELECT * FROM tdb_goods\G
*************************** 1. row ***************************
   goods_id: 1
 goods_name: R510VC 15.6英寸笔记本
 goods_cate: 5
 brand_name: 华硕
goods_price: 3399.000
    is_show: 1
 is_saleoff: 0
*************************** 2. row ***************************
   goods_id: 2
 goods_name: Y400N 14.0英寸笔记本电脑
 goods_cate: 5
 brand_name: 联想
goods_price: 4899.000
    is_show: 1
 is_saleoff: 0
*************************** 3. row ***************************
   goods_id: 3
 goods_name: G150TH 15.6英寸游戏本
 goods_cate: 4
 brand_name: 雷神
goods_price: 8499.000
    is_show: 1
 is_saleoff: 0
```

####  一步到位的多表更新

首先，我们可以在创建表的同时将查询结果写入到数据表（使用CREATE...SELECT语句）

如：

```
mysql> ALTER DATABASE lcyDB CHARACTER SET = utf8; //先改变数据库编码方式，否则不能在创建同时写入
mysql> CREATE TABLE tdb_goods_brands(
    -> brand_id SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    -> brand_name VARCHAR(40) NOT NULL)
    -> SELECT brand_name  FROM tdb_goods GROUP BY brand_name;
```

查看结果：

```
mysql> SELECT * FROM tdb_goods_brands;
+----------+------------+
| brand_id | brand_name |
+----------+------------+
|        1 | IBM        |
|        2 | 华硕       |
|        3 | 宏碁       |
|        4 | 惠普       |
|        5 | 戴尔       |
|        6 | 索尼       |
|        7 | 联想       |
|        8 | 苹果       |
|        9 | 雷神       |
+----------+------------+
```

更新商品表品牌信息：

先按照上面的方法更新，报错：

```
mysql> UPDATE tdb_goods INNER JOIN tdb_goods_brands ON brand_name=brand_name SET brand_name=brand_id;
ERROR 1052 (23000): Column 'brand_name' in field list is ambiguous
```

原因是，两表中都含有字段brand_name，无法区分字段是属于哪个表（含义不明），这时候需要在字段前加表名或者为表起别名：

```
mysql> UPDATE tdb_goods AS g INNER JOIN tdb_goods_brands AS b ON g.brand_name=b.brand_name SET g.brand_name=b.brand_id;
```

更新完成，查看：

```
mysql> SELECT * FROM tdb_goods\G
*************************** 1. row ***************************
   goods_id: 1
 goods_name: R510VC 15.6英寸笔记本
 goods_cate: 5
 brand_name: 2
goods_price: 3399.000
    is_show: 1
 is_saleoff: 0
```

但查看表的结构发现虽然更新了字段内容，字段的数据类型依然为“varchar(40)”，这时候，最好能修改字段的类型。

同时修改列名称和列定义需使用ALTER...CHANGE语句：

```
mysql> ALTER TABLE tdb_goods
    -> CHANGE goods_cate cate_id SMALLINT UNSIGNED NOT NULL,
    -> CHANGE brand_name brand_id SMALLINT UNSIGNED NOT NULL;
```

再来查看字段类型：

```
mysql> SHOW COLUMNS FROM tdb_goods;
+-------------+------------------------+------+-----+---------+----------------+
| Field       | Type                   | Null | Key | Default | Extra          |
+-------------+------------------------+------+-----+---------+----------------+
| goods_id    | smallint(5) unsigned   | NO   | PRI | NULL    | auto_increment |
| goods_name  | varchar(150)           | NO   |     | NULL    |                |
| cate_id     | smallint(5) unsigned   | NO   |     | NULL    |                |
| brand_id    | smallint(5) unsigned   | NO   |     | NULL    |                |
| goods_price | decimal(15,3) unsigned | NO   |     | 0.000   |                |
| is_show     | tinyint(1)             | NO   |     | 1       |                |
| is_saleoff  | tinyint(1)             | NO   |     | 0       |                |
+-------------+------------------------+------+-----+---------+----------------+
```

此时，上面修改的两个字段已经成为了一种“事实上的外键”

#### 内连接

使用ON关键字来设定连接条件，也可以使用WHERE来代替

但通常使用ON关键字来设定连接条件，而使用WHERE关键字进行结果集记录的过滤

内连接：

只显示坐标和右表符合连接条件的记录

我们先在商品表中插入一条类型表中不含有其cate_id的记录：

```
mysql> INSERT tdb_goods(goods_name,cate_id,brand_id,goods_price) VALUES(' LaserJet Pro P1606dn 黑白激光打印机','12','4','1849');
```

使用内连接查找两张表的字段：

```
mysql> SELECT goods_id,goods_name cate_name FROM tdb_goods INNER JOIN tdb_goods_cates ON tdb_goods.cate_id=tdb_goods_cates.cate_id\G
```

找到22条商品记录

```
*************************** 22. row ***************************
 goods_id: 22
cate_name: 商务双肩背包
22 rows in set (0.00 sec)
```

因为新增加的一条记录不符合连接条件（商品分类表不存在其cate_id）

#### 外连接

左外连接 LEFT JOIN：

显示左表全部和右表符合连接条件的记录

右外连接 RIGHT JOIN：

显示右表全部和左表符合连接条件的记录

左外连接示例：

```
mysql> SELECT goods_id,goods_name,cate_name FROM tdb_goods LEFT JOIN tdb_goods_cates ON tdb_goods.cate_id=tdb_goods_cates.cate_id\G
```

查找到23条记录：

```
*************************** 23. row ***************************
  goods_id: 23
goods_name:  LaserJet Pro P1606dn 黑白激光打印机
 cate_name: NULL
23 rows in set (0.00 sec)
```

可以看到，当左表中的记录，在右表中不存在含有符合连接条件的字段，右表中字段显示为NULL

右外连接示例：

先在类型表中插入几条商品表中不含有其cate_id的记录：

```
mysql> INSERT tdb_goods_cates(cate_name) VALUES('路由器'),('交换机'),('网卡');
```

进行右外连接查询：

```
mysql> SELECT goods_id,goods_name,cate_name FROM tdb_goods RIGHT JOIN tdb_goods_cates ON tdb_goods.cate_id=tdb_goods_cates.cate_id\G
```

这次查询到了25条记录：

```
*************************** 23. row ***************************
  goods_id: NULL
goods_name: NULL
 cate_name: 路由器
*************************** 24. row ***************************
  goods_id: NULL
goods_name: NULL
 cate_name: 交换机
*************************** 25. row ***************************
  goods_id: NULL
goods_name: NULL
 cate_name: 网卡
25 rows in set (0.00 sec)
```

同样，当右表中的记录在左表中没有相应记录能满足连接条件，查询结果中，来自左表的字段被设置为NULL

#### 多表连接

连接三张表查询商品的完整信息：（要为表起别名方便操作）

```
mysql> SELECT goods_id,goods_name,cate_name,brand_name,goods_price FROM tdb_goods AS g
    -> INNER JOIN tdb_goods_cates AS c ON g.cate_id=c.cate_id
    -> INNER JOIN tdb_goods_brands AS b ON g.brand_id=b.brand_id\G
```

查询结果示例：

```
*************************** 1. row ***************************
   goods_id: 1
 goods_name: R510VC 15.6英寸笔记本
  cate_name: 笔记本
 brand_name: 华硕
goods_price: 3399.000
```

多表连接操作相当于外键的逆向操作。

#### 多表删除

在当前数据表中，id为18,19的记录和id为21,22的记录是完全相同的，怎样删除数据表中的重复记录？

可以用一张表模拟多表删除的方式来实现。

首先——查找重复记录

```
mysql> SELECT MIN(goods_id),goods_name FROM tdb_goods GROUP BY goods_name\G
```

以goods_name对记录进行分组（mysql 5.7以后必须用MIN(goods_id)指定显示分组数据时，每组若有多个goods_id，应具体显示哪个）

发现查找到21条记录，即有21个不同goods_name的商品.

但是这样操作只显示了不重复的数据有多少，我们还需要查找出是那些记录出现了重复，可以在查询结果中用HAVING语句加以筛选：

```
mysql> SELECT MIN(goods_id),goods_name FROM tdb_goods GROUP BY goods_name HAVING count(goods_name)>=2;
```

结果显示：

```
+---------------+-----------------------------+
| MIN(goods_id) | goods_name                  |
+---------------+-----------------------------+
|            18 |  HMZ-T3W 头戴显示设备       |
|            19 | 商务双肩背包                |
+---------------+-----------------------------+
```

查找到了重复的记录。

接下来，我们可以将查询结果看成一张新表，参照这张表来删除原有表中的记录：

```
mysql> DELETE t1 FROM tdb_goods AS t1 LEFT JOIN (SELECT MIN(goods_id) AS goods_id,goods_name FROM tdb_goods GROUP BY goods_name HAVING count(goods_name)>=2) AS t2 ON t1.goods_name=t2.goods_name WHERE t1.goods_id>t2.goods_id;
```

语句比较复杂。语句中执行了如下操作：

- 将表tdb_goods设置别名t1
- 以goods_name分类，查找出重复商品记录的goods_id和goods_name，将查询结果生成一张新表，起别名t2
- 左外连接两张表，连接条件为t1.goods_name=t2.goods_name
- 删除t1表中的重复记录，删除的条件是：t1.goods_id>t2.goods_id;

个人理解：删除是在连接两张表得到的结果中查找满足WHERE后语句的记录，并将这些记录从t1中删除。

 试验证明，连接时使用INNER  JOIN也可以，连接结果是重复的四条记录。

#### 无限极分类数据表（重点内容）

如果商品分类下还有小分类，小分类下继续包含分类，这样无限极分类的数据表如何设计？

通过表自身的连接来实现。

首先创建一张无限分类数据表

```
mysql> CREATE TABLE tdb_goods_types(
    ->type_id   SMALLINT UNSIGNED PRIMARY KEY AUTO_INCREMENT,
    ->type_name VARCHAR(20) NOT NULL,
    ->parent_id SMALLINT UNSIGNED NOT NULL DEFAULT 0
    ->);
```

parent_id字段 表示其父类的id

插入若干记录：

```
INSERT tdb_goods_types(type_name,parent_id) VALUES('家用电器',DEFAULT);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('电脑、办公',DEFAULT);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('大家电',1);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('生活电器',1);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('平板电视',3);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('空调',3);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('电风扇',4);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('饮水机',4);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('电脑整机',2);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('电脑配件',2);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('笔记本',9);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('超级本',9);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('游戏本',9);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('CPU',10);
  INSERT tdb_goods_types(type_name,parent_id) VALUES('主机',10);
```

查看记录:

```
mysql> SELECT * FROM tdb_goods_types;
+---------+-----------------+-----------+
| type_id | type_name       | parent_id |
+---------+-----------------+-----------+
|       1 | 家用电器        |         0 |
|       2 | 电脑、办公      |         0 |
|       3 | 大家电          |         1 |
|       4 | 生活电器        |         1 |
|       5 | 平板电视        |         3 |
|       6 | 空调            |         3 |
|       7 | 电风扇          |         4 |
|       8 | 饮水机          |         4 |
|       9 | 电脑整机        |         2 |
|      10 | 电脑配件        |         2 |
|      11 | 笔记本          |         9 |
|      12 | 超级本          |         9 |
|      13 | 游戏本          |         9 |
|      14 | CPU             |        10 |
|      15 | 主机            |        10 |
+---------+-----------------+-----------+
```

"大家电"父类——“ 家用电器 ”

"空调"父类——“ 大家电 ”

以此类推。

**自身连接查找：**

可以想象在当前表右侧出现一张相同的表。我们将左边的表当做子表，右边的表当做父表进行连接：

```
mysql> SELECT s.type_id,s.type_name,p.type_name FROM tdb_goods_types AS s LEFT JOIN tdb_goods_types AS p ON s.parent_id=p.type_id;
```

连接结果：

```
+---------+-----------------+-----------------+
| type_id | type_name       | type_name       |
+---------+-----------------+-----------------+
|       1 | 家用电器        | NULL            |
|       2 | 电脑、办公      | NULL            |
|       3 | 大家电          | 家用电器        |
|       4 | 生活电器        | 家用电器        |
|       5 | 平板电视        | 大家电          |
|       6 | 空调            | 大家电          |
|       7 | 电风扇          | 生活电器        |
|       8 | 饮水机          | 生活电器        |
|       9 | 电脑整机        | 电脑、办公      |
|      10 | 电脑配件        | 电脑、办公      |
|      11 | 笔记本          | 电脑整机        |
|      12 | 超级本          | 电脑整机        |
|      13 | 游戏本          | 电脑整机        |
|      14 | CPU             | 电脑配件        |
|      15 | 主机            | 电脑配件        |
+---------+-----------------+-----------------+
```

查找出了子类对应的父类名称。

接下来查找父类中包含的子类：

```
mysql> SELECT p.type_id,p.type_name,s.type_name FROM tdb_goods_types AS p LEFT JOIN tdb_goods_types AS s ON s.parent_id=p.type_id;
```

结果：

```
+---------+-----------------+--------------+
| type_id | type_name       | type_name    |
+---------+-----------------+--------------+
|       1 | 家用电器        | 大家电       |
|       1 | 家用电器        | 生活电器     |
|       3 | 大家电          | 平板电视     |
|       3 | 大家电          | 空调         |
|       4 | 生活电器        | 电风扇       |
|       4 | 生活电器        | 饮水机       |
|       2 | 电脑、办公      | 电脑整机     |
|       2 | 电脑、办公      | 电脑配件     |
|       9 | 电脑整机        | 笔记本       |
|       9 | 电脑整机        | 超级本       |
|       9 | 电脑整机        | 游戏本       |
|      10 | 电脑配件        | CPU          |
|      10 | 电脑配件        | 主机         |
|       5 | 平板电视        | NULL         |
|       6 | 空调            | NULL         |
|       7 | 电风扇          | NULL         |
|       8 | 饮水机          | NULL         |
|      11 | 笔记本          | NULL         |
|      12 | 超级本          | NULL         |
|      13 | 游戏本          | NULL         |
|      14 | CPU             | NULL         |
|      15 | 主机            | NULL         |
+---------+-----------------+--------------+
```

对结果以父类名称分类呈现，并显示子类的数目(别名为child_count)：

```
mysql> SELECT MIN(p.type_id),p.type_name,count(s.type_name) AS child_count FROM tdb_goods_types AS p LEFT JOIN tdb_goods_types AS s ON s.parent_id=p.type_id GROUP BY p.type_name;
```

结果为：

```
+----------------+-----------------+-------------+
| MIN(p.type_id) | type_name       | child_count |
+----------------+-----------------+-------------+
|             14 | CPU             |           0 |
|             15 | 主机            |           0 |
|              3 | 大家电          |           2 |
|              1 | 家用电器        |           2 |
|              5 | 平板电视        |           0 |
|             13 | 游戏本          |           0 |
|              4 | 生活电器        |           2 |
|              2 | 电脑、办公      |           2 |
|              9 | 电脑整机        |           3 |
|             10 | 电脑配件        |           2 |
|              7 | 电风扇          |           0 |
|              6 | 空调            |           0 |
|             11 | 笔记本          |           0 |
|             12 | 超级本          |           0 |
|              8 | 饮水机          |           0 |
+----------------+-----------------+-------------+
```

为最左一列字段起别名并以此为标准排序：

```
mysql> SELECT MIN(p.type_id) AS type_id,p.type_name,count(s.type_name) AS child_count FROM tdb_goods_types AS p LEFT JOIN tdb_goods_types AS s ON s.parent_id=p.type_id GROUP BY p.type_name ORDER BY child_count DESC,type_id;
```

结果：

```
+---------+-----------------+-------------+
| type_id | type_name       | child_count |
+---------+-----------------+-------------+
|       9 | 电脑整机        |           3 |
|       1 | 家用电器        |           2 |
|       2 | 电脑、办公      |           2 |
|       3 | 大家电          |           2 |
|       4 | 生活电器        |           2 |
|      10 | 电脑配件        |           2 |
|       5 | 平板电视        |           0 |
|       6 | 空调            |           0 |
|       7 | 电风扇          |           0 |
|       8 | 饮水机          |           0 |
|      11 | 笔记本          |           0 |
|      12 | 超级本          |           0 |
|      13 | 游戏本          |           0 |
|      14 | CPU             |           0 |
|      15 | 主机            |           0 |
+---------+-----------------+-------------+

```

