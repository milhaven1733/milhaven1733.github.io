---
title: 《利用Python进行数据分析》笔记——pandas入门
date: 2017-05-11
tags:
categories: 数据分析
---

### 数据结构相关

#### Series

pandas的isnull、notnull函数可用于检测缺失数据：

```
aSer
Out[3]:
AXP      86.40
CSCO    122.64
BA       99.44
AAPL       NaN
dtype: object
In [4]:
pd.isnull(aSer)
Out[4]:
AXP     False
CSCO    False
BA      False
AAPL     True
dtype: bool
In [5]:
pd.notnull(aSer)
Out[5]:
AXP      True
CSCO     True
BA       True
AAPL    False
dtype: bool
```

Series的索引可以通过赋值方式就地修改。

<!---more--->

#### DataFrame

将嵌套字典传给DataFrame，会将外层字典的键作为列索引，内层键作为行索引。

```
pop={'Nevada':{2001:2.4,2002:2.9},'Ohio':{2000:1.5,2001:1.7,2002:3.6}}
frame=DataFrame(pop)
frame
Out[8]:
Nevada	Ohio
2000	NaN	1.5
2001	2.4	1.7
2002	2.9	3.6
```

对结果转置：

```
frame.T
Out:
2000	2001	2002
Nevada	NaN	2.4	2.9
Ohio	1.5	1.7	3.6
```

DataFrame的values属性会以二维ndarray形式返回

```
frame.values
out:
array([[ nan,  1.5],
       [ 2.4,  1.7],
       [ 2.9,  3.6]])
```

若各列数据类型不同，则返回array的数据类型会选取可以兼容所有列的数据类型。

#### 索引对象

构建Series和DataFrame时，任何数组或标签都会被转化为一个Index对象：

```
obj=Series(range(3),index=['a','b','c'])
obj.index
Out[12]:
Index([u'a', u'b', u'c'], dtype='object')
```

Index对象具有不可修改性（immutable）

Index的功能类似于固定大小的集合。

### 基本功能

#### 重新索引

对Series重新索引及填充缺失值：

```
obj=Series(range(4),index=['d','b','a','c'])
obj2=obj.reindex(['a','b','c','d','e'])
obj2
Out[6]:
a    2.0
b    1.0
c    3.0
d    0.0
e    NaN
dtype: float64
obj.reindex(['a','b','c','d','e'],fill_value=0)
Out[7]:
a    2
b    1
c    3
d    0
e    0
dtype: int64
```

重新索引时前向值填充（后向可以使用bfill）

```
obj=Series(['a','b','c'],index=[0,2,4])
obj.reindex(range(6),method='ffill')
Out[9]:
0    a
1    a
2    b
3    b
4    c
5    c
dtype: object
```

对于DataFrame，reindex可以修改索引和列名，仅传入一个参数时重新索引index，使用关键字column可重新索引列名。也可以同时索引，但插值只能按纵轴应用：

```
frame=DataFrame(np.arange(9).reshape(3,3),index=['a','c','d'],columns=['Ohio','Texas','California'])
state=['Texas','Utah','California']
frame.reindex(['a','b','c','d'],method='ffill',columns=state)
Out[19]:
Texas	Utah	California
a	1	NaN	2
b	1	NaN	2
c	4	NaN	5
d	7	NaN	8
```

利用ix标签索引，也可以达到重新索引效果：

```
frame.ix[['a','b','c','d'],state]
```

#### 丢弃指定轴上的项

通过axis指定轴：

```
data=DataFrame(np.arange(16).reshape(4,4),index=['Ohio','Texas','Utah','California'],columns=['1','2','3','4'])
```

|            | 2    | 3    | 4    |
| ---------- | ---- | ---- | ---- |
| Ohio       | 1    | 2    | 3    |
| Texas      | 5    | 6    | 7    |
| Utah       | 9    | 10   | 11   |
| California | 13   | 14   | 15   |

#### 索引、选取、过滤

