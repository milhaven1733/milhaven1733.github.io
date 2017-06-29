---
title: 关于udacity学生数据集的t检验分析
date: 2017-04-19 22:23:28
tags:
categories: 数据分析
---

#### 前言

上一次，应用udacity提供的学生数据集，我们熟悉数据分析的基本流程，完成了数据的获取、导入、整理、取样、分类等步骤，也得到了相应的推测结论。但由于没有经过统计学检查，得到的只是未经证实的试验性结论。

因此，这一次我希望选取上次分析过程中得到的几个采样方法有代表的数据样本，对其进行统计学上的检验，看看分析结果是否能验证上次做出的一些预测或推论。

<!---more--->

#### 检验什么问题

根据上次分析中得到的一个结论：“通过第一个项目的学生上课分钟数要多于未通过的学生”，若进行验证，我们可以将要检验的论题定为：“通过第一个项目与未通过的学生，上课时长是否有显著的差异”。

#### 检验方式

通过项目的学生与未通过项目的学生为两个不同的、无相互影响的群体，针对他们的一些特征值进行对比分析，应该采用**独立样本的t检验**

#### 数据导出

上一次，为了对数据进行简单的描述统计，我们对注册一周内的参与记录进行分组，并根据字段提取了我们感兴趣的数据。那么上一次用于描述统计的数据集同样也是我们这一次进行推论统计分析的数据来源。

为了对数据有直观的感觉，也为了便于计算、对比，虽然也可以直接在程序内部继续加工，整理，分析数据，但这一次我还是希望能将数据重新导出到csv文件，利用熟悉的表格应用来展示分析过程。

Python csv库的写入功能比想象中略繁琐一些，csv.writer.writerows方法接受一个包含tuple的list，其中每个list元素（即一个tuple）写入一行，tuple的每个元素占一列，元素需要有多个且必须为string类型。（经试验，tuple只含一个元素时，该字符串会被逐字符分入单元格。。加一个','可以解决问题）

但我们之前得到的数据集，如:non_passing_minutes.values()返回的是一个元素为float类型的list，必须对数据加以加工转化才能顺利写入csv文件。

设计写入函数：

```
import csv
def writecsv(filename,listname):
    with open (filename, 'wb+') as csvfile:
        writer = csv.writer(csvfile)
        data_tuple=()
        for value in listname.values():
            data=[]	#list
            data_tuple=(value,) #list的tuple元素，一定要加','
            data.append(data_tuple)
            writer.writerows(data) #写入
```

得到csv文件：

```
items_list=[non_passing_minutes,passing_minutes]
items_name=['non_passing_minutes','passing_minutes']
for name,item in zip(items_name,items_list):
    writecsv(name+'.csv',item)
```

打开本地文件检查是否成功写入：

```
print len(non_passing_minutes.values())
print len(passing_minutes.values())
```

out:

```
348
647
```



![](关于udacity学生数据集的t检验分析\t1.png)![](关于udacity学生数据集的t检验分析\t2.png)

与list长度一致，写入完成。

#### 进入分析——假设检验

数据导出到表格，可以正式进入分析阶段。

针对我们想要研究的问题：“通过第一个项目与未通过的学生，上课时长是否有显著的差异”

我们可以先设定假设检验：

> $$T_n表示未通过项目的学生的听课时长，T_p表示通过项目的学生的听课时长$$

$$H_0:T_n=T_p(两者没有显著差异)$$ 

$$H_A:T_n >T_p (两者存在显著差异,且前者大于后者)$$

$$(\alpha=0.05)$$

首先计算两组样本用时的**均值**（单位：min）：

Pass: 		$$\bar{X}_p=394.59$$

Non_Pass:    $$\bar{X}_n=143.33$$

再来计算两组样本的**标准偏差：**(利用excel的stdev函数计算，之前分析结果中的标准差是不包含校正系数的)

$$S_p=448.85$$

$$S_n=269.93$$

接下来**计算总体的标准误差：**

根据两独立样本的标准误差的计算公式：

$$sd=\sqrt{\frac{S_1^2}{n_1}+\frac{S_2^2}{n_2}}$$

$$SEM=S_{\bar X_p-\bar X_n}=\sqrt{\frac{S_p^2}{n_p}+\frac{S_n^2}{n_n}}=22.82$$

**计算t统计量：**

$$t=\frac{\bar X_p-\bar X_n}{SEM}=11.01$$

**计算t临界值：**

$$DF=n_p+n_n-2=993$$

对照t表格可知$$df=1000,单尾\alpha=0.05 时，t_c=1.646$$

**结论：**

$$t> t_c$$

所以：拒绝零假设

证明：通过第一个项目与未通过的学生，上课时长有显著的差异，且通过项目学生听课时长显著偏高。

#### 总结

应用统计学的检验方法，我们可以证明我们的推论：通过第一个项目的学生上课分钟数要多于未通过的学生。

用类似的方式，我们也可以验证对与其他数据集的推论，此篇不赘述。