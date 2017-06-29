---
title: 利用udacity学生数据集实践数据分析过程
date: 2017-04-19 11:22:06
tags:
categories: 数据分析
---

### 利用udacity学生数据集实践数据分析过程

#### 数据采集与清理

**数据分析中的数据来源**

- 下载数据文件
- 从API中获取 
- 从网页收集 
- 合并多种不同格式数据

在这次实践中，我们采用下载数据文件的方式。（将课程提供的三个csv数据集下载至本地)

接下来，需要对数据文件进行数据整理使数据能够用于分类及分析。

就要先了解我们下载的csv文件。

<!--more-->

**关于CSV**

Comma Separated Values——逗号分隔值

易于通过代码处理（相比.xlsx）

csv文件的表格与文本格式:

![csv文件的表格与文本格式](利用udacity学生数据集实践数据分析过程\1.png)

#### Python中的CSV

csv文件内容通常呈现为一系列行

- 每行为一个列表，整体数据结构为包含一系列列表的列表

  ```csv=[[&#39;A1&#39;,&#39;A2&#39;,&#39;A3&#39;],[&#39;B1&#39;,&#39;B2&#39;,&#39;B3&#39;]]
  csv=[['A1','A2','A3'],['B1','B2','B3']]
  ```

- 每行为一个字典，整体数据结构为包含一系列字典的列表（标题行的每个关键词为键，每个字段作为值。）

  ```
  csv=[{'name1':'A1','name2':'A2','name3':'A3'},{'name1':'B1','name2':'B2','name3':'B3'}]
  ```

**Python unicodecsv库**

```
import unicodecsv
enrollments=[]
f=open('/home/milhaven1733/enrollments.csv','rb')
reader=unicodecsv.DictReader(f)

for row in reader:
    enrollments.append(row)
f.close()
enrollments[0]
```

输出：

```
{u'account_key': u'448',
 u'cancel_date': u'2015-01-14',
 u'days_to_cancel': u'65',
 u'is_canceled': u'True',
 u'is_udacity': u'True',
 u'join_date': u'2014-11-10',
 u'status': u'canceled'}
```

运用DictReader方法将csv文件每行转化为字典并生成一个迭代器（注意迭代器只能一次用于循环一次。）

优化代码：

```
import unicodecsv
with open('/home/milhaven1733/enrollments.csv','rb') as f:
    reader=unicodecsv.DictReader(f)
    enrollments=list(reader)
enrollments[0]
```

利用list方法不使用循环生成列表

分别读取所需的三个csv文件：

```
import unicodecsv 
def read_csv(filename):
    with open(filename, 'rb') as f:
        reader = unicodecsv.DictReader(f)
        return list(reader)
enrollments = read_csv('/home/milhaven1733/enrollments.csv')
daily_engagement = read_csv('/home/milhaven1733/daily_engagement.csv')
project_submissions = read_csv('/home/milhaven1733/project_submissions.csv')
```

#### 修正数据类型

从生成的字典列表中可以看出，unicodecsv处理时并不区分数据类型，需要根据实际使用来转换数据类型。（最好在获取数据之初就进行转换以免遗忘）

修正enrollments：

```
{u'account_key': u'448',
 u'cancel_date': u'2015-01-14',
 u'days_to_cancel': u'65',
 u'is_canceled': u'True',
 u'is_udacity': u'True',
 u'join_date': u'2014-11-10',
 u'status': u'canceled'}
```

```
from datetime import datetime as dt
def parse_date(date):
    if date=='':
        return None
    else:
        return dt.strptime(date,'%Y-%m-%d')
    
def parse_int(i):
    if i=='':
        return None
    else:
        return int(i)

for enrollment in enrollments:
    enrollment['join_date']=parse_date(enrollment['join_date'])
    enrollment['cancel_date']=parse_date(enrollment['cancel_date'])
    enrollment['days_to_cancel']=parse_int(enrollment['days_to_cancel'])
    enrollment['is_udacity']=enrollment['is_udacity']=='True'
    enrollment['is_canceled']=enrollment['is_canceled']=='True'
```

**运用datetime.strptime转换时间格式**

修正Project_submissions：

