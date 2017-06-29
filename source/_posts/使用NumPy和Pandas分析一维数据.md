---
title: 使用NumPy和Pandas分析一维数据
date: 2017-04-22 10:25:46
tags:
categories: 数据分析
---

#### 课程内容概述

- 熟悉NumPy的使用
- 熟悉Pandas库的使用
- 两种库的优势：降低编写数据分析代码的难度，提高代码的运算速度。

<!---more--->

#### 数据准备

从 [gapminder.org](http://www.gapminder.org/data/) 中下载一些课程分析所需的数据集，包含：

- 不同国家就业水平（15 岁以上人口就业率 (%)）
- 预期寿命（年）
- 人均 GDP（美元，已经过通胀调整)
- 教育普及率（% 男性）
- 教育普及率（% 女性）

通过观察某数据集，如不同国家就业率变化表，我们可以提出诸如以下的问题：

- 某国家的就业率随时间发生了怎样的变化？
- 最高和最低的就业水平出现在哪个国家？我国处于哪个水平？
- 各国就业率有没有相同的持续性趋势？

等。

#### NumPy和Pandas中的一维数据

利用Pandas处理csv文档具有很多优势。

首先是：

**编写代码便捷**

没有引入Pandas之前，我们读取一个csv文件，并统计数据集中某个字段的不重复量需要经过如下处理：

```
import unicodecsv 
def read_csv(filename):
    with open(filename, 'rb') as f:
        reader = unicodecsv.DictReader(f)
        return list(reader)
def get_unique_students(data):
    unique_students=set()
    for data_point in data:
        unique_students.add(data_point['acct'])
    return unique_students
daily_engagement = read_csv('/home/milhaven1733/daily_engagement.csv')
unique_engagement_students=get_unique_students(daily_engagement)
print len(unique_engagement_students)
```

引入Pandas之后，大大缩短了代码量：

```
import pandas as pd
daily_engagement=pd.read_csv('/home/milhaven1733/daily_engagement.csv')
len(daily_engagement['acct'].unique())
```

其次是：

**运算、处理速度高**

现在有一个包含100万条左右记录的csv，在引入Pandas之前用read_csv方法读入，统计用时如下：

```
import unicodecsv 
import time
def read_csv(filename):
    with open(filename, 'rb') as f:
        reader = unicodecsv.DictReader(f)
        return list(reader)
start = time.time()
daily_engagement = read_csv('/home/milhaven1733/daily_engagement_full.csv')
end=time.time()
print end-start
```

```
Out：
48.103812933
```

现在引入Pandas，同样计算读取用时，速度在未引入前的十倍以上：

```
import pandas as pd
start = time.time()
daily_engagement=pd.read_csv('/home/milhaven1733/daily_engagement_full.csv')
end=time.time()
print end-start
```

```
Out：
3.31470179558
```

统计不重复的字段数，两者对比，可以发现使用Pandas速度同样有了很大的提升，神器！

```
start = time.time()
unique_engagement_students=get_unique_students(daily_engagement)
print len(unique_engagement_students)
end=time.time()
print end-start
```

```
1.04902696609
```

```
start = time.time()
len(daily_engagement['acct'].unique())
end=time.time()
print end-start
```

```
0.027951002121
```

#### NumPy数组

Pandas和NumPy都有表示一维、二维数组的特殊数据结构。其中一维的数组：

Pandas——series

NumPy——array

series建立在array基础上

对比NumPy array和Python list：

相同点：

- 按顺序排列，可以通过位置获取元素
- 可以利用数据切片
- 可以使用循环

不同点：

- NumPy数组中各个元素必须为同一类别（int，boolean，string等）
- NumPy数组有很多方便使用的函数且运行速度很快。
- NumPy 数组可以是多维的。

NumPy数组的建立：

```
import numpy as np
countries = np.array([
    'Afghanistan', 'Albania', 'Algeria', 'Angola', 'Argentina',
    'Armenia', 'Australia', 'Austria', 'Azerbaijan', 'Bahamas',
    'Bahrain', 'Bangladesh', 'Barbados', 'Belarus', 'Belgium',
    'Belize', 'Benin', 'Bhutan', 'Bolivia',
    'Bosnia and Herzegovina'
])
employment = np.array([
    55.70000076,  51.40000153,  50.5       ,  75.69999695,
    58.40000153,  40.09999847,  61.5       ,  57.09999847,
    60.90000153,  66.59999847,  60.40000153,  68.09999847,
    66.90000153,  53.40000153,  48.59999847,  56.79999924,
    71.59999847,  58.40000153,  70.40000153,  41.20000076
])
```

