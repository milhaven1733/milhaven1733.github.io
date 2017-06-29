---
title: 《利用Python进行数据分析》范例数据集_babynames
date: 2017-05-04 18:34:31
tags:
categories: 数据分析
---

《利用Python进行数据分析》范例数据集-babynames

```python
import pandas as pd
```

载入包含1880年美国新出生婴儿名字统计数据的文件

```python
path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/names/yob1880.txt'
```

加载为DataFrame

```python
names1880=pd.read_csv(path,names=['names','sex','birth'])
```

```python
names1880.head()
```

|      | names     | sex  | birth |
| ---- | --------- | ---- | ----- |
| 0    | Mary      | F    | 7065  |
| 1    | Anna      | F    | 2604  |
| 2    | Emma      | F    | 2003  |
| 3    | Elizabeth | F    | 1939  |
| 4    | Minnie    | F    | 1746  |

<!---more--->

#### 统计总出生数

以‘sex’分组统计birth人数总数：

```python
names1880.groupby('sex').birth.sum()
```

Outs:

```
sex
F     90993
M    110493
Name: birth, dtype: int64
```

将1880-2010年所有数据文件加载至一个DataFrame

```python
pieces=[]
def get_df_pieces(year):
    path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/names/yob%d.txt'%year
    df=pd.read_csv(path,names=['names','sex','birth'])
    df['year']=year
    pieces.append(df)
    return pieces
years=range(1880,2011)
for year in years:
    get_df_pieces(year)
```

用pd.concat连接pieces中的每个元素：

```python
names=pd.concat(pieces,ignore_index=True)
```

```python
names.info()
```

```
<class 'pandas.core.frame.DataFrame'>
RangeIndex: 1690784 entries, 0 to 1690783
Data columns (total 4 columns):
names    1690784 non-null object
sex      1690784 non-null object
birth    1690784 non-null int64
year     1690784 non-null int64
dtypes: int64(2), object(2)
memory usage: 51.6+ MB
```

数据聚合。利用pivot_table或groupby在year和sex级别上进行聚合

```python
total_birth=names.pivot_table('birth',index='year',columns='sex',aggfunc=sum)
#total_birth=names.groupby(['year','sex']).birth.sum().unstack('sex').tail()
total_birth.tail()
```

| sex  | F       | M       |
| ---- | ------- | ------- |
| year |         |         |
| 2006 | 1896468 | 2050234 |
| 2007 | 1916888 | 2069242 |
| 2008 | 1883645 | 2032310 |
| 2009 | 1827643 | 1973359 |
| 2010 | 1759010 | 1898382 |

对分组统计结果绘图：

```python
%matplotlib inline
import seaborn as sns
total_birth.plot(title='Total births by sex and year')
```

![png](《利用Python进行数据分析》范例数据集-babynames\output_16_1.png)

#### 分析命名趋势

添加一个prop列，表示当年某名字婴儿数对于总出生数的比例（可以先按年份和性别分组，对每个分组进行处理，再验证完整数据集的结果变化）

```python
def add_prop(group):
    birth=group.birth.astype(float)
    group['prop']=birth/birth.sum()
    return group
names=names.groupby(['year','sex']).apply(add_prop)
```

```python
names.head()
```

|      | names     | sex  | birth | year | prop     |
| ---- | --------- | ---- | ----- | ---- | -------- |
| 0    | Mary      | F    | 7065  | 1880 | 0.077643 |
| 1    | Anna      | F    | 2604  | 1880 | 0.028618 |
| 2    | Emma      | F    | 2003  | 1880 | 0.022013 |
| 3    | Elizabeth | F    | 1939  | 1880 | 0.021309 |
| 4    | Minnie    | F    | 1746  | 1880 | 0.019188 |

验证分组总和是否为1

```python
import numpy as np
np.allclose(names.groupby(['year','sex']).prop.sum(),1)
#np.allclose(names.groupby(['year']).prop.sum(),2)
```

Outs:

```
True
```

由于数据集过大，我们可以取出每个['year','sex']分组中总数目排在前1000的名字，作为之后分析工作的数据集：

```python
def get_top1000(group):
    return group.sort_values(by='birth',ascending=False)[:1000]
top1000=names.groupby(['year','sex']).apply(get_top1000)
```

```python
top1000.info()
```

```
<class 'pandas.core.frame.DataFrame'>
MultiIndex: 261877 entries, (1880, F, 0) to (2010, M, 1677645)
Data columns (total 5 columns):
names    261877 non-null object
sex      261877 non-null object
birth    261877 non-null int64
year     261877 non-null int64
prop     261877 non-null float64
dtypes: float64(1), int64(2), object(2)
memory usage: 13.7+ MB
```

按性别区分出2个子数据集：

```python
boys=top1000[top1000.sex=='M']
girls=top1000[top1000.sex=='F']
```

按照年份和名字分组统计，抽取几个名字的birth数据，绘图显示130年间该名字的使用趋势：

```python
total_births=top1000.pivot_table('birth',index='year',columns='names',aggfunc=sum)
```

