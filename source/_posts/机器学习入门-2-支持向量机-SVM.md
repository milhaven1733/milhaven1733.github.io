---
title: 机器学习入门(2) 支持向量机(SVM)
date: 2017-05-22 17:19:06
tags:
categories: 机器学习
---

#### 分隔线

算法：支持向量机（support vector machine）

核心：寻找介于两个类别的数据之间的一条separating line/hyperplane(超平面)

输入：两个类别的数据。输出：一条分隔线

#### 最优分隔线

好的分隔线的特点：最大化了到两分类最近点的距离（通常称为间隔:margin）

健壮性(robust)强，不容易出现分类误差（不易受noise干扰）

SVM内部原理：to  maximize robustness

#### 首要条件

SVM总是先考虑正确分类标签，再对间隔进行最大化（先尽力保证分类正确）

如下图选择可以正确分类的分隔线，而不是首要考虑最大化：

![](机器学习入门-2-支持向量机-SVM\1.png)

#### 对异常值的响应

当数据集中存在异常值，使可以分隔两个类的决策面不存在，SVM会产生一个决策边界，将一类数据中的异常值直接归于另一类，同时使间距最大化。

#### SKlearn&SVM

分类器创建、训练数据集与标签拟合过程：

```
from sklearn import svm
x=[[0,0],[1,1]]
y=[0,1]
clf=svm.SVC()
clf.fit(x,y)
```

注意：SVM生成的决策面为一条直线（区别于朴素贝叶斯）

同样，可以对预测结果进行准确率计算：

```
pred=clf.predict(features_test)
from sklearn.metrics import accuracy_score
acc = accuracy_score(pred, labels_test)
```

#### 非线性SVM

常规的SVM给出的决策面是线性的，对于一些数据集无法给出好的线性决策面，如：

![](机器学习入门-2-支持向量机-SVM\2.png)



此时，对于每个数据点的数据特征(x,y),引入一个新特征：

$$x^2+y^2$$

在以上三个特征上运行SVM，将得到一个决策面（新特征值为各数据点到原点距离的平方，可根据距离找到划分的超平面）

其他特征值引入：

![](机器学习入门-2-支持向量机-SVM\3.png)

传入`|x|` 将左侧数据点按沿y轴翻转，可以划分线性决策面

#### kernel trick

存在一些函数，获取低维度输入空间或特征空间，将其映射到高维度空间。（将不可线性分离的内容变为可分离）

称为**核函数** 。

可以应用kernel trick将输入空间扩大维度，再使用SVM分离数据点。

可以为决策函数制定不同的核函数，如在当前使用的SVC(支持向量分类器——一种SVM)中：

> class sklearn.svm.`SVC`(*C=1.0*, *kernel='rbf'*, *degree=3*, *gamma='auto'*, *coef0=0.0*, *shrinking=True*, *probability=False*, *tol=0.001*, *cache_size=200*, *class_weight=None*, *verbose=False*, *max_iter=-1*, *decision_function_shape=None*, *random_state=None*)

可以通过 `kernel` 参数指定核的类型

如生成直线决策边界

```
clf=svm.SVC(kernel='linear')
```

可以选用的kernel类型：

linear（线性）	poly（多项式）	rbf（径向基函数）	sigmoid（S形）等

也可以调用自己编写的核函数。

#### 其他参数

##### gamma

> Kernel coefficient for ‘rbf’, ‘poly’ and ‘sigmoid’. Higher the value of gamma, will try to exact  fit the as per training data set i.e. generalization error and cause over-fitting problem.
>
> If gamma is ‘auto’ then 1/n_features will be used instead.

gamma值越高，训练程度越精准，也容易导致过拟合现象

如gamma=1000，kernel='rbf'时分类图像：

![](机器学习入门-2-支持向量机-SVM\gamma.png)

##### C（惩罚系数）

C用来控制光滑决策边界与正确分类所有训练点之间的度。C越大，相当于惩罚松弛变量，希望松弛变量接近0，即对误分类的惩罚增大，趋向于对训练集全分对的情况，这样对训练集测试时准确率很高，但泛化能力弱。C值小，对误分类的惩罚减小，允许容错，将他们当成噪声点，泛化能力较强。

如对比C=1(默认值)与C=1000时的分类：

![](机器学习入门-2-支持向量机-SVM\c1.png)![](机器学习入门-2-支持向量机-SVM\c1000.png)

#### 过度拟合（overfitting）

过拟合：算法产生了过于复杂的分类结果。

上述几个参数的选取均可能导致过度拟合。

可以使用测试集测试检测出现过度拟合的程度。	

#### SVM的优缺点

优点：在具有复杂领域和明显分隔边界的情况下表现良好

不足：在海量数据集和noise过多的数据集中表现一般（处理时间长，处理独立证据不如bayes）

在某些使用情境下，对算法速度要求非常高，如:

- 标记信用卡诈骗，并在诈骗交易完成前进行拦截
- 声音识别，如 Siri