```
{u'account_key': u'256',
 u'assigned_rating': u'UNGRADED',
 u'completion_date': u'2015-01-16',
 u'creation_date': u'2015-01-14',
 u'lesson_key': u'3176718735',
 u'processing_state': u'EVALUATED'}
```

```
for submission in project_submissions:
    submission['creation_date']=parse_date(submission['creation_date'])
    submission['completion_date']=parse_date(submission['completion_date'])
```

修正daily_engagement：

```
{u'acct': u'0',
 u'lessons_completed': u'0.0',
 u'num_courses_visited': u'1.0',
 u'projects_completed': u'0.0',
 u'total_minutes_visited': u'11.6793745',
 u'utc_date': u'2015-01-09'}
```

```
for engagement in daily_engagement:
    engagement['lessons_completed']=int(float(engagement['lessons_completed']))
    engagement['num_courses_visited']=int(float(engagement['num_courses_visited']))
    engagement['projects_completed']=int(float(engagement['projects_completed']))
    engagement['total_minutes_visited']=float(engagement['total_minutes_visited'])
    engagement['utc_date']=parse_date(engagement['utc_date'])
```

**先将包含小数点的字符串转化为float再转化为int**

#### 对处理好的数据集提出疑问

数据集处理完成后需要针对数据提出疑问，如：

- 通常需要多久提交项目？
- 通过和未通过项目的学生有什么区别？
- 学员上课的平均时间
- 上课时间、数量与完成项目之间的关系
- 参与度随时间的改变
- 通过项目前提交的次数

等。

本次探究针对问题：**通过与未通过首个项目的学生参与课程的差异。**

#### 整理数据

成功加载数据并确保数据格式良好，说明你已经开始数据整理过程了。下一步就是调查，看看数据中是否存在不一致处或问题，如果有，则需要清理它们。

可以在每个数据表中查找不重复学员数，并创建一组帐号列表：（以处理enrollments列表为例）

```
unique_enrollment=[]
for enrollment in enrollments:
    unique_enrollment.append(enrollment['account_key'])
unique_enrollment=set(unique_enrollment) 
```

用set(list)达到去重目的。

#### 修正数据表中的问题

enrollments表与daily_engagement表中，表示学员账户的键名称不同，可能会对之后的数据处理及分析产生影响，怎么修改能使之保持一致？

```
for engagement_record in daily_engagement:
    engagement_record['account_key'] = engagement_record['acct']
    del[engagement_record['acct']]
```

修改后，亦可创建函数方便实现上个话题中三个数据表分别清理重复学员账户的问题。

#### 缺失的参与记录

从之前的学员账户统计结果可以看出，enrollments表与daily_engagement表中，'account_key‘的数量不同，为什么部分注册用户没有参与记录呢？

我们可以找出缺失参与记录的注册用户，对其特点展开探究：

```
for enrollment in enrollments:
    if enrollment['account_key'] not in unique_engagement:
        print enrollment
```

通过遍历enrollments表，查找缺失参与记录的用户。大体可以发现：

**缺失记录的用户，大多在同一天注册与注销。**

（即可能只有账户存在时长超过一天，才会存在参与记录）

#### 核查更多问题记录

我们上面找到的原因是造成部分用户缺失参与记录的全部原因吗？是否存在其他原因？可以对缺失参与记录的注册用户展开进一步探究：

```
for enrollment in enrollments:
    if enrollment['account_key'] not in unique_engagement and enrollment['join_date'] != enrollment['cancel_date']:
        print enrollment
```

#### 剩余问题

对上个问题中存在的另外3个异常用户展开探究，发现其’is_udacity‘的值为True，即为系统的测试用户——找到了问题。

接下来，我们需要排除数据集中的测试用户，以避免对之后分析的干扰。

首先，创建测试用户集合：

```
test_account=[]
for enrollment in enrollments:
    if enrollment['is_udacity']:
        test_account.append(enrollment['account_key'])
test_account=set(test_account)
```

创建从数据集中清理测试用户的函数：

```
def remove_uda_acc(data):
    non_uda_data=[]
    for data_point in data:
        if data_point['account_key'] not in test_account:
            non_uda_data.append(data_point)
    return non_uda_data
```

