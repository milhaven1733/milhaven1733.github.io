---
title: 《利用Python进行数据分析》笔记-数据规整化：清理、转换、合并、重塑
date: 2017-05-26 10:27:50
tags:
categories: 数据分析
---

### 合并数据集

#### SQL风格的merge

pandas的merge方法默认将重叠的列名作为键，也可以显式指定。

连接方式默认为内连接

（多对多连接产生笛卡尔积）

```
df1=DataFrame({'lkey':['b','b','a','c','a','b'],'data1':range(6)})
df2=DataFrame({'rkey':['a','b','a','b','d'],'data2':range(5)})
pd.merge(df1,df2,left_on='lkey',right_on='rkey')
```

|      | data1 | lkey | data2 | rkey |
| ---- | ----- | ---- | ----- | ---- |
| 0    | 0     | b    | 1     | b    |
| 1    | 0     | b    | 3     | b    |
| 2    | 1     | b    | 1     | b    |
| 3    | 1     | b    | 3     | b    |
| 4    | 5     | b    | 1     | b    |
| 5    | 5     | b    | 3     | b    |
| 6    | 2     | a    | 0     | a    |
| 7    | 2     | a    | 2     | a    |
| 8    | 4     | a    | 0     | a    |
| 9    | 4     | a    | 2     | a    |

要根据多个键进行合并，可传入由列名组成的list

```
left=DataFrame({'key1':['foo','foo','bar'],'key2':['one','two','one'],'lval':[1,2,3]})
right=DataFrame({'key1':['foo','foo','bar','bar'],'key2':['one','one','one','two'],'lval':[4,5,6,7]})
pd.merge(left,right,on=['key1','key2'])
```

或：

```
left=DataFrame({'lkey1':['foo','foo','bar'],'lkey2':['one','two','one'],'lval':[1,2,3]})
right=DataFrame({'rkey1':['foo','foo','bar','bar'],'rkey2':['one','one','one','two'],'lval':[4,5,6,7]})
pd.merge(left,right,left_on=['lkey1','lkey2'],right_on=['rkey1','rkey2'])
```

处理重复列名，可以使用suffixes选项指定附加在重叠列名上的字符后缀：

```
pd.merge(left,right,on=['key1'],suffixes=('_x','_y'))
```

|      | key1 | key2_x | lval | key2_y | rval |
| ---- | ---- | ------ | ---- | ------ | ---- |
| 0    | foo  | one    | 1    | one    | 4    |
| 1    | foo  | one    | 1    | one    | 5    |
| 2    | foo  | two    | 2    | one    | 4    |
| 3    | foo  | two    | 2    | one    | 5    |
| 4    | bar  | one    | 3    | one    | 6    |
| 5    | bar  | one    | 3    | two    | 7    |

#### 按索引合并

连接时，索引可以作为连接键——指定`left_index=Ture`或`right_index=True`

对于层次化索引，必须以列表形式说明作为合并键的多个列

```
left=DataFrame({'key1':['Ohio','Ohio','Ohio','Nevada','Nevada'],'key2':[2000,2001,2002,2001,2002],'data':range(5)})
right=DataFrame(np.arange(12).reshape((6,2)),index=[['Nevada','Nevada','Ohio','Ohio','Ohio','Ohio'],[2001,2000,2000,2000,2001,2002]],columns=['envent1','envent2'])
pd.merge(left,right,left_on=['key1','key2'],right_index=True,how='left') #左外连接
```

|      | data | key1   | key2 | envent1 | envent2 |
| ---- | ---- | ------ | ---- | ------- | ------- |
| 0    | 0    | Ohio   | 2000 | 4.0     | 5.0     |
| 0    | 0    | Ohio   | 2000 | 6.0     | 7.0     |
| 1    | 1    | Ohio   | 2001 | 8.0     | 9.0     |
| 2    | 2    | Ohio   | 2002 | 10.0    | 11.0    |
| 3    | 3    | Nevada | 2001 | 0.0     | 1.0     |
| 4    | 4    | Nevada | 2002 | NaN     | NaN     |

join方法可以更方便的实现索引合并

```
left.join(right,how='……')
```

还可以支持调用DF的某列与参数DF的索引之间的连接

```
left.join(right,on='left的列名')
```

轴向连接

concat方法可以指定连接的轴向：（默认axis=0）

```
In [32]:
s1=pd.Series([0,1],index=['a','b'])
s2=pd.Series([2,3,4],index=['c','d','e'])
s3=pd.Series([5,6],index=['f','g'])
s4=pd.concat([s1*5,s3])
s4
Out[32]:
a    0
b    5
f    5
g    6
dtype: int64

In [37]:
pd.concat([s1,s4],axis=0)
Out[37]:
a    0
b    1
a    0
b    5
f    5
g    6
dtype: int64
In [36]:
inner
pd.concat([s1,s4],axis=1,join='inner')
Out[36]:
	0	1
a	0	0
b	1	5
```