```python
subset=total_births[['John','Harry','Mary','Marilyn']]
```

```python
subset.plot(subplots=True,figsize=(12,10),title='Number of births per year')
```

Outs:



![png](《利用Python进行数据分析》范例数据集-babynames\output_30_1.png)

#### 评估命名多样性的增长

首先可以统计取最流行的1000个名字的婴儿占所有婴儿比例的变化趋势：

```python
table=top1000.pivot_table('prop',index='year',columns='sex',aggfunc=sum)
```

```python
table.plot(xticks=range(1880,2020,10),yticks=np.linspace(0,1.1,12))
```

Outs:



![png](《利用Python进行数据分析》范例数据集-babynames\output_34_1.png)

另外可以计算占总出生人数50%的名字的数量，先以2010年出生的男孩为例:

```python
df=boys[boys.year==2010]
```

排序后计算累积和：

```python
prop_cumsum=df.sort_values(by='prop',ascending=False).prop.cumsum()
```

```python
prop_cumsum.head()
```

Outs:

```
year  sex         
2010  M    1676644    0.011523
           1676645    0.020934
           1676646    0.029959
           1676647    0.038930
           1676648    0.047817
Name: prop, dtype: float64
```

searchsorted方法求出0.5插入序列且不破坏顺序的位置：

```python
prop_cumsum.searchsorted(0.5)
```

Outs:

```
array([116])
```

对所有组合执行计算，绘图表示趋势：

```python
def get_quantile_count(group):
    group=group.sort_values(by='prop',ascending=False)
    return group.prop.cumsum().searchsorted(0.5)[0]+1
diversity=top1000.groupby(['year','sex']).apply(get_quantile_count)
diversity=diversity.unstack('sex')
diversity.head()
```

| sex  | F    | M    |
| ---- | ---- | ---- |
| year |      |      |
| 1880 | 38   | 14   |
| 1881 | 38   | 14   |
| 1882 | 38   | 15   |
| 1883 | 39   | 15   |
| 1884 | 39   | 16   |

```python
diversity.plot(title='Number of popular names in top 50%')
```

Outs:



![png](《利用Python进行数据分析》范例数据集-babynames\output_44_1.png)

#### 探究名字最后一个字母的变革

首先为names添加表示名字末字母的一列，将全部出生数据进行聚合：

```python
get_last_letter=lambda x:x[-1]
last_letters=names.names.apply(get_last_letter)
names['last_latter']=last_letters
table=names.pivot_table('birth',index='last_latter',columns=['sex','year'],aggfunc=sum)
```

取出有代表性的三列：

```python
subtable=table.reindex(columns=[1910,1960,2010],level='year')
```

```python
subtable.head()
```

| sex         | F        | M        |          |         |          |          |
| ----------- | -------- | -------- | -------- | ------- | -------- | -------- |
| year        | 1910     | 1960     | 2010     | 1910    | 1960     | 2010     |
| last_latter |          |          |          |         |          |          |
| a           | 108376.0 | 691247.0 | 670605.0 | 977.0   | 5204.0   | 28438.0  |
| b           | NaN      | 694.0    | 450.0    | 411.0   | 3912.0   | 38859.0  |
| c           | 5.0      | 49.0     | 946.0    | 482.0   | 15476.0  | 23125.0  |
| d           | 6750.0   | 3729.0   | 2607.0   | 22111.0 | 262112.0 | 44398.0  |
| e           | 133569.0 | 435013.0 | 313833.0 | 28655.0 | 178823.0 | 129012.0 |

计算各年份末字母所占比例，绘图：

```python
letter_prop=subtable/subtable.sum(0)
#letter_prop=subtable.div(subtable.sum(0))
```

```python
import matplotlib.pyplot as plt
fig,axes=plt.subplots(2,1,figsize=(10,8))
letter_prop['M'].plot(kind='bar',ax=axes[0],title='Male')
letter_prop['F'].plot(kind='bar',ax=axes[1],title='Female')
```

Outs:

![png](《利用Python进行数据分析》范例数据集-babynames\output_53_1.png)

#### 探究一些女性化的“男性名字”的变化趋势：

从全部的名字中取出包含'lesl'的集合：

```python
all_names=pd.Series(top1000.names.unique())
def lesley(str):
    return 'lesl' in str.lower()
lesley_like=all_names[all_names.apply(lesley)]
```

从top1000中取出名字在lesley_like中的分组：

```python
filtered=top1000[top1000.names.isin(lesley_like)]
```

```python
filtered.groupby('names').sum().birth
```



```
names
Leslee      1082
Lesley     35022
Lesli        929
Leslie    370429
Lesly      10067
Name: birth, dtype: int64
```



按年份和性别进行聚合：

```python
table=filtered.pivot_table('birth',index='year',columns='sex',aggfunc='sum')
```

计算各年份各性别某名字的性别比例，并绘图展示变化趋势：

```python
table=table.div(table.sum(1),axis=0)
```

```python
table.plot(style={'M':'-','F':'--'})
```

![png](《利用Python进行数据分析》范例数据集-babynames\output_64_1.png)