利用标签切片包含末端：

```
obj=Series(np.arange(4.0),index=['a','b','c','d'])
obj['b':'c']
Out[32]:
b    1.0
c    2.0
dtype: float64
```

对DataFrame选取：

>obj.ix[val] 选取单个行
>
>obj.ix[:,nal] 选取单个列或列子集
>
>obj.ix[val1,val2] 同时选取行列

```
data=DataFrame(np.arange(16).reshape(4,4),index=['Ohio','Texas','Utah','California'],columns=['one','two','three','four'])
data
```

|            | one  | two  | three | four |
| ---------- | ---- | ---- | ----- | ---- |
| Ohio       | 0    | 1    | 2     | 3    |
| Texas      | 4    | 5    | 6     | 7    |
| Utah       | 8    | 9    | 10    | 11   |
| California | 12   | 13   | 14    | 15   |

```
data.ix[2]
```

```
one       8
two       9
three    10
four     11
Name: Utah, dtype: int64
```

```
data.ix[:,'two']
```

```
Ohio           1
Texas          5
Utah           9
California    13
Name: two, dtype: int64
```

```
data.ix[data.three>5,:3]
```

|            | one  | two  | three |
| ---------- | ---- | ---- | ----- |
| Texas      | 4    | 5    | 6     |
| Utah       | 8    | 9    | 10    |
| California | 12   | 13   | 14    |

#### 算数运算、数据对齐

填充NA值：

```
df1=DataFrame(np.arange(12).reshape((3,4)),columns=list('abcd'))
df2=DataFrame(np.arange(20).reshape((4,5)),columns=list('abcde'))
df1.add(df2,fill_value=0)
```

|      | a    | b    | c    | d    | e    |
| ---- | ---- | ---- | ---- | ---- | ---- |
| 0    | 0.0  | 2.0  | 4.0  | 6.0  | 4.0  |
| 1    | 9.0  | 11.0 | 13.0 | 15.0 | 9.0  |
| 2    | 18.0 | 20.0 | 22.0 | 24.0 | 14.0 |
| 3    | 15.0 | 16.0 | 17.0 | 18.0 | 19.0 |

DataFrame和Series间运算：

默认匹配column，沿行向下广播。

如果需要匹配Index，沿列运算，必须使用算术运算方法，并指定轴

```
df1=DataFrame(np.arange(12).reshape((3,4)),columns=list('abcd'),index=[1,2,3])
series1=df1['b']
df1.sub(series1,axis=0)
```

|      | a    | b    | c    | d    |
| ---- | ---- | ---- | ---- | ---- |
| 1    | -1   | 0    | 1    | 2    |
| 2    | -1   | 0    | 1    | 2    |
| 3    | -1   | 0    | 1    | 2    |

#### 函数应用和映射

DataFrame的apply方法默认axis=0

```
df1=DataFrame(np.random.rand(3,4),columns=list('abcd'),index=[1,2,3])
df1
```

|      | a        | b        | c        | d        |
| ---- | -------- | -------- | -------- | -------- |
| 1    | 0.002007 | 0.801513 | 0.260836 | 0.373641 |
| 2    | 0.033373 | 0.976768 | 0.853929 | 0.453304 |
| 3    | 0.914533 | 0.539858 | 0.268698 | 0.655732 |

```
df1.apply(f)
```

```
a    0.912527
b    0.436910
c    0.593093
d    0.282091
dtype: float64
```

```
df1.apply(f,axis=1)
```

```
1    0.799506
2    0.943394
3    0.645835
dtype: float64
```

传递给apply的函数可以返回Series：

```
def f(x):
    return Series([x.min(),x.max()],index=['min','max'])
df1.apply(f)
```

|      | a        | b        | c        | d        |
| ---- | -------- | -------- | -------- | -------- |
| min  | 0.002007 | 0.539858 | 0.260836 | 0.373641 |
| max  | 0.914533 | 0.976768 | 0.853929 | 0.655732 |