首先import NumPy 创建一个list 调用np.array,将其转换为NumPy array。

查看array的元素类型：

```
print countries.dtype
print employment.dtype
print np.array([True, False, True]).dtype
```

```
|S22 #s表示字符串，22表示最大串长度
float64
|S2
```

计算一个array中数据的均值、标准差、最大值、总和：

```
print employment.mean()
print employment.std()
print employment.max()
print employment.sum()
```

练习：编写函数，返回employment数组的最大值和它对应的countries数组中的国家。

```
def max_employment(countries, employment):
    x=employment.argmax()
    max_country = countries[x]   
    max_value = employment[x]   
    return (max_country, max_value)
```

array.argmax() 方法，返回最大值下标.

#### 向量化运算

在Python list中，将两个列表相加，返回的是列表连接的结果，如：

```
a=[1,2,3]
b=[4,5,6]
print a+b
Out:
[1, 2, 3, 4, 5, 6]
```

但NumPy array支持向量运算，支持两个长度相同的向量相加：

```
a=np.array([1,2,3])
b=np.array([4,5,6])
print a+b
Out:
[5, 7, 9]
```

#### 向量（vector）乘以标量（ scalar）

```
a=[1,2,3]
b=np.array([1,2,3])
print a*3
print b*3
Out：
[1, 2, 3, 1, 2, 3, 1, 2, 3]
[3 6 9]
```

在Python list中，list*num表示将list中所有值重复num次

在NumPy array中，表示向量乘法计算。

#### 其他向量运算

| Math Operations | Logical Operations | Comparison Operations |
| --------------- | ------------------ | --------------------- |
| +               | &（and）             | \>                    |
| -               | \|（or）             | \>=                   |
| *               | ～（not）             | <                     |
| /               |                    | <=                    |
| **              |                    | ==                    |
|                 |                    | !=                    |

Math Operation 用于两个array间或一个array和一个数字之间

Logical Operations 用于两个布尔型array之间，如果数组类型为整数，运算将执行按位与、按位或、按位取反

向量的算数运算：

```
a = np.array([1, 2, 3, 4])
b = np.array([1, 2, 1, 2])
print a * b
print a / b
print a ** b
Out：
[1 4 3 8]
[1 1 3 2]
[ 1  4  3 16]
```

```
a = np.array([1, 2, 3, 4])
b = 2
print a + b
print a - b
print a * b
print a / b
print a ** b
Out:
[3 4 5 6]
[-1  0  1  2]
[2 4 6 8]
[0 1 1 2]
[ 1  4  9 16]
```

**注意一点，在Python2中，浮点数或整数除以整数，得到的是整除的结果。如果想保留结果小数点后的值，需要在除数后加小数点。**

向量的逻辑运算：

```
a = np.array([True, True, False, False])
b = np.array([True, False, True, False])
print a & b
print a | b
print ~a
Out:
[ True False False False]
[ True  True  True False]
[False False  True  True]
```

向量的比较运算：

```
a = np.array([1, 2, 3, 4, 5])
b = np.array([5, 4, 3, 2, 1])
print a > b
print a >= b
print a < b
print a <= b
print a == b
print a != b
Out:
[False False False  True  True]
[False False  True  True  True]
[ True  True False False False]
[ True  True  True False False]
[False False  True False False]
[ True  True False  True  True]
```

#### 标准化数据点

标准化——将数据值转化为与总体均值相差多少个标准偏差

练习：编写标准化数据函数，返回某个NumPy array的标准化结果。

```
employment = np.array([
    55.70000076,  51.40000153,  50.5       ,  75.69999695,
    58.40000153,  40.09999847,  61.5       ,  57.09999847,
    60.90000153,  66.59999847,  60.40000153,  68.09999847,
    66.90000153,  53.40000153,  48.59999847,  56.79999924,
    71.59999847,  58.40000153,  70.40000153,  41.20000076
])
def standardize_data(values):
    standardize_values=(values-values.mean())/values.std()
    return standardize_values
standardize_data(employment)
```

#### NumPy索引数组

现在存在两个长度相同的NumPy array，其中第一个array类型任意，第二个为布尔型。如：

```
a=[1,2,3,4,5]
b=[False,False,True,True,True]
```

此时，执行a[b]将返回a array在b中对应索引为T的数值构成的array

```
a=np.array([1,2,3,4,5])
b=np.array([False,False,True,True,True])
print a[b]
Out:
[3,4,5]
```

相当于：

```
a=np.array([1,2,3,4,5])
b=a>2
print a[b]  
```

或：

```
a=np.array([1,2,3,4,5])
print a[a>2]
```