执行函数，创建不包含测试用户的新数据集：

```
non_uda_enrollments=remove_uda_acc(enrollments)
non_uda_engagement=remove_uda_acc(daily_engagement)
non_uda_submissions=remove_uda_acc(project_submissions)
```

#### 提炼问题

目前，**想探究的问题**：

对于通过和未通过第一个项目的学员，他们在daily_engagement表中的数据有何不同？

目前整理出的**数据集**对于探究这个问题存在的**缺陷**：

- 提交后的参与数据与项目无关
- 可能会对比不同时间段的参与数据（参与度受时间影响）
- 参与数据包含不属于第一个项目的课程

对策：（针对前两个问题）

只查看学生注册前一周的数据，并排除一周内注销的用户。

针对以上方案，可以首先：创建未注销或注销前注册时长超过七天的学生字典paid_students

对于字典：Key：’account_key‘, Value:enrollment_date

创建过程如下：

```
paid_student={}
for enrollment in non_uda_enrollments:
    if (not enrollment['is_canceled']) or (enrollment['days_to_cancel']>7):
        account_key=enrollment['account_key']
        enrollment_date=enrollment['join_date']
        if(account_key not in paid_student or 
           enrollment_date>paid_student[account_key]):
            paid_student[account_key]=enrollment_date
```

由于一个学生可能多次注册，仅在account_key不存在，或enrollment_date比原注册日期更晚时才添加数据，可以保证保存的的是最近的注册日期。

#### 获取第一周数据

创建新列表，存储paid_students中学生注册一周内的参与数据：

首先，移除各数据集中注册时长小于一周的学生数据（即account_key未在paid_students中的数据）

```
def remove_free_trial_cancels(data):
    new_data = []
    for data_point in data:
        if data_point['account_key'] in paid_students:
            new_data.append(data_point)
    return new_data

paid_enrollments = remove_free_trial_cancels(non_udacity_enrollments)
paid_engagement = remove_free_trial_cancels(non_udacity_engagement)
paid_submissions = remove_free_trial_cancels(non_udacity_submissions)
```

创建检测参与时间是否在注册一周内的函数：

```
def within_one_week(join_date, engagement_date):
    time_delta = engagement_date - join_date
    return time_delta.days < 7
```

最后，创建新列表并存储符合要求的数据：（利用上述函数进行检验）

```
paid_engagement_in_first_week = []
for engagement_record in paid_engagement:
    account_key = engagement_record['account_key']
    join_date = paid_students[account_key]
    engagement_record_date = engagement_record['utc_date']

    if within_one_week(join_date, engagement_record_date):
        paid_engagement_in_first_week.append(engagement_record)
```

#### 对参与度的探索

怎样探索某学员第一周上课的平均时间？

可以将搜娱的参与记录按学员账户分组存入字典，每组包含某学生的所有参与记录

key:account_key

Value:engagement_table

```
from collections import defaultdict #允许设置默认值的字典

engagement_by_account=defaultdict(list) #当在字典中寻找商不存在的关键字，会得到一个空列表。
for engagement_record in paid_engagement_in_first_week:
    account_key=engagement_record['account_key']
    engagement_by_account[account_key].append(engagement_record)
```

运行代码后，一周内参与数据已分组存入字典。

接下来，可以另设一字典，存入某学生一周内学习的总时长

Key:account_key

Value:total_minutes

```
total_minutes_by_account={}
for account_key,engagement_for_student in engagement_by_account.items():
    total_minutes=0
    for engagement_record in engagement_for_student:
        total_minutes+=engagement_record['total_minutes_visited']
    total_minutes_by_account[account_key]=total_minutes
```

得到了包含所有学员一周内学习时长的字典，可以通过dict.values()方法，将字典中所有value导入一列表，引入numpy库，进行简单的分析：

```
total_minutes=total_minutes_by_account.values()
import numpy as np
print 'Mean:',np.mean(total_minutes)
print 'Standard deviation:',np.std(total_minutes)
print 'Min:',np.min(total_minutes)
print 'Max:',np.max(total_minutes)
```

从输出结果：

```
Mean: 647.590173826
Standard deviation: 1129.27121042
Min: 0.0
Max: 10568.1008673  #（异常值）
```