```
df1.apply(f,axis=1)
```

|      | min      | max      |
| ---- | -------- | -------- |
| 1    | 0.002007 | 0.801513 |
| 2    | 0.033373 | 0.976768 |
| 3    | 0.268698 | 0.914533 |

#### 排序和排名

按照行或列索引进行排序（字典序）可使用sort_index方法

默认按升序排列，可以指定ascending=False按降序排列

按值排序，可以使用order方法

对于DataFrame，可以指定by选项，按一或多个列中的值进行排序

```
frame
frame=DataFrame({'b':[4,7,-3,2],'a':[0,1,0,-1]})
frame
Out[3]:
a	b
0	0	4
1	1	7
2	0	-3
3	-1	2
frame.sort_values(by='b')
Out[5]:
a	b
2	0	-3
3	-1	2
0	0	4
1	1	7
frame.sort_values(by=['a','b'])
Out[7]:
a	b
3	-1	2
2	0	-3
0	0	4
1	1	7
```

Series和DataFrame的rank方法，会对序列产生一个排名值，同时按照特定功能处理平级的数据间的平级关系。

默认通过分配平均排名来破坏平级关系：

```
obj=Series([7,-5,7,-4,2,0,-4])
obj.rank()
Out[9]:
0    6.5
1    1.0
2    6.5
3    2.5
4    5.0
5    4.0
6    2.5
dtype: float64   #如第0,2个数据排名为6,7，平均排名为6.5，第3,6个数据排名为2,3，平均排名2.5
```

可以按照值在原数据中出现的顺序分配排名：

```
obj.rank(method='first')
Out[11]:
0    6.0
1    1.0
2    7.0
3    2.0
4    5.0
5    4.0
6    3.0
dtype: float64
```

或统一使用分组的最大排名：（同时降序排列）

```
obj.rank(ascending=False,method='max')
Out[12]:
0    2.0
1    7.0
2    2.0
3    6.0
4    3.0
5    4.0
6    6.0
dtype: float64 #1,2统一标记为2,5,6统一标记为6
```

#### 带有重复值的轴索引

Series和DataFrame可以带有重复索引值

判断索引值是否唯一：

```
obj=Series(range(5),index=['a','a','b','c','d'])
obj.index.is_unique
Out[18]:
False
```

对于DataFrame，检测方法类似：

```
df=DataFrame(np.arange(12).reshape((3,4)),columns=['one','one','two','three'],index=['a','a','b'])
df
Out[25]:
one	one	two	three
a	0	1	2	3
a	4	5	6	7
b	8	9	10	11
df.columns.is_unique
Out[32]:
False
df.index.is_unique
Out[33]:
False
```

进行索引：

```
df['one']
```

Out[34]:

|      | one  | one  |
| ---- | ---- | ---- |
| a    | 0    | 1    |
| a    | 4    | 5    |
| b    | 8    | 9    |

```
df.ix['a']
```

|      | one  | one  | two  | three |
| ---- | ---- | ---- | ---- | ----- |
| a    | 0    | 1    | 2    | 3     |
| a    | 4    | 5    | 6    | 7     |

### 汇总和计算描述统计

在计算DataFrame的描述值时，NA值会被自动排除，除非整个行或列均为NA：

```
df=DataFrame([[1.4,np.nan],[7.1,-4.5],[np.nan,np.nan],[0.75,-1.3]],index=['a','b','c','d'],columns=['one','two'])
df.mean()
Out[47]:
one    3.083333
two   -2.900000
dtype: float64
```

可以通过skipna选项禁用：

```
df.sum(axis=1,skipna=False)
Out[49]:
a     NaN
b    2.60
c     NaN
d   -0.55
dtype: float64
```

有些方法返回间接统计：（达到某值的索引）

```
df.idxmax()
Out[11]:
one    b
two    d
dtype: object
np.argmax(df.one)  #该方法只能接受Series
Out[12]:
'b'
```

