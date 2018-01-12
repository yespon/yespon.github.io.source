---
title: k-近邻算法
date: 2017-12-28 11:56:02
category:
    - 机器学习
tags:
    - kNN 算法
---

## k-近邻算法

---
优点：精度高，对异常值不敏感、无数据输入假定

缺点：计算复杂度高、空间复杂度高

适用范围：数值型和标量型

---

### k近邻算法的一般流程
  1. 收集数据：可以使用任何方法
  2. 准备数据：距离计算所需要的数值，最好是结构化的数据格式
  3. 分析数据：any
  4. 训练算法：kNN无需预训练
  5. 测试算法：计算错误率
  6. 使用算法：首先需要输入样本数据和结构化的输出结果，然后运行k近邻算法判定输入数据分别属于哪个分类，最后应用对计算出的分类执行后续的处理

<!--more-->

```python
# 导入数据
import numpy as np
import operator as op

def createDataset():
    group = np.array([[1.0,1.1],
                      [1.0,1.0],
                      [0,0],
                      [0,0.1]])
    labels = ['A','A','B','B']
    return group, labels
```


```python
# kNN 分类器实现
def kNN_Classify(inX, dataSet, labels, k):
    dataSetSize = dataSet.shape[0]
    # 距离计算 2-范数 （欧式距离）
    diffMat = np.tile(inX, (dataSetSize, 1)) - dataSet # 重复A，B次，这里的B可以时int类型也可以是远组类型,eg.这里是延行轴复制dataSetSize倍，列复制1倍
    sqDiffMat = diffMat ** 2
    sqDistances = sqDiffMat.sum(axis=1)# 没有axis参数表示全部相加，axis＝0表示按列相加，axis＝1表示按照行的方向相加
    distances = sqDistances ** 0.5
    sortedDistIndicies = distances.argsort() # argsort()函数，是numpy库中的函数，返回的是数组值从小到大的索引值
    classCount = {}
    # 选择距离最小的k个点
    for i in range(k):
        voteIlabel = labels[sortedDistIndicies[i]]
        classCount[voteIlabel] = classCount.get(voteIlabel, 0) + 1
    # 排序
    sortedClassCount = sorted(classCount.items(),key=op.itemgetter(1), reverse=True)
    return sortedClassCount[0][0]

```


```python
group, labels = createDataset()
print(classify0([1,1], group, labels, 3))
```

    A
    


```python

```
