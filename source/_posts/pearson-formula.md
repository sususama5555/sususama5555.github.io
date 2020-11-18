---
title: 'pearson相关系数'
date: 2020-02-26 16:09:38
tags: 
- Python
- 大数据分析
- 算法
categories: 
- 学习笔记
---
**pearson相关系数**

![avatar](/picture/pearson公式.png)

公式定义为： 两个连续变量(X,Y)的pearson相关性系数(Px,y)等于它们之间的协方差cov(X,Y)除以它们各自标准差的乘积(σX,σY)。系数的取值总是在-1.0到1.0之间，接近0的变量被成为无相关性，接近1或者-1被称为具有强相关性。

简单来说，它用来衡量两个数据集合是否在一条线上面，是否有相关性，这在数据分析中是很有效的。

用python3实现：
<!--more-->
```python
import math

def pearson(vector1, vector2):
    n = len(vector1)
    #simple sums
    sum1 = sum(float(vector1[i]) for i in range(n))
    sum2 = sum(float(vector2[i]) for i in range(n))
    #sum up the squares
    sum1_pow = sum([pow(v, 2.0) for v in vector1])
    sum2_pow = sum([pow(v, 2.0) for v in vector2])
    #sum up the products
    p_sum = sum([vector1[i]*vector2[i] for i in range(n)])
    #分子num，分母den
    num = p_sum - (sum1*sum2/n)
    den = math.sqrt((sum1_pow-pow(sum1, 2)/n)*(sum2_pow-pow(sum2, 2)/n))
    if den == 0:
        return 0.0
    return num/den
```
选择两组数据
```
vector1 = [2, 7, 18, 88, 157, 90, 177, 570]
vector2 = [3, 5, 15, 90, 180, 88, 160, 580]
print('result is: ' + int(pearson(vector1, vector2)))
```
运行结果为0.998，可见这两组数是高度正相关的
```
result is: 0.998348748644
```

&emsp;&emsp;美国零售业有这样一个案例，美国沃尔玛百货将他们的纸尿裤和啤酒并排摆在一起销售，结果纸尿裤和啤酒的销量双双增长。
原来，美国的太太们常叮嘱她们的丈夫下班后为小孩买尿布，而丈夫们在买尿布后又随手带回了两瓶啤酒。
这一消费行为导致了这两件商品经常被同时购买。这其实是经过数据挖掘、趋势分析后做出的决策。

******
参考：[统计学三大相关系数之皮尔森（pearson）相关系数](https://blog.csdn.net/AlexMerer/article/details/74908435)  
&emsp;&emsp;&emsp;[从啤酒和纸尿裤，你能想到什么？](https://www.jianshu.com/p/a8349052a2a0)