可以看出，数据存在一定的问题，需要进一步地探究。

#### 寻找异常所在

我们可以从学习时长最大的account入手，寻找数据采集是否存在问题。

首先，查找max_minutes对应的max_account:

```
max_minutes=0
for account_key,total_minutes in total_minutes_by_account.items():
    if total_minutes>max_minutes:
        max_minutes=total_minutes
        max_account=account_key
```

在一周参与记录中查找该账户对应的记录以及该账户的注册信息：

```
for engagement in paid_engagement_in_first_week:
    if engagement['account_key']==max_account:
        print engagement
print paid_student[max_account]
```

从输出结果看到条目远远大于7条，即收集的参与数据包含了学员最新一次注册之前的所有数据，检查可发现，是检测参与时间是否在注册一周内的函数出现问题，加以更正：

```
def within_one_week(join_date, engagement_date):
    time_delta = engagement_date - join_date
    return time_delta.days < 7 and time_delta.days>=0
```

重新运行其下代码，在新的数据集中进行简要分析，得到结果：

```
Mean: 306.708326753
Standard deviation: 412.996933409
Min: 0.0
Max: 3564.7332645
```

虽然Max值接近60小时，但较为可信。

可以执行本节中前两段代码再次检测最大值对应用户的参与记录，核验是否仍存在问题。

#### 探索课程完成情况

怎样探索学员第一周完成的课程数量？

利用目前已经得到的某学员一周内参与记录，我们可以用类似得到上课时长的方式得到完成课程数量。

由于代码相似度非常高，可以定义几个函数，完成类似的内容，如：

```
def sum_grouped_items(grouped_data, field_name):
    summed_data = {}
    for account_key,engagement_for_student in grouped_data.items():
        total=0
        for engagement_record in engagement_for_student:
            total+=engagement_record[field_name]
        summed_data[account_key]=total
    return summed_data
    
import numpy as np
def describe_data(data):
    print 'Mean:', np.mean(data)
    print 'Standard deviation:', np.std(data)
    print 'Minimum:', np.min(data)
    print 'Maximum:', np.max(data)
```

得到某学员某项指标总值的函数，与简要分析包含所有学员某项总值的列表的函数

进行分析:

```
total_minutes_by_account=sum_grouped_items(engagement_by_account,'total_minutes_visited')
lessons_completed=sum_grouped_items(engagement_by_account,'lessons_completed')
describe_data(total_minutes_by_account.values())
describe_data(lessons_completed.values())
```

为增强运算灵活性，也可以在这里将按学员account_key分组得到某学员参与数据的代码重写为函数并执行：

```
from collections import defaultdict #允许设置默认值的字典
def group_data(data,key_name):
    grouped_data=defaultdict(list) #当在字典中寻找商不存在的关键字，会得到一个空列表。
    for data_point in data:
        key=data_point[key_name]
        grouped_data[key].append(data_point)
    return grouped_data
engagement_by_account=group_data(paid_engagement_in_first_week,'account_key')
```

#### 探索一周访问天数

怎样可以得到学员每周访问课程的天数？

从记录中可以看出，学员某日记录的‘num_courses_visited’字段不同。大于0说明当日访问了课程，等于0则说明没有访问，我们可以为记录新增加'has_visited'字段，来记录当天是否访问，再调用上节的各个函数，就可以计算一周内访问天数的总和以及对所有访问天数进行分析了。

为paid_engagement（所有注册时间长于一周的学员的记录）添加'has_visited'字段：

```
for engagement in paid_engagement:
    if engagement['num_courses_visited']>0:
        engagement['has_visited']=1
    else:
        engagement['has_visited']=0
```

进行统计和分析：

```
total_visited=sum_grouped_items(engagement_by_account,'has_visited')
describe_data(total_visited.values())
```

#### 划分学员参与记录

现在继续正题，以是否通过第一个项目为标准，将学员划分为两组，同时也将参与记录划分为两组。

首先查看一条提交记录：

```
paid_submissions[0]:
{u'account_key': u'256',
 u'assigned_rating': u'UNGRADED',
 u'completion_date': datetime.datetime(2015, 1, 16, 0, 0),
 u'creation_date': datetime.datetime(2015, 1, 14, 0, 0),
 u'lesson_key': u'3176718735',
 u'processing_state': u'EVALUATED'}
```

