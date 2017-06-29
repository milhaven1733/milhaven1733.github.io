---
title: '《利用Python进行数据分析》范例数据集:usagov_bitly_data'
date: 2017-05-03 16:56:38
tags:
categories: 数据分析
---

读取生成.gov或.mil短链接的用户数据文本文件（文本中每行的为JSON）

```python
path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/usagov_bitly_data2012-03-16-1331923249.txt'
```

```python
open(path).readline()
```

Outs:

```
'{ "a": "Mozilla\\/5.0 (Windows NT 6.1; WOW64) AppleWebKit\\/535.11 (KHTML, like Gecko) Chrome\\/17.0.963.78 Safari\\/535.11", "c": "US", "nk": 1, "tz": "America\\/New_York", "gr": "MA", "g": "A6qOVH", "h": "wfLQtf", "l": "orofrog", "al": "en-US,en;q=0.8", "hh": "1.usa.gov", "r": "http:\\/\\/www.facebook.com\\/l\\/7AQEFzjSi\\/1.usa.gov\\/wfLQtf", "u": "http:\\/\\/www.ncbi.nlm.nih.gov\\/pubmed\\/22415991", "t": 1331923247, "hc": 1331822918, "cy": "Danvers", "ll": [ 42.576698, -70.954903 ] }\n'
```

用Python的JSON模块将JSON字符串转换为Python字典对象

```python
import json
```

```python
records=[json.loads(line) for line in open(path)]
```

<!---more--->

查看list中的第一个字典元素：

```python
records[0]
```

Outs:

```
{u'a': u'Mozilla/5.0 (Windows NT 6.1; WOW64) AppleWebKit/535.11 (KHTML, like Gecko) Chrome/17.0.963.78 Safari/535.11',
 u'al': u'en-US,en;q=0.8',
 u'c': u'US',
 u'cy': u'Danvers',
 u'g': u'A6qOVH',
 u'gr': u'MA',
 u'h': u'wfLQtf',
 u'hc': 1331822918,
 u'hh': u'1.usa.gov',
 u'l': u'orofrog',
 u'll': [42.576698, -70.954903],
 u'nk': 1,
 u'r': u'http://www.facebook.com/l/7AQEFzjSi/1.usa.gov/wfLQtf',
 u't': 1331923247,
 u'tz': u'America/New_York',
 u'u': u'http://www.ncbi.nlm.nih.gov/pubmed/22415991'}
```

统计数据集中的时区（tz字段）信息：

用列表推导式取出tz字段：

```python
time_zones=[rec['tz'] for rec in records if 'tz' in rec]
time_zones[:5]
```

Outs:

```
[u'America/New_York',
 u'America/Denver',
 u'America/New_York',
 u'America/Sao_Paulo',
 u'America/New_York']
```

利用Python标准库中的collections.Counter类，对time_zones列表进行分类计数，取出出现频次最多的十个。

```python
from collections import Counter
counts=Counter(time_zones)
counts.most_common(10)
```

Outs:

```
[(u'America/New_York', 1251),
 (u'', 521),
 (u'America/Chicago', 400),
 (u'America/Los_Angeles', 382),
 (u'America/Denver', 191),
 (u'Europe/London', 74),
 (u'Asia/Tokyo', 37),
 (u'Pacific/Honolulu', 36),
 (u'Europe/Madrid', 35),
 (u'America/Sao_Paulo', 33)]
```

用Pandas对时区计数：

```python
import pandas as pd
frame=pd.DataFrame(records)
```

```
frame.head()
```

将tz字段取出为Series对象。利用value_counts方法进行统计：

```python
tz=frame['tz']
tz.value_counts()[:5]
```

Outs:

```
America/New_York       1251
                        521
America/Chicago         400
America/Los_Angeles     382
America/Denver          191
Name: tz, dtype: int64
```

替换NA值和空字符串：

```python
clean_tz=frame['tz'].fillna('Missing')
clean_tz[clean_tz=='']='Unknow'
tz_counts=clean_tz.value_counts()[:10]
tz_counts
```

Outs:

```
America/New_York       1251
Unknow                  521
America/Chicago         400
America/Los_Angeles     382
America/Denver          191
Missing                 120
Europe/London            74
Asia/Tokyo               37
Pacific/Honolulu         36
Europe/Madrid            35
Name: tz, dtype: int64
```

为以上整理好的数据创建一张水平条形图：

```python
%matplotlib inline
import seaborn as sns
tz_counts.plot(kind='barh',rot=0)
```

![png](《利用Python进行数据分析》范例数据集-usagov-bitly-data\output_21_1.png)

通过a字段，利用agent信息对数据集进行更细致的加工：

舍弃NA值，对字符串分段，取出浏览器信息，创建一个Series：

```python
agent=pd.Series([x.split()[0] for x in frame.a.dropna()])
```

```python
agent.value_counts()[:5]
```

Outs:

```
Mozilla/5.0               2594
Mozilla/4.0                601
GoogleMaps/RochesterNY     121
Opera/9.80                  34
TEST_INTERNET_AGENT         24
dtype: int64
```

按用户是否使用Windows系统对时区信息进行分解：

首先分离出frame中a字段非空的数据：

```python
new_frame=frame[frame.a.notnull()]
```

判断a字段是否包含“Windows”：

```python
new_frame.a.str.contains('Windows').head()
```

Outs:

```
0     True
1    False
2     True
3    False
4     True
Name: a, dtype: bool
```

添加一个os字段：

```python
import numpy as np
new_frame['os']=np.where(new_frame.a.str.contains('Windows'),'Windows','Not Windows')
```

按照‘tz’和‘os’字段分组：

```python
by_tz_os=new_frame.groupby(['tz','os'])
```

通过size对分组结果进行计数,利用unstack重塑结果，填充NA值：

```python
aggent=by_tz_os.size().unstack().fillna(0)
```

为了选取最常出现的时区，先对统计结果构造一个整理顺序（按升序排列）的索引：

```python
index=aggent.sum(1).argsort()
```

按照索引对统计结果排序，选取最后10行：

```python
aggent_count=aggent.take(index)[-10:]
aggent_count
```

| os                  | Not Windows | Windows |
| ------------------- | ----------- | ------- |
| tz                  |             |         |
| America/Sao_Paulo   | 13.0        | 20.0    |
| Europe/Madrid       | 16.0        | 19.0    |
| Pacific/Honolulu    | 0.0         | 36.0    |
| Asia/Tokyo          | 2.0         | 35.0    |
| Europe/London       | 43.0        | 31.0    |
| America/Denver      | 132.0       | 59.0    |
| America/Los_Angeles | 130.0       | 252.0   |
| America/Chicago     | 115.0       | 285.0   |
|                     | 245.0       | 276.0   |
| America/New_York    | 339.0       | 912.0   |

生成堆积条形图：

```python
aggent_count.plot(kind='bar',stacked=True)
```

![png](《利用Python进行数据分析》范例数据集-usagov-bitly-data\output_42_1.png)

```python
aggent_count.plot(kind='barh',stacked=True)
```

![png](《利用Python进行数据分析》范例数据集-usagov-bitly-data\output_43_1.png)

将使用widows和不使用的用户数量表示为相对比例：

```python
aggent_normal=aggent_count.div(aggent_count.sum(1),axis=0)
```

```python
aggent_normal
```

| os                  | Not Windows | Windows  |
| ------------------- | ----------- | -------- |
| tz                  |             |          |
| America/Sao_Paulo   | 0.393939    | 0.606061 |
| Europe/Madrid       | 0.457143    | 0.542857 |
| Pacific/Honolulu    | 0.000000    | 1.000000 |
| Asia/Tokyo          | 0.054054    | 0.945946 |
| Europe/London       | 0.581081    | 0.418919 |
| America/Denver      | 0.691099    | 0.308901 |
| America/Los_Angeles | 0.340314    | 0.659686 |
| America/Chicago     | 0.287500    | 0.712500 |
|                     | 0.470250    | 0.529750 |
| America/New_York    | 0.270983    | 0.729017 |

生成堆积条形图：

```python
aggent_normal.plot(kind='barh',stacked=True)
```

![png](《利用Python进行数据分析》范例数据集-usagov-bitly-data\output_47_1.png)