对于非数值数据，describe方法会产生如下统计：

```
obj=Series(['a','b','b','c','d']*5)
obj.describe()
Out[59]:
count     25
unique     4
top        b
freq      10
dtype: object
```

#### 相关系数和协方差

DataFrame的corr和cov方法可以返回两组数据的相关系数和协方差：

首先从雅虎财经获取几家公司的股票价格和成交量

```
from pandas_datareader import data, wb
all_data={}
for ticker in ['AAPL','IBM','MSFT','GOOG']:
    all_data[ticker]=data.get_data_yahoo(ticker,'1/1/2000','1/1/2010')
price=DataFrame({tic:data['Adj Close'] for tic,data in all_data.iteritems()})
volume=DataFrame({tic:data['Volume'] for tic,data in all_data.iteritems()})
```

计算价格的百分比变化：

```
returns=price.pct_change()
```

计算两列数值之间的相关系数或协方差：

```
returns['MSFT'].corr(returns['IBM'])
Out[101]:
0.4959796429132885
```

可以直接使用corr/cov方法返回完整的相关系数或协方差矩阵：

```
returns.corr()
```

|      | AAPL     | GOOG     | IBM      | MSFT     |
| ---- | -------- | -------- | -------- | -------- |
| AAPL | 1.000000 | 0.470676 | 0.410011 | 0.424305 |
| GOOG | 0.470676 | 1.000000 | 0.390689 | 0.443587 |
| IBM  | 0.410011 | 0.390689 | 1.000000 | 0.495980 |
| MSFT | 0.424305 | 0.443587 | 0.495980 | 1.000000 |

可以使用corrwith方法，计算某一列跟所有列间的相关系数，或计算两个DataFrame相应列之间的相关系数：

```
returns.corrwith(returns['AAPL'])
Out[107]:
AAPL    1.000000
GOOG    0.470676
IBM     0.410011
MSFT    0.424305
dtype: float64
returns.corrwith(volume)
Out[108]:
AAPL   -0.057549
GOOG    0.062647
IBM    -0.007892
MSFT   -0.014245
dtype: float64
```

#### 唯一值、计数相关：

unique函数用于得到Series中的唯一值：

```
obj=Series(['a','b','b','c','d']*5)
uniques=obj.unique()
uniques
Out[115]:
array(['a', 'b', 'c', 'd'], dtype=object)
```

value_count用于计算各值出现频率，默认按频率排序，可以指定参数进行限制。

```
obj.value_counts(sort=False)
Out[124]:
a     5
c     5
b    10
d     5
```

可以将该方法传递至DataFrame的apply函数，统计每一列的频率计数：

```
data=DataFrame([[1,3,4,3,4],[2,3,1,2,3],[1,5,2,4,4]],index=['Qu1','Qu2','Qu3']).T
data.apply(pd.value_counts).fillna(0)
Out[17]:
Qu1	Qu2	Qu3
1	1.0	1.0	1.0
2	0.0	2.0	1.0
3	2.0	2.0	0.0
4	2.0	0.0	2.0
5	0.0	0.0	1.0
```

### 处理缺失数据

dropna方法可以过滤掉DataFrame的含NA值的行或列，默认对行操作、丢弃包含任一NA值的行：

```
NA=np.nan
data=DataFrame([[1,6,3],[1,NA,NA],[NA,NA,NA],[NA,6,3]])
data
Out[18]:
0	1	2
0	1.0	6.0	3.0
1	1.0	NaN	NaN
2	NaN	NaN	NaN
3	NaN	6.0	3.0
data.dropna() 
Out[19]:
0	1	2
0	1.0	6.0	3.0
```

设置选项how丢弃全部为NA的行：

```
data.dropna(how='all') 
Out[20]:
0	1	2
0	1.0	6.0	3.0
1	1.0	NaN	NaN
3	NaN	6.0	3.0
```

参数thresh过滤少于n个非NA值的行或列：

