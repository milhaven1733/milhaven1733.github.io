---
title: Series&DateFrame
date: 2017-04-27 09:12:55
tags:
categories: 数据分析
---

### Series

- 定长有序字典
- 包含数据和索引

<!---more--->

查看index和value：

```
bSer=pd.Series(['apple','banana','peach'],index=[1,2,3])
bSer.index
bSer.values
```

```
1     apple
2    banana
3     peach
dtype: object
Int64Index([1, 2, 3], dtype='int64')
array(['apple', 'banana', 'peach'], dtype=object)
```

对元素应用一些基本运算和函数：

```
aSer=pd.Series([1,2.0,'a'])
aSer*2
Out:
0     2
1     4
2    aa
dtype: object
```

```
aSer=pd.Series([3,5,7])
np.exp(aSer) #计算自然对数的n次方
Out:
0      20.085537
1     148.413159
2    1096.633158
dtype: float64
```

数据对齐：

提供一些待查找的索引，在另一张存在索引和数值的表中，将要查找的索引和表中存在的索引对齐：

```
data={'AXP':'86.40','CSCO':'122.64','BA':'99.44'}
sindex=['AXP','CSCO','BA','AAPL']
aSer=pd.Series(data,index=sindex)
aSer
```

```
AXP      86.40
CSCO    122.64
BA       99.44
AAPL       NaN
dtype: object
```

> NaN (Not a Number)表示未定义或不可表示的值，这里可以理解为缺失值或控制。

检测空值：

```
pd.isnull(aSer)
Out:
AXP     False
CSCO    False
BA      False
AAPL     True
dtype: bool
```

name属性：

Series对象本身及其索引均有一个name属性，可以进行指定：

```
aSer.name='cname'
aSer.index.name='volumn'
aSer
Out:
volumn
AXP      86.40
CSCO    122.64
BA       99.44
AAPL       NaN
Name: cname, dtype: object
```

### DataFrame

- 可以看做共享一个index的Series的集合
- 可以指定index
- 可以进行数据对齐

取Dataframe对象的行和列：

```
data={'name':['XueXue','TuTu','JingJing','GaiGai'],'Age':[22,21,20,19]}
frame=pd.DataFrame(data,index=[1,2,3,4])
frame.name
Out:
0      XueXue
1        TuTu
2    JingJing
3      GaiGai
Name: name, dtype: object
frame.ix[2]  #根据index名称取值
Age           20
name    JingJing
Name: 2, dtype: object
```

删除一列：

```
del frame['Age']
frame
Out:
name
1	XueXue
2	TuTu
3	JingJing
4	GaiGai
```

指定name属性：

```
frame.index.name='No'
frame
```

|      | Age  | name     |
| ---- | ---- | -------- |
| No   |      |          |
| 1    | 22   | XueXue   |
| 2    | 21   | TuTu     |
| 3    | 20   | JingJing |
| 4    | 19   | GaiGai   |