练习：现有time_spent、days_to_cancel两个相对应的数据集表示学生学习所花时间和注册到注销的时长，计算注册七天内未注销的学生的平均学习时间：

```
def mean_time_for_paid_students(time_spent, days_to_cancel):
    total_time=time_spent[days_to_cancel>=7]
    return total_time.mean()
```

#### NumPy + VS +=

NumPy中 运算符+ 和+=是有区别的

```
a=np.array([1,2,3,4])
b=a
a+=np.array([1,1,1,1])
print b
```

Out：

```
[2 3 4 5]
```

使用+=，更改了现有的数组。a，b array均被更改为[2 3 4 5]

```
a=np.array([1,2,3,4])
b=a
a=a+np.array([1,1,1,1])
print b
```

Out:

```
[1,2,3,4]
```

使用= 相当于创建了一个新的数组命名为a，a内容更新而b不变。

### # in-place VS NOT-in-place

+=通常被称为原位运算 （Operation in-place），将新值储存在原值所在位置 	

而 + 称为非原位运算（Operation NOT-in-place），创建新的内容（较常用）

NumPy array 针对切片的运算与Python list不同，如：

```
a=np.array([1,2,3,4,5])
slice=a[:3]
slice[0]=10
print a
```

Out:

```
[10  2  3  4  5]
```

即：对array生成切片时，没有创建新的数组，只是反映了array的情况，当更改切片时，原array也跟着改变。

因为不需创建新的数组，复制任何数据，这也使得NumPy 数据切片速度加快。

#### Pandas series

series与NumPy array类似，但在array基础上具有其他功能。（array适用的方法、函数series也适用）

如 series.describe()可以返回 series的相关统计数据。

```
gdp.describe()  #gdp在下方练习中定义
Out:
count       20.000000
mean      9147.879916
std       9763.958973
min        366.044967
25%       1362.124518
50%       2967.190270
75%      15495.296870
max      27036.487332
dtype: float64
```

练习：目前有部分国家平均寿命和GDP的数据，想检查各变量是否具有相关性。即预期寿命高（低）于平均值时，GDP是否也高（低）于平均值？编写函数统计两个值均高于或低于平均值的国家数量，以及一者高于均值而一者低的国家数量。

```
life_expectancy_values = [74.7,  75. ,  83.4,  57.6,  74.6,  75.4,  72.3,  81.5,  80.2,
                          70.3,  72.1,  76.4,  68.1,  75.2,  69.8,  79.4,  70.8,  62.7,
                          67.3,  70.6]
gdp_values = [ 1681.61390973,   2155.48523109,  21495.80508273,    562.98768478,
              13495.1274663 ,   9388.68852258,   1424.19056199,  24765.54890176,
              27036.48733192,   1945.63754911,  21721.61840978,  13373.21993972,
                483.97086804,   9783.98417323,   2253.46411147,  25034.66692293,
               3680.91642923,    366.04496652,   1175.92638695,   1132.21387981]
life_expectancy = pd.Series(life_expectancy_values)
gdp = pd.Series(gdp_values)
def variable_correlation(variable1, variable2):
    above=(variable1>variable1.mean())&(variable2>variable2.mean())
    below=(variable1<variable1.mean())&(variable2<variable2.mean())
    both_above_or_below=above|below
    num_same_direction =   both_above_or_below.sum() 
    num_different_direction = len(both_above_or_below)- num_same_direction
    return (num_same_direction, num_different_direction)
variable_correlation(life_expectancy,gdp)

Out:(17,3)
```

above统计是否两者均高于平均值，below统计是否两者均低于平均值，both_above_or_below统计是否符合前两项中一项。三者均返回布尔型的series

布尔型的series可以通过 .sum()方法返回其真值的数量，即符合两者均高于或低于均值的国家数量，再用总数量减去这一数量，得到不满足均高或低于均值的国家数量。

从结果看，这两者之间有较强的正相关关系。

#### series 索引

series和NumPy array的主要区别是：series有索引

如：

```
countries = ['Afghanistan', 'Albania', 'Algeria', 'Angola']
employment_values = [55.70000076,  51.40000153,  50.5,  75.69999695,]
employment = pd.Series(employment_values, index=countries)
print employment
Out：
Afghanistan    55.700001
Albania        51.400002
Algeria        50.500000
Angola         75.699997
dtype: float64
```

设置employment的索引为对应的国家名称。因此series更像list和dictionary的合体

series中的值可以通过位置和索引来查找，如：

```
employment[0]
employment.loc['Albania']
```

当不设置索引时，索引值默认从0排列。

