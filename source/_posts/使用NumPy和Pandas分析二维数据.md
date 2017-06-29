---
title: 使用NumPy和Pandas分析二维数据
date: 2017-04-23 08:34:45
tags:
categories: 数据分析
---

#### 课程内容概述

- 了解NumPy和Pandas更多特征
- 用两种库分析二维数据

<!---more--->

#### 数据准备及提出问题

从[nyc-subway-weather.csv](https://d17h27t6h515a5.cloudfront.net/topher/2016/October/57fdf8f7_nyc-subway-weather/nyc-subway-weather.csv)下载纽约地铁客流量与天气数据。针对数据集提出一些问题，如：

- 哪些变量与地铁客流量相关？
- 哪个车站客流量最多？各车间之间有什么差异？
- 客流量模式（如客流高峰期时段，工作日和周末的差别等
- 天气对客流量有什么影响？
- 单独研究天气数据，如一段时间内气温走势，不同城市间天气差异等。

#### NumPy和Pandas中的二维数据

二维数据的表示：

Python: list of lists

 NumPy: 2-Dimensional array （更简要，容易理解）

Pandas: Data Frame  (功能更多)

创建一个 2D arrays和创建一个 array of arrays的区别：

- 2D arrays更节省内存
- 获取元素语法不同，2D arrays使用l类似a[1,3]来获取元素而非a\[1][3],且可以将行、列或两者都表示为slice切片
- mean、std等函数，可以用于整个2D arrays

2D arrays slice:

```
import numpy as np
ridership = np.array([
    [   0,    0,    2,    5,    0],
    [1478, 3877, 3674, 2328, 2539],
    [1613, 4088, 3991, 6461, 2691],
    [1560, 3392, 3826, 4787, 2613],
    [1608, 4802, 3932, 4477, 2705],
    [1576, 3933, 3909, 4979, 2685],
    [  95,  229,  255,  496,  201],
    [   2,    0,    1,   27,    0],
    [1438, 3785, 3589, 4174, 2215],
    [1342, 4043, 4009, 4665, 3033]
])
print ridership[1, 3]
print ridership[1:3, 3:5]
print ridership[1, :]
```

Out:

```
2328
[[2328 2539]
 [6461 2691]]
[1478 3877 3674 2328 2539]
```

行列向量运算：

```
print ridership[0, :] + ridership[1, :]
print ridership[:, 0] + ridership[:, 1]
Out:
[1478 3877 3676 2333 2539]
[   0 5355 5701 4952 6410 5509  324    2 5223 5385]
```

array的向量运算：

```
a = np.array([[1, 2, 3], [4, 5, 6], [7, 8, 9]])
b = np.array([[1, 1, 1], [2, 2, 2], [3, 3, 3]])
print a + b
Out:
[[ 2  3  4]
 [ 6  7  8]
 [10 11 12]]
```

练习：用2D arrays表示不同日期、车站的地铁客流量。编写函数，使其能够找出第一天客流量最多的车站，并找出这个车站每天的平均乘客数。同时，返回各个车站每天的平均乘客数，进行比较。

```
def mean_riders_for_max_station(ridership):
    fist_day_data=ridership[0:1,:]
    sta=fist_day_data.argmax()
    all_data=ridership[:,sta]
    overall_mean = ridership.mean()
    mean_for_max = all_data.mean() 
    return (overall_mean, mean_for_max)
mean_riders_for_max_station(ridership)
```

分析得出，第一天客流量多的车站，平均客流量也高于所有车站客流量的平均值。

#### NumPy Axis（轴）

应用Axis，可以以行或列为单位进行运算。

2D array的axis数值通常为0或1

axis=0，在每行中进行运算；axis=1，在每列中进行运算；

如：

```
a = np.array([
    [1, 2, 3],
    [4, 5, 6],
    [7, 8, 9]
])
print a.sum()
print a.sum(axis=0)
print a.sum(axis=1)
Out:
45
[12 15 18]
[ 6 15 24]
```

练习：找出每一个地铁站所有日期内的客流均值，以及每个地铁站客流量均值中的最高和最低值。

```
station_riders=ridership.mean(axis=0)
print station_riders.max() 
print station_riders.min()
```

#### 关于NumPy和Pandas的数据类型

NumPy 2D array具有一个dtype ，数组中每一个元素都应该属于同一类型。

故2D array不便于表现csv文件的内容，如果创建一个包含字符型数据的array 则array中所有的元素将被转换为字符串，而无法进行计算。

Pandas DataFrame——Pandas中的二维数据结构

优势：

- 每一列可以为不同的数值类型
- 拥有index （类似Pandas理论，每行都有一个索引值，每列都有一个名称）

在创建DataFrame时，可以直接输入一个字典，使列名称映射该列的数值列表。

如：

```
import pandas as pd
enrollments_df=pd.DataFrame({
    'account_key':[448,448,448,448,448],
    'status':['canceled','canceled','canceled','canceled','current'],
    'join_data':['2014-11-10','2014-11-5','2015-1-27 ','2014-11-10','2015-3-10'],
    'days_to_cancel':[65,5,0,0,np.nan],
    'is_udacity':[True,True,True,True,True]
})
enrollments_df
```

|      | account_key | days_to_cancel | is_udacity | join_data  | status   |
| ---- | ----------- | -------------- | ---------- | ---------- | -------- |
| 0    | 448         | 65.0           | True       | 2014-11-10 | canceled |
| 1    | 448         | 5.0            | True       | 2014-11-5  | canceled |
| 2    | 448         | 0.0            | True       | 2015-1-27  | canceled |
| 3    | 448         | 0.0            | True       | 2014-11-10 | canceled |
| 4    | 448         | NaN            | True       | 2015-3-10  | current  |

DataFrame.mean() 计算每一列的平均值（仅计算数值、布尔值而忽略非数值），如：

```
enrollments_df.mean()
Out:
account_key       448.0
days_to_cancel     17.5
is_udacity          1.0
dtype: float64
```

以指定index和column名称的方式创建一个DataFrame：

```
ridership_df = pd.DataFrame(
    data=[[   0,    0,    2,    5,    0],
          [1478, 3877, 3674, 2328, 2539],
          [1613, 4088, 3991, 6461, 2691],
          [1560, 3392, 3826, 4787, 2613],
          [1608, 4802, 3932, 4477, 2705],
          [1576, 3933, 3909, 4979, 2685],
          [  95,  229,  255,  496,  201],
          [   2,    0,    1,   27,    0],
          [1438, 3785, 3589, 4174, 2215],
          [1342, 4043, 4009, 4665, 3033]],
    index=['05-01-11', '05-02-11', '05-03-11', '05-04-11', '05-05-11',
           '05-06-11', '05-07-11', '05-08-11', '05-09-11', '05-10-11'],
    columns=['R003', 'R004', 'R005', 'R006', 'R007']
)
```

|          | R003 | R004 | R005 | R006 | R007 |
| -------- | ---- | ---- | ---- | ---- | ---- |
| 05-01-11 | 0    | 0    | 2    | 5    | 0    |
| 05-02-11 | 1478 | 3877 | 3674 | 2328 | 2539 |
| 05-03-11 | 1613 | 4088 | 3991 | 6461 | 2691 |
| 05-04-11 | 1560 | 3392 | 3826 | 4787 | 2613 |
| 05-05-11 | 1608 | 4802 | 3932 | 4477 | 2705 |
| 05-06-11 | 1576 | 3933 | 3909 | 4979 | 2685 |
| 05-07-11 | 95   | 229  | 255  | 496  | 201  |
| 05-08-11 | 2    | 0    | 1    | 27   | 0    |
| 05-09-11 | 1438 | 3785 | 3589 | 4174 | 2215 |
| 05-10-11 | 1342 | 4043 | 4009 | 4665 | 3033 |

从DataFrame中选取一行数据：

```
ridership_df.iloc[0]
ridership_df.loc['05-05-11']
```

从DataFrame中选取一列数据：

```
ridership_df['R003']
```

从DataFrame中选取特定位置的某个数据：

```
ridership_df.iloc[1, 3]
```

数据切片展示：

```
ridership_df.iloc[1:4]
ridership_df[['R003', 'R005']]
```

`DataFrame.sum()` 计算每一行的和（默认axis=0）

`DataFrame.sum(axis=1)` 计算每一列的和

如果要计算所有值的和（均值、标准差等），需要通过`DataFrame.values` 转化为array后计算，如：

```
df = pd.DataFrame({'A': [0, 1, 2], 'B': [3, 4, 5]})
print df.sum()
print df.sum(axis=1)
print df.values.sum()
Out:
A     3
B    12
dtype: int64
0    3
1    5
2    7
dtype: int64
15
```

练习：使用DataFrame重写第一个练习中的函数找出第一天客流量最多的车站，和这个车站每天的平均乘客数。以及所有车站的平均乘客数。

```
def mean_riders_for_max_station(ridership):
    sta=ridership.iloc[0].argmax()
    overall_mean = ridership.values.mean()
    mean_for_max = ridership[sta].mean() 
    return (overall_mean, mean_for_max)
mean_riders_for_max_station(ridership_df)
```

#### 将数据加载到DataFrame

DataFrame可以有效表示csv文件内容。

所以可以使用Pandas的`read_csv` 函数将csv文件加载为DataFrame。

如：

```
subway_df=pd.read_csv('/home/milhaven1733/nyc-subway-weather.csv')
```

用`DataFrame.head()` 函数打印前几行：

```
subway_df.head(3)
```

![](使用NumPy和Pandas分析二维数据\1.png)

使用`DataFrame.describe()` 查看每一列的统计数据

```
subway_df.describe()
```

![](使用NumPy和Pandas分析二维数据\2.png)

#### 计算x相关系数

**皮尔逊积矩相关系数** (Pearson Correlation Coefficient，又称Pearson's r）用来衡量两组变量之间的相关程度。

计算步骤：

- 将每组变量标准化（归一化，每个数值减去均值再除以标准偏差）
- 计算每组标准化后的变量的乘积
- 计算乘积的均值r

即 r=average of （x  in std units）*（y  in std units）

作用：

标准化之后，二维空间被分为四个象限。

一三象限表示两个值均高于或低于平均值，说明一个变量随另一个变量增加而增加，乘积为正——正相关关系

二四象限表示一个值高于、一个值低于平均值，说明一个变量随另一个变量增加而减少，乘积为负——负相关关系

所以，r为正，表示大部分点位于一三象限，两组变量为正相关，反之为负相关。

r的范围：[-1,1],越接近零说明相关程度越小，绝对值越接近1，说明相关程度越大。

注意：

在标准化两变量时，必须使用未更正的标准偏差，即必须加入参数ddof=0

默认情况下，Pandas 的 `std()` 函数使用贝塞耳校正系数来计算标准偏差。调用 `std(ddof=0)` 可以禁止使用贝塞耳校正系数。

练习：编写函数，计算两变量的相关系数

```
def correlation(x, y):
    x_sd=(x-x.mean())/x.std(ddof=0)
    y_sd=(y-y.mean())/y.std(ddof=0)
    mul_sd=x_sd*y_sd
    return mul_sd.mean()
```

实际上，使用NumPy的`corrcoef()` 函数可以直接计算两变量的相关系数。

[这个网站](http://rpsychologist.com/d3/correlation/) 可以通过移动滑块查看不同相关系数的数据模型。

#### Pandas Axis Name

在Pandas中，可以用`axis='index'` `axis='columns'` 来表示axis

但两个参数值容易混淆。

当使用`axis='index'` 时，以索引列为轴，计算的是每一行的某统计量。

当使用`axis='columns'` 时，以列名称这一行为轴，计算的是每一列的某统计量。

#### DataFrame向量运算和偏移量

DataFrame支持向量运算。

以column_name和index作为向量配对条件，如：

```
df1 = pd.DataFrame({'a': [1, 2, 3], 'b': [4, 5, 6], 'c': [7, 8, 9]})
df2 = pd.DataFrame({'d': [10, 20, 30], 'c': [40, 50, 60], 'b': [70, 80, 90]})
print df1 + df2
Outs:
    a   b   c   d
0 NaN  74  47 NaN
1 NaN  85  58 NaN
2 NaN  96  69 NaN
```

```
df1 = pd.DataFrame({'a': [1, 2, 3], 'b': [4, 5, 6], 'c': [7, 8, 9]},
                   index=['row1', 'row2', 'row3'])
df2 = pd.DataFrame({'a': [10, 20, 30], 'b': [40, 50, 60], 'c': [70, 80, 90]},
                   index=['row4', 'row3', 'row2'])
print df1 + df2
Outs:
         a     b     c
row1   NaN   NaN   NaN
row2  32.0  65.0  98.0
row3  23.0  56.0  89.0
row4   NaN   NaN   NaN
```

练习：现在有某地铁站运营截止到每个时段的总进站和出站人数的数据集，由此计算每个时段内的进站和出站人数。

tips：`DataFrame.shift()` 函数——按照偏移量移动索引值

`DataFrame.shift`(*periods=1*, *freq=None*, *axis=0*)

**periods** : int

> Number of periods to move, can be positive or negative

示例：

```
df1 = pd.DataFrame({'a': [1, 2, 3], 'b': [4, 5, 6], 'c': [7, 8, 9]},
                   index=['row1', 'row2', 'row3'])
print df1
print df1.shift(1)
Out:
      a  b  c
row1  1  4  7
row2  2  5  8
row3  3  6  9
        a    b    c
row1  NaN  NaN  NaN
row2  1.0  4.0  7.0
row3  2.0  5.0  8.0
```

以index为轴偏移一个单位。

所以，以原始DataFrame减去偏移一个单位的DataFrame，可以得到每小时内进出站的人数：

```
def get_hourly_entries_and_exits(entries_and_exits):
    return (entries_and_exits-entries_and_exits.shift(1)).dropna() #可以舍弃NaN值
```

#### DataFrame.applymap()

与series类似，DataFrame支持apply函数，对DataFrame中的每一个数值调用apply中的函数，得到一个新的DataFrame。如将数字成绩转化为字母等级：

```
grades_df = pd.DataFrame(
    data={'exam1': [43, 81, 78, 75, 89, 70, 91, 65, 98, 87],
          'exam2': [24, 63, 56, 56, 67, 51, 79, 46, 72, 60]},
    index=['Andre', 'Barry', 'Chris', 'Dan', 'Emilio', 
           'Fred', 'Greta', 'Humbert', 'Ivan', 'James']
)
def convert_grade(grades):
    if grades>=90:
        return 'A'
    elif grades>=80:
        return 'B'
    elif grades>=70:
        return 'C'    
    elif grades>=60:
        return 'D'   
    else:
        return 'F'
def convert_grades(grades):
    return grades.applymap(convert_grade)
convert_grades(grades_df)
```

#### DataFrame.apply()

在DataFrame中，apply() 和applymap() 调用原理不同。

apply()在调用时，相当于将DataFrame的每一列单独作为一个series，调用针对这个series的函数进行运算。

即调用的函数针对于DataFrame的每一列运算并返回一个新的DataFrame列。

如：若字母等级成绩取决于某个成绩在这一科所有成绩中的占比，那么直接应用applymap(),对DataFrame中每个元素运算就无法完成转化，必须以每一列为对象进行运算：

```
def convert_grades_curve(exam_grades):
    return pd.qcut(exam_grades,
                   [0, 0.1, 0.2, 0.5, 0.8, 1],
                   labels=['F', 'D', 'C', 'B', 'A'])                   
convert_grades_curve(grades_df['exam1'])  #对某一列调用convert_grades_curve函数
grades_df.apply(convert_grades_curve)	#对DataFrame的每一列调用convert_grades_curve函数
```

> pandas.qcut() 函数：
>
> `pandas.qcut`(*x*, *q*, *labels=None*, *retbins=False*, *precision=3*)
>
> **x** : ndarray or Series
>
> **q** : integer or array of quantiles
>
> > Number of quantiles. 10 for deciles, 4 for quartiles, etc. Alternately array of quantiles, e.g. [0, .25, .5, .75, 1.] for quartiles
>
> **labels** : array or boolean, default None
>
> > Used as labels for the resulting bins. Must be of the same length as the resulting bins. If False, return only integer indicators of the bins.
>
> 即利用分位数划分Series，返回数值位于哪个区间上，并可以使用labels对区间进行逐个命名。

练习：对DataFrame的每一列进行标准化。

```
def standardize(df):
    return (df-df.mean())/df.std(ddof = 0) #不使用校正系数
print grades_df.apply(standardize)
```

DataFrame.apply()还可以对每一列调用函数 ，最后将每一列的结果生成一个series返回，如：

```
df = pd.DataFrame({
    'a': [4, 5, 3, 1, 2],
    'b': [20, 10, 40, 50, 30],
    'c': [25, 20, 5, 15, 10]
})
print df.apply(np.mean)
print df.apply(np.max)
```

Out:

```
a     3.0
b    30.0
c    15.0
dtype: float64
a     5
b    50
c    25
dtype: int64
```

练习：找出一个DataFrame中，每列第二大的数值。

```
def second_largest_in_column(column):
    sort_column=column.sort_values(ascending=False) #对column降序排列
    return sort_column.iloc[1]  #返回第二大的值
def second_largest(df):
	return df.apply(second_largest_in_column)    #对DataFrame的每一列调用函数
second_largest(df)
```

#### DataFrame与series	相加

Pandas中，DataFrame可以和Series相加。

列数相同：

```
s = pd.Series([1, 2, 3, 4])
df = pd.DataFrame({0: [10], 1: [20], 2: [30], 3: [40]})
print df
print '' 
print df + s
```

Out:

```
    0   1   2   3
0  10  20  30  40

    0   1   2   3
0  11  22  33  44
```

列数不同：

```
s = pd.Series([1, 2, 3, 4])
df = pd.DataFrame({0: [10, 20, 30, 40]})
print df
print '' 
print df + s
```

Out:

```
    0
0  10
1  20
2  30
3  40

    0   1   2   3
0  11 NaN NaN NaN
1  21 NaN NaN NaN
2  31 NaN NaN NaN
3  41 NaN NaN NaN
```

索引与默认索引相同：

```
s = pd.Series([1, 2, 3, 4])
df = pd.DataFrame({
    0: [10, 20, 30, 40],
    1: [50, 60, 70, 80],
    2: [90, 100, 110, 120],
    3: [130, 140, 150, 160]
})
print df + s
```

Out:

```
    0   1    2    3
0  11  52   93  134
1  21  62  103  144
2  31  72  113  154
3  41  82  123  164
```

指定索引相同：

```
s = pd.Series([1, 2, 3, 4], index=['a', 'b', 'c', 'd'])
df = pd.DataFrame({
    'a': [10, 20, 30, 40],
    'b': [50, 60, 70, 80],
    'c': [90, 100, 110, 120],
    'd': [130, 140, 150, 160]
})
print df + s
```

Out:

```
    a   b    c    d
0  11  52   93  134
1  21  62  103  144
2  31  72  113  154
3  41  82  123  164
```

索引不同：

```
s = pd.Series([1, 2, 3, 4])
df = pd.DataFrame({
    'a': [10, 20, 30, 40],
    'b': [50, 60, 70, 80],
    'c': [90, 100, 110, 120],
    'd': [130, 140, 150, 160]
})
print df + s
```

Out：

```
    0   1   2   3   a   b   c   d
0 NaN NaN NaN NaN NaN NaN NaN NaN
1 NaN NaN NaN NaN NaN NaN NaN NaN
2 NaN NaN NaN NaN NaN NaN NaN NaN
3 NaN NaN NaN NaN NaN NaN NaN NaN
```

#### 标准化行

运用向量运算对DataFrame的每一行进行标准化：

创建DataFrame:

```
grades_df = pd.DataFrame(
    data={'exam1': [43, 81, 78, 75, 89, 70, 91, 65, 98, 87],
          'exam2': [24, 63, 56, 56, 67, 51, 79, 46, 72, 60]},
    index=['Andre', 'Barry', 'Chris', 'Dan', 'Emilio', 
           'Fred', 'Greta', 'Humbert', 'Ivan', 'James']
)
```

设置`axis='columns'` ,计算每一行的均值：

```
df_mean=grades_df.mean(axis='columns')
```

```
Andre      33.5
Barry      72.0
Chris      67.0
Dan        65.5
Emilio     78.0
Fred       60.5
Greta      85.0
Humbert    55.5
Ivan       85.0
James      73.5
```

同理，计算每一行的标准偏差：

```
df_std=grades_df.std(ddof=0,axis='columns')
```

```
Andre       9.5
Barry       9.0
Chris      11.0
Dan         9.5
Emilio     11.0
Fred        9.5
Greta       6.0
Humbert     9.5
Ivan       13.0
James      13.5
```

但由于grades_df的column与df_mean、df_std 的不同，不能直接相减或相除，必须在运算时指定`axis='index'`,即在每一行上进行运算：

```
df_diffs=grades_df.sub(df_mean,axis='index')
print df_diffs.div(df_std,axis='index')
```

```
         exam1  exam2
Andre      1.0   -1.0
Barry      1.0   -1.0
Chris      1.0   -1.0
Dan        1.0   -1.0
Emilio     1.0   -1.0
Fred       1.0   -1.0
Greta      1.0   -1.0
Humbert    1.0   -1.0
Ivan       1.0   -1.0
James      1.0   -1.0
```

得到每一行的标准化量。

#### Pandas groupby()

DataFrame具有分组功能。

如可以将地铁客流量数据按照"hour"字段分组，计算分组后每组数据的均值，在对各个均值进行分析。

通过一个实例来查看分组统计的步骤：

首先创建一个DataFrame：

```
enrollments_df=pd.DataFrame({
    'account_key':['1200','1200','1200','1200','1200','1200','1200',
    '1175','1175','1175','1175','1175','1175','1175'],
    'utc_date':['2015-03-04','2015-03-05','2015-03-06','2015-03-07','2015-03-08',
    '2015-03-09','2015-03-10','2015-04-02','2015-04-03','2015-04-04','2015-04-05',
    '2015-04-06','2015-04-07','2015-04-08'],
    'total_minutes_visited':[114.9,43.4,187.8,150.1,19.6,0,8.8,2.7,0,0,0,0,0,0]
})
```

按照某个字段来分组：

创建了一个特殊的自定义对象：

```
enrollments_df.groupby('account_key')
Out:
<pandas.core.groupby.DataFrameGroupBy object at 0x7f924f702510>
```

接下来，对创建的groupby对象使用.sum()函数

**注意这里的sum是groupby对象自身的函数，groupby具有很多内置函数，也支持apply**

```
enrollments_df.groupby('account_key').sum()
```

返回一个列表：

|             | total_minutes_visited |
| ----------- | --------------------- |
| account_key |                       |
| 1175        | 2.7                   |
| 1200        | 524.6                 |

由于'utc_date'字段无法计算总和，故结果不含该列。

最后，选取结果中的一列：

```
enrollments_df.groupby('account_key').sum()['total_minutes_visited']
```

返回一个series：

```
account_key
1175      2.7
1200    524.6
Name: total_minutes_visited, dtype: float64
```

groupby示例二：

```
values = np.array([1, 3, 2, 4, 1, 6, 4])
example_df = pd.DataFrame({
    'value': values,
    'even': values % 2 == 0,
    'above_three': values > 3 
}, index=['a', 'b', 'c', 'd', 'e', 'f', 'g'])
print example_df
print ''
grouped_data = example_df.groupby('even')
print grouped_data.sum()
print ''
print grouped_data.sum()['value']
print ''
print grouped_data['value'].sum()
```

Out:

```
  above_three   even  value
a       False  False      1
b       False  False      3
c       False   True      2
d        True   True      4
e       False  False      1
f        True   True      6
g        True   True      4

       above_three  value
even                     
False          0.0      5
True           3.0     16

even
False     5
True     16
Name: value, dtype: int64

even
False     5
True     16
Name: value, dtype: int64
```

练习：以某个统计量对地铁数据分组，求分组后各组，各数值统计量的均值，选取其中一个有具体意义的进行绘图

如：以日期为单位（字段："day_week"）

分组，计算其与统计量均值：

```
subway_df.groupby('day_week')
subway_df.groupby('day_week').mean()
```

选取'ENTRIESn_hourly'字段查看每小时进入地铁站的人数均值：

```
ridership_weekday=subway_df.groupby('hour').mean()['ENTRIESn_hourly']
```

得到以下结果：

```
day_week
0    1825.264907
1    2164.836433
2    2297.097957
3    2317.072379
4    2277.372294
5    1383.901479
6    1066.436106
Name: ENTRIESn_hourly, dtype: float64
```

绘图：

```
%pylab inline
import seaborn as sns
ridership_weekday.plot()
```

![](使用NumPy和Pandas分析二维数据\3.png)

周末客流量小于平日，图像比较合理。

#### groupby  apply()

对每个分组内的某一列值调用指定函数。

如：

```
def second_largest(xs):
    sorted_xs = xs.sort_values(inplace=False, ascending=False)
    return sorted_xs.iloc[1]
grouped_data = example_df.groupby('even')
print grouped_data['value'].apply(second_largest)
```

分组后，False组的value包含[1,3,1],True组包含[2,4,6,4],分别对这两组自定义对象调用 second_largest函数：

```
even
False    1
True     4
Name: value, dtype: int64
```

又如：

```
def standardize(xs):
    return (xs - xs.mean()) / xs.std()
grouped_data = example_df.groupby('even')
print grouped_data['value'].apply(standardize)
```

通过'even'分组，在每组的'value'字段上调用standardize()进行组内数据的标准化

练习：对地铁数据按照站台分组，使用apply()调用之前所写的get_hourly_entries_and_exits函数，计算每个时段内的出站和进站人数。

```
ridership_df.groupby('UNIT')[['ENTRIESn','EXITSn']].apply(get_hourly_entries_and_exits)
```

用'UNIT'分组，指定仅对于['ENTRIESn','EXITSn']两个字段的分组值调用get_hourly_entries_and_exits()函数

#### DataFrame merge()

两个不同的DataFrame可以进行合并操作，类似于SQL中的JOIN操作。

语法：DF1.merge(DF2,on="",how="")

on="",指定进行匹配的字段，相当于SQL的连接条件

how=""指定连接类型，可选值有"inner","left","right",相当于SQL的内，左外，右外连接。

使用外连接时，没有相应数据的字段显示为NaN

练习，合并地铁和天气数据。（要求仅保存两个DataFrame中有相同记录字段的行）

```
subway_df = pd.DataFrame({
    'UNIT': ['R003', 'R003', 'R003', 'R003', 'R003', 'R004', 'R004', 'R004',
             'R004', 'R004'],
    'DATEn': ['05-01-11', '05-02-11', '05-03-11', '05-04-11', '05-05-11',
              '05-01-11', '05-02-11', '05-03-11', '05-04-11', '05-05-11'],
    'hour': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    'ENTRIESn': [ 4388333,  4388348,  4389885,  4391507,  4393043, 14656120,
                 14656174, 14660126, 14664247, 14668301],
    'EXITSn': [ 2911002,  2911036,  2912127,  2913223,  2914284, 14451774,
               14451851, 14454734, 14457780, 14460818],
    'latitude': [ 40.689945,  40.689945,  40.689945,  40.689945,  40.689945,
                  40.69132 ,  40.69132 ,  40.69132 ,  40.69132 ,  40.69132 ],
    'longitude': [-73.872564, -73.872564, -73.872564, -73.872564, -73.872564,
                  -73.867135, -73.867135, -73.867135, -73.867135, -73.867135]
})
weather_df = pd.DataFrame({
    'DATEn': ['05-01-11', '05-01-11', '05-02-11', '05-02-11', '05-03-11',
              '05-03-11', '05-04-11', '05-04-11', '05-05-11', '05-05-11'],
    'hour': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    'latitude': [ 40.689945,  40.69132 ,  40.689945,  40.69132 ,  40.689945,
                  40.69132 ,  40.689945,  40.69132 ,  40.689945,  40.69132 ],
    'longitude': [-73.872564, -73.867135, -73.872564, -73.867135, -73.872564,
                  -73.867135, -73.872564, -73.867135, -73.872564, -73.867135],
    'pressurei': [ 30.24,  30.24,  30.32,  30.32,  30.14,  30.14,  29.98,  29.98,
                   30.01,  30.01],
    'fog': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    'rain': [0, 0, 0, 0, 0, 0, 0, 0, 0, 0],
    'tempi': [ 52. ,  52. ,  48.9,  48.9,  54. ,  54. ,  57.2,  57.2,  48.9,  48.9],
    'wspdi': [  8.1,   8.1,   6.9,   6.9,   3.5,   3.5,  15. ,  15. ,  15. ,  15. ]
})
```

首先，查看两个DataFrame的结构：

```
subway_df.head()
```

|      | DATEn    | ENTRIESn | EXITSn  | UNIT | hour | latitude  | longitude  |
| ---- | -------- | -------- | ------- | ---- | ---- | --------- | ---------- |
| 0    | 05-01-11 | 4388333  | 2911002 | R003 | 0    | 40.689945 | -73.872564 |
| 1    | 05-02-11 | 4388348  | 2911036 | R003 | 0    | 40.689945 | -73.872564 |
| 2    | 05-03-11 | 4389885  | 2912127 | R003 | 0    | 40.689945 | -73.872564 |
| 3    | 05-04-11 | 4391507  | 2913223 | R003 | 0    | 40.689945 | -73.872564 |
| 4    | 05-05-11 | 4393043  | 2914284 | R003 | 0    | 40.689945 | -73.872564 |

```
weather_df.head()
```

|      | DATEn    | fog  | hour | latitude  | longitude  | pressurei | rain | tempi | wspdi |
| ---- | -------- | ---- | ---- | --------- | ---------- | --------- | ---- | ----- | ----- |
| 0    | 05-01-11 | 0    | 0    | 40.689945 | -73.872564 | 30.24     | 0    | 52.0  | 8.1   |
| 1    | 05-01-11 | 0    | 0    | 40.691320 | -73.867135 | 30.24     | 0    | 52.0  | 8.1   |
| 2    | 05-02-11 | 0    | 0    | 40.689945 | -73.872564 | 30.32     | 0    | 48.9  | 6.9   |
| 3    | 05-02-11 | 0    | 0    | 40.691320 | -73.867135 | 30.32     | 0    | 48.9  | 6.9   |
| 4    | 05-03-11 | 0    | 0    | 40.689945 | -73.872564 | 30.14     | 0    | 54.0  | 3.5   |

可以进行连接的字段有'DATEn','hour','latitude','longitude'，所以进行连接：

```
 subway_df.merge(weather_df,
                on=['DATEn','hour','latitude','longitude'],
                how="inner")
```

若两DataFrame中，内容相同的字段column名称不同，可以分别注明left_on="";right_on="".

如：（需匹配的字段相对应）

```
subway_df.merge(weather_df,
                left_on=['DATEn','hour','latitude','longitude'],
                right_on=['DATEn','hour','latitude','longitude'],
                how="inner")
```

#### 使用DataFrame绘制图形

DataFrame 也像 Pandas Series 一样拥有 plot() 方法。如果 `df` 是 DataFrame，那么 `df.plot()` 将生成线条图，其中不同颜色的每条线代表 DataFrame 中的一个变量。

但是对于更复杂的图形，通常需要直接使用 matplotlib。

每日平均客流量折线图：

```
data_by_date=subway_df.groupby(['DATEn'],as_index=False).mean()
data_by_date['EXITSn_hourly'].plot()
```

![](使用NumPy和Pandas分析二维数据\4.png)

以经纬度作为 x 和 y 轴、客流量作为气泡大小的地铁站散点图:

以经纬度分组：（as_index=False表示使用来分组的值仍作为column）

```
data_by_location=subway_df.groupby(['latitude','longitude'],as_index=False).mean()
```

绘图：

```
%pylab inline
import matplotlib.pyplot as plt
import seaborn as sns
scaled=(data_by_location['ENTRIESn_hourly']/data_by_location['ENTRIESn_hourly'].std()*10)
plt.scatter(data_by_location['latitude'],data_by_location['longitude'],s=scaled)
```

参数s用来设置每个散点的气泡大小，可以设置为该点对应平均客流量。（除以标准偏差deng 操作用来缩放气泡大小）

![](使用NumPy和Pandas分析二维数据\5.png)

大体可以根据气泡大小，经纬度来判断客流量大的地区。