'lesson_key'字段指示提交项目的代码

'assigned_rating'字段指示项目是否通过

而首个项目的'lesson_key'为'746169184' 或 '3176718735'

表示通过项目的状态为：'PASSED'或'DISTINCTION'

可以执行以下代码，将所有通过第一个项目的学员的account_key 加入pass_subway_project列表：

```
subway_project_lesson_keys = ['746169184', '3176718735']
rating_passed=['PASSED','DISTINCTION']
pass_subway_project = set()
for submission in paid_submissions:
    account_key=submission['account_key']
    lesson_key=submission['lesson_key']
    assigned_rating=submission['assigned_rating']
    if (lesson_key in subway_project_lesson_keys) and (assigned_rating in rating_passed):
        pass_subway_project.add(account_key)
```

再以account_key是否在pass_subway_project列表中为标准划分参与记录：

```
passing_engagement = []
non_passing_engagement = []
for engagement_record in paid_engagement_in_first_week:
    if engagement_record['account_key'] in pass_subway_project:
        passing_engagement.append(engagement_record)
    else:
        non_passing_engagement.append(engagement_record)
```

#### 比较两组学员

现在可以调用之间group_data与统计、分析函数，新创建两个包含某学员参与数据的字典并进行通过与未通过项目的学员参与数据之间统计与比较：

```
passing_engagement_by_account = group_data(passing_engagement,'account_key')
non_passing_engagement_by_account = group_data(non_passing_engagement,'account_key')

print 'total_minutes_visited:'
print 'non-passing students:'
non_passing_minutes = sum_grouped_items(non_passing_engagement_by_account,'total_minutes_visited')
describe_data(non_passing_minutes.values())

print '\npassing students:'
passing_minutes = sum_grouped_items(passing_engagement_by_account,'total_minutes_visited')
describe_data(passing_minutes.values())

print '\n\nlessons_completed:'
print 'non-passing students:'
non_passing_lessons = sum_grouped_items(non_passing_engagement_by_account,'lessons_completed')
describe_data(non_passing_lessons.values())

print '\npassing students:'
passing_lessons = sum_grouped_items(passing_engagement_by_account,'lessons_completed')
describe_data(passing_lessons.values())

print '\n\nhas_visited:'
print 'non-passing students:'
non_passing_visits = sum_grouped_items(non_passing_engagement_by_account,'has_visited')
describe_data(non_passing_visits.values())

print '\npassing students:'
passing_visits = sum_grouped_items(passing_engagement_by_account,'has_visited')
describe_data(passing_visits.values())
```

输出结果：

```
total_minutes_visited:
non-passing students:
Mean: 143.326474267
Standard deviation: 269.538619011
Minimum: 0.0
Maximum: 1768.52274933

passing students:
Mean: 394.586046484
Standard deviation: 448.499519327
Minimum: 0.0
Maximum: 3564.7332645


lessons_completed:
non-passing students:
Mean: 0.862068965517
Standard deviation: 2.54915994183
Minimum: 0
Maximum: 27

passing students:
Mean: 2.05255023184
Standard deviation: 3.14222705558
Minimum: 0
Maximum: 36


has_visited:
non-passing students:
Mean: 1.90517241379
Standard deviation: 1.90573144136
Minimum: 0
Maximum: 7

passing students:
Mean: 3.38485316847
Standard deviation: 2.25882147092
Minimum: 0
Maximum: 7
```

可对以上分析结果自行进行比较探究。

#### 创建直方图

对于上述6个统计结果，可以创建图表加以描述。

**jupyter notebook创建直方图：**

```
data = [1, 2, 1, 3, 3, 1, 4, 2]
%matplotlib inline  #设定图表在notebook内部输出
import matplotlib.pyplot as plt
plt.hist(data)
```

得到以下图表：

![](利用udacity学生数据集实践数据分析过程\plot.png)