```
data=DataFrame(np.random.randn(7,3))
data.ix[:4,1] = NA
data.ix[:2,2] = NA
data
```

|      | 0         | 1         | 2         |
| ---- | --------- | --------- | --------- |
| 0    | -1.343368 | NaN       | NaN       |
| 1    | 0.833353  | NaN       | NaN       |
| 2    | -0.810085 | NaN       | NaN       |
| 3    | 0.791986  | NaN       | 2.076980  |
| 4    | 1.033117  | NaN       | -0.441156 |
| 5    | -0.802218 | -0.427902 | -0.437456 |
| 6    | 0.616425  | 1.052126  | -0.242327 |

```
data.dropna(thresh=3,axis=1)
```

|      | 0         | 2         |
| ---- | --------- | --------- |
| 0    | -1.343368 | NaN       |
| 1    | 0.833353  | NaN       |
| 2    | -0.810085 | NaN       |
| 3    | 0.791986  | 2.076980  |
| 4    | 1.033117  | -0.441156 |
| 5    | -0.802218 | -0.437456 |
| 6    | 0.616425  | -0.242327 |

填充缺失数据：

通过字典调用fillna，可以实现对不同列填充不同值：

注意：fillna默认返回新的对象，可以指定inplace参数就地修改：

```
data2 = DataFrame(np.random.randn(7,3),columns=['one','two','three'])
data2.ix[:4,1] = NA
data2.ix[:2,2] = NA
data2.fillna({'two':0,'three':1},inplace=True)
data2
```

Out[31]:

|      | one       | two      | three     |
| ---- | --------- | -------- | --------- |
| 0    | -1.431984 | 0.000000 | 1.000000  |
| 1    | 0.884340  | 0.000000 | 1.000000  |
| 2    | -1.959415 | 0.000000 | 1.000000  |
| 3    | 1.037119  | 0.000000 | 0.205024  |
| 4    | 0.637704  | 0.000000 | -0.830259 |
| 5    | -1.574250 | 1.159501 | 0.343172  |
| 6    | -1.238427 | 0.934534 | -0.639167 |

插值填充方法对于fillna也有效，可以使用limit参数指定连续填充的最大数量：

```
data2.fillna(method='ffill')
Out[34]:
one	two	three
0	-0.146278	0.529139	-1.540249
1	-1.003318	2.270424	-0.939689
2	0.056639	2.270424	-0.757976
3	0.703487	2.270424	1.106422
4	-0.242079	2.270424	1.106422
5	-0.007966	2.270424	1.106422
6	1.021809	2.270424	1.106422
data2.fillna(method='ffill',limit=2)
Out[35]:
one	two	three
0	-0.146278	0.529139	-1.540249
1	-1.003318	2.270424	-0.939689
2	0.056639	2.270424	-0.757976
3	0.703487	2.270424	1.106422
4	-0.242079	NaN	1.106422
5	-0.007966	NaN	1.106422
6	1.021809	NaN	NaN
```

### 层次化索引

层次化索引可以在一个轴上创建多个索引级别：

```
data
data=Series(np.random.randn(10),index=[['a','a','a','b','b','b','c','c','d','d'],[1,2,3,1,2,3,1,2,2,3]])
data
Out[3]:
a  1   -0.268003
   2    0.102808
   3    0.478865
b  1   -1.386811
   2   -0.032409
   3   -0.711963
c  1    0.531615
   2   -0.257716
d  2    0.531573
   3   -0.600997
dtype: float64
```

可以方便地选取子集或在内层中选取：

```
data['a']
Out[9]:
1   -0.268003
2    0.102808
3    0.478865
dtype: float64
data[:,1]
Out[10]:
a   -0.268003
b   -1.386811
c    0.531615
dtype: float64
```

可以通过unstack方法将数据安排至一个DataFrame中：

```
data.unstack()
```

