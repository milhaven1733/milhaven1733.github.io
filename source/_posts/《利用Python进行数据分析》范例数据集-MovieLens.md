---
title: 《利用Python进行数据分析》范例数据集_MovieLens
date: 2017-05-03 22:27:51
tags:
categories: 数据分析
---



```python
import pandas as pd
```

加载.dat文件中的数据：

```python
unames=['user_id','gender','age','occupation','zip']
path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/movielens/users.dat'
users=pd.read_table(path,sep='::',header=None,names=unames,engine = 'python')
```

```python
rnames=['user_id','movie_id','rating','timestamp']
path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/movielens/ratings.dat'
ratings=pd.read_table(path,sep='::',header=None,names=rnames,engine = 'python')
```

```python
mnames=['movie_id','title','genres']
path='/home/milhaven1733/Desktop/Data Analyst/pydata-book-master/ch02/movielens/movies.dat'
movies=pd.read_table(path,sep='::',header=None,names=mnames,engine = 'python')
```

<!---more--->

查看数据加载情况：

```python
users[:5]
```

|      | user_id | gender | age  | occupation | zip   |
| ---- | ------- | ------ | ---- | ---------- | ----- |
| 0    | 1       | F      | 1    | 10         | 48067 |
| 1    | 2       | M      | 56   | 16         | 70072 |
| 2    | 3       | M      | 25   | 15         | 55117 |
| 3    | 4       | M      | 45   | 7          | 02460 |
| 4    | 5       | M      | 25   | 20         | 55455 |

```python
ratings[:5]
```

|      | user_id | movie_id | rating | timestamp |
| ---- | ------- | -------- | ------ | --------- |
| 0    | 1       | 1193     | 5      | 978300760 |
| 1    | 1       | 661      | 3      | 978302109 |
| 2    | 1       | 914      | 3      | 978301968 |
| 3    | 1       | 3408     | 4      | 978300275 |
| 4    | 1       | 2355     | 5      | 978824291 |

```python
movies[:5]
```

|      | movie_id | title                              | genres                         |
| ---- | -------- | ---------------------------------- | ------------------------------ |
| 0    | 1        | Toy Story (1995)                   | Animation\|Children's\|Comedy  |
| 1    | 2        | Jumanji (1995)                     | Adventure\|Children's\|Fantasy |
| 2    | 3        | Grumpier Old Men (1995)            | Comedy\|Romance                |
| 3    | 4        | Waiting to Exhale (1995)           | Comedy\|Drama                  |
| 4    | 5        | Father of the Bride Part II (1995) | Comedy                         |

利用Pandas的merge函数合并三个表（Pandas会根据列名的重叠情况自动推断合并的键）：

```python
pd.merge(users,ratings).head()
```

|      | user_id | gender | age  | occupation | zip   | movie_id | rating | timestamp |
| ---- | ------- | ------ | ---- | ---------- | ----- | -------- | ------ | --------- |
| 0    | 1       | F      | 1    | 10         | 48067 | 1193     | 5      | 978300760 |
| 1    | 1       | F      | 1    | 10         | 48067 | 661      | 3      | 978302109 |
| 2    | 1       | F      | 1    | 10         | 48067 | 914      | 3      | 978301968 |
| 3    | 1       | F      | 1    | 10         | 48067 | 3408     | 4      | 978300275 |
| 4    | 1       | F      | 1    | 10         | 48067 | 2355     | 5      | 978824291 |

```python
data=pd.merge(pd.merge(users,ratings),movies)
data.head()
```

|      | user_id | gender | age  | occupation | zip   | movie_id | rating | timestamp | title                                  | genres |
| ---- | ------- | ------ | ---- | ---------- | ----- | -------- | ------ | --------- | -------------------------------------- | ------ |
| 0    | 1       | F      | 1    | 10         | 48067 | 1193     | 5      | 978300760 | One Flew Over the Cuckoo's Nest (1975) | Drama  |
| 1    | 2       | M      | 56   | 16         | 70072 | 1193     | 5      | 978298413 | One Flew Over the Cuckoo's Nest (1975) | Drama  |
| 2    | 12      | M      | 25   | 12         | 32793 | 1193     | 4      | 978220179 | One Flew Over the Cuckoo's Nest (1975) | Drama  |
| 3    | 15      | M      | 25   | 7          | 22903 | 1193     | 4      | 978199279 | One Flew Over the Cuckoo's Nest (1975) | Drama  |
| 4    | 17      | M      | 50   | 1          | 95350 | 1193     | 5      | 978158471 | One Flew Over the Cuckoo's Nest (1975) | Drama  |



对合并后的表进行聚合操作（数据透视）。根据电影title分类后按评分用户的性别计算每部电影的平均得分：

```python
mean_ratings=data.pivot_table('rating',index=['title'],columns=['gender'],aggfunc='mean')
mean_ratings.head()
```