[关于hist参数](http://blog.csdn.net/u013571243/article/details/48998619)

参照示例创建描述及绘图函数：

```
%matplotlib inline
import matplotlib.pyplot as plt
import numpy as np
def describe_data(data,bins,x_label,y_label,title):
    print 'Mean:', np.mean(data)
    print 'Standard deviation:', np.std(data)
    print 'Minimum:', np.min(data)
    print 'Maximum:', np.max(data)
    plt.hist(data,bins)
    plt.xlabel(x_label)  
    plt.ylabel(y_label)  
    plt.title(title)  
```

首先对比两组学生的听课总时长分布：

```
describe_data(non_passing_minutes.values(),50,
	'Time/minutes','Number of people',r'non_passing_students_total_minutes')
describe_data(passing_minutes.values(),50,
	'Time/minutes','Number of people',r'passing_students_total_minutes')
```

得到图像：

![](利用udacity学生数据集实践数据分析过程\plot1.png)![](利用udacity学生数据集实践数据分析过程\plot2.png)

对比发现，分布形状大致类似，但通过项目的学生组持续较长听课时间的人数要明显多于未通过组。

然后对比两组完成课程数量的分布：

```
describe_data(non_passing_lessons.values(),54,
              'Lessons','Number of people',r'non_passing_students_lessons')
describe_data(passing_lessons.values(),72,
              'Lessons','Number of people',r'non_passing_students_lessons')
```

![](利用udacity学生数据集实践数据分析过程\plot3.png)![](利用udacity学生数据集实践数据分析过程\plot4.png)

发现：虽然第一周两组内都是未听课的人数居多，但通过组听课量保持在1-10节的人数明显多于未通过组。

最后对比两组访问课程天数：

```
describe_data(non_passing_visits.values(),20,
              'Visits/days','Number of people',r'non_passing_students_visits')
describe_data(passing_visits.values(),20,
              'Visits/days','Number of people',r'passing_students_visits')
```

![](利用udacity学生数据集实践数据分析过程\plot5.png)![](利用udacity学生数据集实践数据分析过程\plot6.png)

这次的图形外形差异很大：

对与未通过组，随着天数增多，人数逐渐减少，而通过组，各个访问天数内人数相差不大。

#### 是否能得到结论？

通过以上对变量间联系的探究，我们可以做出一些预测或初步结论：

如：通过第一个项目的学生上课分钟数要多于未通过的学生。

但是，两组间的差异是真实存在的差异还是偶然造成（或受一些隐含因素影响）的呢？

要得到确切的结论，还需要严格的统计学检查，否则得到的只是未经证实的试验性结论。

关于应用统计学分析以上数据集得出更严格的结论，可以看下一篇博客：

[关于udacity学生数据集的t检验分析](https://milhaven1733.github.io/2017/04/19/%E5%85%B3%E4%BA%8Eudacity%E5%AD%A6%E7%94%9F%E6%95%B0%E6%8D%AE%E9%9B%86%E7%9A%84t%E6%A3%80%E9%AA%8C%E5%88%86%E6%9E%90/)

#### 关于相关性与因果性

相关性：通过第一个项目的学生更愿意在第一周内多听课

因果性：在第一周内更多上课可以使学生通过第一个项目

两者之间有非常大的不同，如果想要验证因果关系，可以进行**A/B测试**

#### 基于众多特征进行预测

通过以上的整理分析，现在可以尝试预测：**哪些学生更可能通过首个项目？**

可以运用启发式方法（heuristics）

也可以利用**机器学习**，自动进行较为准确的预测。

#### 改善图形分享心得

在得出结论和做出预测后，需要分享研究结果、进行交流。

但如果想以可视化的方法呈现结果，最好可以对图形加以优化，使其更美观、更能表现图表想要传达的内容。

如，可以添加坐标轴名称，图表标题，改变bins参数设置直方图所使用的分组数量等优化图表

还可以使用 seaborn 库自动美化 matplotlib 图形。

即在代码中导入此库：

```
import seaborn as sns 
```

在此后创建的图形就会自动进行美化。

优化后的效果

![](利用udacity学生数据集实践数据分析过程\plot7.png)

![](利用udacity学生数据集实践数据分析过程\plot8.png)

至此，我们较为完整地完成了一个数据分析的流程，但还有许多不完善的地方有待加以补充。