练习：重新编写函数，返回employment series中的最大值和它对应的的国家。

```
employment = pd.Series(employment_values, index=countries)
def max_employment(employment):
    max_country = employment.argmax() 
    max_value = employment.max()  
    return (max_country, max_value)
```

employment.argmax() 返回最大值的索引，即对应的国家。

#### 向量运算与series索引

series索引对向量运算有一定影响。

将索引不同的series相加：

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['a', 'b', 'c', 'd'])
print s1 + s2
Out:
a    11
b    22
c    33
d    44
dtype: int64
```

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['b', 'd', 'a', 'c'])
print s1 + s2
Out:
a    31
b    12
c    43
d    24
dtype: int64
```

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['c', 'd', 'e', 'f'])
print s1 + s2
Out:
a     NaN
b     NaN
c    13.0
d    24.0
e     NaN
f     NaN
dtype: float64
```

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['e', 'f', 'g', 'h'])
print s1 + s2
Out:
a   NaN
b   NaN
c   NaN
d   NaN
e   NaN
f   NaN
g   NaN
h   NaN
dtype: float64
```

可以看出，索引相同顺序不同时，逐个进行相加；索引相同，顺序不同时，值的匹配根据索引进行。

索引不同时，相同索引的值相加，其余索引得到NaN，表示非数字。

#### 填充缺失值

如果不想让相加结果中出现NaN，可以怎样处理？

- 删除缺失值

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['c', 'd', 'e', 'f'])
result=s1+s2
print result.dropna()
Out:
c    13.0
d    24.0
dtype: float64
```

- 填充缺失值

可以在计算前将缺失索引的值视为0：

```
s1=s1.reindex(['a', 'b', 'c', 'd','e','f'],fill_value=0)
s2=s2.reindex(['a', 'b', 'c', 'd','e','f'],fill_value=0)
print s1 + s2
```

比较繁琐，需要填写所有索引。更便捷的有：

```
s1 = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
s2 = pd.Series([10, 20, 30, 40], index=['c', 'd', 'e', 'f'])
s1.add(s2,fill_value=0)
Out:
a     1.0
b     2.0
c    13.0
d    24.0
e    30.0
f    40.0
dtype: float64
```

用add方法取代 ‘+’，同时设置填充值

####  Series apply()

如何对 Series进行没有内置函数或无法通过向量运算的计算？

 Pandas为Series提供了apply()函数,

apply()载入 Series和函数，然后对 Series中的每个元素调用这个函数，并创建一个新的 Series

（类似map的apply函数）

如：

```
s = pd.Series([1, 2, 3, 4, 5])
def add_one(x):
    return x + 1
print s.apply(add_one)
Out:
0    2
1    3
2    4
3    5
4    6
dtype: int64
```

练习：目前有一个names Series，存储格式为“名 姓”，编写函数，使其转换为"姓, 名"的格式。

```
def reverse_name(names):
    name_split=names.split(' ')
    name_new=name_split[1]+" "+name_split[0]
    return name_new
names.apply(reverse_name)
```

#### 在 Pandas 中绘图

如果变量 data是一个 NumPy array或 Pandas Series，使用

```
import matplotlib.pyplot as plt
plt.hist(data)
```

将创建数据的直方图。

Pandas 还在后台使用 matplotlib 的内置绘图函数，因此如果 `data` 是一个 Series，你可以使用 `data.hist()` 创建直方图。

有时候 使用Pandas 封装器更加方便，如可以使用 `data.plot()` 创建 Series 的折线图。Series 索引被用于 x 轴，值被用于 y 轴。

在Jupyter notebook内绘图，需要添加 `%pylab inline`

首先，载入csv文件为“数据框”，并抽取index对应的记录：

`````
import pandas as pd
import seaborn as sns
path='/home/milhaven1733/Desktop/Data Analyst/data/'
employment = pd.read_csv(path + 'employment_above_15.csv', index_col='Country')
life_expectancy = pd.read_csv(path + 'life_expectancy.csv', index_col='Country')
employment_ch = employment.loc['China']
life_expectancy_ch = life_expectancy.loc['China']
​```

​```
employment_ch = employment.loc['China']
life_expectancy_ch = life_expectancy.loc['China']
gdp_ch=gdp.loc['China']
​```

绘制图形：

​```
%pylab inline
employment_ch.plot()
​```

![](使用NumPy和Pandas分析一维数据\1.png)

​```
life_expectancy_us.plot()
​```

![](使用NumPy和Pandas分析一维数据\2.png)

​```
gdp_ch.plot()
​```

![](使用NumPy和Pandas分析一维数据\3.png)
`````