| gender                        | F        | M        |
| ----------------------------- | -------- | -------- |
| title                         |          |          |
| $1,000,000 Duck (1971)        | 3.375000 | 2.761905 |
| 'Night Mother (1986)          | 3.388889 | 3.352941 |
| 'Til There Was You (1997)     | 2.675676 | 2.733333 |
| 'burbs, The (1989)            | 2.793478 | 2.962085 |
| ...And Justice for All (1979) | 3.828571 | 3.689024 |

过滤评分数据小于250分的电影：

根据title对评分数据分组，用size得到各组数据量的大小：

```python
ratings_by_title=data.groupby('title').size()
ratings_by_title.head()
```

```
title
$1,000,000 Duck (1971)            37
'Night Mother (1986)              70
'Til There Was You (1997)         52
'burbs, The (1989)               303
...And Justice for All (1979)    199
dtype: int64
```

从上面的Series中分离出size>250的数据：

```python
active_titles=ratings_by_title[ratings_by_title>=250]
```

根据active_titles的条目的索引分离mean_ratings中评分数据多于250条的影片信息：

```python
active_mean_ratings=mean_ratings.ix[active_titles.index]
active_mean_ratings.head()
```

| gender                            | F        | M        |
| --------------------------------- | -------- | -------- |
| title                             |          |          |
| 'burbs, The (1989)                | 2.793478 | 2.962085 |
| 10 Things I Hate About You (1999) | 3.646552 | 3.311966 |
| 101 Dalmatians (1961)             | 3.791444 | 3.500000 |
| 101 Dalmatians (1996)             | 3.240000 | 2.911215 |
| 12 Angry Men (1957)               | 4.184397 | 4.328421 |

根据’F‘列降序排序，得到女性观众评分最高电影的前十条：

```python
top_female_ratings=active_mean_ratings.sort_values(by='F',ascending=False)
top_female_ratings.head()
```

| gender                                   | F        | M        |
| ---------------------------------------- | -------- | -------- |
| title                                    |          |          |
| Close Shave, A (1995)                    | 4.644444 | 4.473795 |
| Wrong Trousers, The (1993)               | 4.588235 | 4.478261 |
| Sunset Blvd. (a.k.a. Sunset Boulevard) (1950) | 4.572650 | 4.464589 |
| Wallace & Gromit: The Best of Aardman Animation (1996) | 4.563107 | 4.385075 |
| Schindler's List (1993)                  | 4.562602 | 4.491415 |

找出男女观众评分分歧最大的影片：

对active_mean_ratings添加一列’diff‘表示女性观众与男性观众对某部电影评分均值的差值：

```python
active_mean_ratings['diff']=active_mean_ratings['F']-active_mean_ratings['M']
```

根据diff字段排序，得到差值最大的前5条：

```python
sort_by_diff=active_mean_ratings.sort_values(by='diff',ascending=False)
sort_by_diff.head()
```

| gender                    | F        | M        | diff     |
| ------------------------- | -------- | -------- | -------- |
| title                     |          |          |          |
| Dirty Dancing (1987)      | 3.790378 | 2.959596 | 0.830782 |
| Jumpin' Jack Flash (1986) | 3.254717 | 2.578358 | 0.676359 |
| Grease (1978)             | 3.975265 | 3.367041 | 0.608224 |
| Little Women (1994)       | 3.870588 | 3.321739 | 0.548849 |
| Steel Magnolias (1989)    | 3.901734 | 3.365957 | 0.535777 |



将排序结果逆序，得到男性观众评分更高的5部：

```python
sort_by_diff[::-1].head()
```

| gender                                 | F        | M        | diff      |
| -------------------------------------- | -------- | -------- | --------- |
| title                                  |          |          |           |
| Good, The Bad and The Ugly, The (1966) | 3.494949 | 4.221300 | -0.726351 |
| Kentucky Fried Movie, The (1977)       | 2.878788 | 3.555147 | -0.676359 |
| Dumb & Dumber (1994)                   | 2.697987 | 3.336595 | -0.638608 |
| Longest Day, The (1962)                | 3.411765 | 4.031447 | -0.619682 |
| Cable Guy, The (1996)                  | 2.250000 | 2.863787 | -0.613787 |

不考虑性别因素，仅根据评分标准差得出评分分歧最大的影片：

```python
rating_std_by_title=data.groupby('title')['rating'].std()
```

筛除评分数据小于250条的：

```python
rating_std_by_title=rating_std_by_title.ix[active_titles.index]
```

降序排序得到标准差最大的五部：

```python
rating_std_by_title.sort_values(ascending=False).head()
```

```
title
Dumb & Dumber (1994)                     1.321333
Blair Witch Project, The (1999)          1.316368
Natural Born Killers (1994)              1.307198
Tank Girl (1995)                         1.277695
Rocky Horror Picture Show, The (1975)    1.260177
Name: rating, dtype: float64
```