|      | 1         | 2         | 3         |
| ---- | --------- | --------- | --------- |
| a    | -0.268003 | 0.102808  | 0.478865  |
| b    | -1.386811 | -0.032409 | -0.711963 |
| c    | 0.531615  | -0.257716 | NaN       |
| d    | NaN       | 0.531573  | -0.600997 |

对于DataFrame，每个轴都可以分层索引：

```
frame=DataFrame(np.arange(12).reshape((4,3)),index=[['a','a','b','b'],[1,2,1,2]],columns=[['Ohio','Ohio','Colorado'],['Green','Red','Green']])
frame
```

|      |      | Ohio  | Colorado |       |
| ---- | ---- | ----- | -------- | ----- |
|      |      | Green | Red      | Green |
| a    | 1    | 0     | 1        | 2     |
|      | 2    | 3     | 4        | 5     |
| b    | 1    | 6     | 7        | 8     |
|      | 2    | 9     | 10       | 11    |

可以对各层指定名称：

```
frame.index.names=['key1','key2']
frame.columns.names=['state','color']
```

|      | state | Ohio  | Colorado |       |
| ---- | ----- | ----- | -------- | ----- |
|      | color | Green | Red      | Green |
| key1 | key2  |       |          |       |
| a    | 1     | 0     | 1        | 2     |
|      | 2     | 3     | 4        | 5     |
| b    | 1     | 6     | 7        | 8     |
|      | 2     | 9     | 10       | 11    |

可以通过swaplevel调整某轴上的级别顺序，在调整顺序时，可以使用sortlevel使调整后结果有序：

```
frame.swaplevel(axis=1)
```

|      | color | Green | Red  | Green    |
| ---- | ----- | ----- | ---- | -------- |
|      | state | Ohio  | Ohio | Colorado |
| key1 | key2  |       |      |          |
| a    | 1     | 0     | 1    | 2        |
|      | 2     | 3     | 4    | 5        |
| b    | 1     | 6     | 7    | 8        |
|      | 2     | 9     | 10   | 11       |

```
frame.swaplevel(axis=1).sortlevel(axis=1)
```

|      | color | Green    | Red  |      |
| ---- | ----- | -------- | ---- | ---- |
|      | state | Colorado | Ohio | Ohio |
| key1 | key2  |          |      |      |
| a    | 1     | 2        | 0    | 1    |
|      | 2     | 5        | 3    | 4    |
| b    | 1     | 8        | 6    | 7    |
|      | 2     | 11       | 9    | 10   |

可以根据level进行汇总：

```
frame.sum(level='color',axis=1)
```

|      | color | Green | Red  |
| ---- | ----- | ----- | ---- |
| key1 | key2  |       |      |
| a    | 1     | 2     | 1    |
|      | 2     | 8     | 4    |
| b    | 1     | 14    | 7    |
|      | 2     | 20    | 10   |

DataFrame的列与行索引转换：

```
frame=DataFrame({'a':range(7),'b':range(7,0,-1),'c':['one','one','one','two','two','two','two'],'d':[0,1,2,0,1,2,3]})
frame
```

|      | a    | b    | c    | d    |
| ---- | ---- | ---- | ---- | ---- |
| 0    | 0    | 7    | one  | 0    |
| 1    | 1    | 6    | one  | 1    |
| 2    | 2    | 5    | one  | 2    |
| 3    | 3    | 4    | two  | 0    |
| 4    | 4    | 3    | two  | 1    |
| 5    | 5    | 2    | two  | 2    |
| 6    | 6    | 1    | two  | 3    |

```
frame.set_index(['c','d'])
```

|      |      | a    | b    |
| ---- | ---- | ---- | ---- |
| c    | d    |      |      |
| one  | 0    | 0    | 7    |
|      | 1    | 1    | 6    |
|      | 2    | 2    | 5    |
| two  | 0    | 3    | 4    |
|      | 1    | 4    | 3    |
|      | 2    | 5    | 2    |
|      | 3    | 6    | 1    |

逆函数为reset_index.