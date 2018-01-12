---
title: 使用kNN算法的手写识别系统
date: 2017-12-28 11:56:02
category:
    - 机器学习
tags:
    - kNN 算法
---
# 使用kNN算法的手写识别系统

```code
1. 收集数据：提供文本文件
2. 准备数据：编写函数classify0（），将图像格式转换为分类器使用的list格式
3. 分析数据：在Python命令提示符中检察数据，确保它符合要求
4. 训练数据：此步骤不适用kNN
5. 测试算法：编写函数使用提供的部分数据集作为测试样本，另一部分作为验证样本
6. 使用算法：
```

<!--more-->

```python
import numpy as np
import operator as op
import matplotlib
import matplotlib.pyplot as plt
import os

# 准备数据：将图像转换成测试向量
def img2vector(filename):
    # print(filename)
    returnVec = np.zeros((1, 1024))
    fr = open(filename)
    for i in range(32):
        lineStr = fr.readline()
        for j in range(32):
            returnVec[0, 32*i+j] = int(lineStr[j])
    return returnVec
```


```python
fileName = '0_0.txt'
testVec = img2vector('AnacondaProjects\\digits\\testDigits\\%s' % fileName)
testVec[0, 0:31]
```




    array([ 0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,
            1.,  1.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,  0.,
            0.,  0.,  0.,  0.,  0.])




```python
# kNN 分类器实现
def classify0(inX, dataSet, labels, k):
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
# 测试算法：使用kNN识别手写数字
def handwritingClassTest():
    hwLabels = []
    trainingFileList = os.listdir('AnacondaProjects\\digits\\trainingDigits') # 获取目录内容
    # print(trainingFileList)
    m = len(trainingFileList)
    # print(m)
    trainingMat = np.zeros((m, 1024))
    for i in range(m):
        # 从文件名解析分类数字
        fileNameStr = trainingFileList[i]
        # print(fileNameStr)
        fileStr = fileNameStr.split('.')[0]
        classNumStr = int(fileStr.split('_')[0])
        hwLabels.append(classNumStr)
        trainingMat[i,:] = img2vector('AnacondaProjects\\digits\\trainingDigits\\%s' % fileNameStr)
    testFileList = os.listdir('AnacondaProjects\\digits\\testDigits')
    errorCount = 0.0
    mTest = len(testFileList)
    for i in range(mTest):
        fileNameStr = testFileList[i]
        fileStr = fileNameStr.split('.')[0]
        classNumStr = int(fileStr.split('_')[0])
        vectorUnderTest = img2vector('AnacondaProjects\\digits\\testDigits\\%s' % fileNameStr)
        classifierResult = classify0(vectorUnderTest, trainingMat, hwLabels, 3)
        print("the classifier came back with: %d, the real answer is: %d" % (classifierResult, classNumStr))
        if (classifierResult != classNumStr):
            errorCount += 1.0
    print("\nthe total number of errors is: %d" % errorCount)
    print("\nthe total error rate is: %f" % (float(errorCount)/float(mTest)))

```


```python
handwritingClassTest()
```
