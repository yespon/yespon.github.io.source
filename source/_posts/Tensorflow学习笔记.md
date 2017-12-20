---
title: Tensorflow学习笔记
date: 2017-11-26 20:43:52
category:
    - 机器学习
    - Tensorflow
tags:
    - Tensorflow
---
# 术语

## Tensor —— 张量

TensorFlow 的核心数据单位就是**张量**。

* 张量的定义：**张量**可看做是由原始值组成的任意维度数组。**张量的阶**就是它的维度。
* 张量的**形状（shape）**：这里形状是Tensorflow的专用术语，它同时刻画了张量的维（阶）和每一维的长度。

<!--more-->

## The Computational Graph —— 计算图

**计算图**是一系列 TensorFlow 运算作为节点组成的节点图。

## Node —— 节点

## edge —— 边

## placeholder —— 占位符

为运算时即将到来的某个 Tensor 对象预留位置。placeholder的值无法计算，也就是说，如果试图将它传入Session.run()， 将引发错误。

## constant —— 常量

## Variable —— 变量

* 创建 Valiable 对象

    Tensor 对象和 Op 对象都是不可变的（immutable），而 Variable 被设计用来保存随时间变化的值。
    ```python
    # 为 Variable 对象传入一个初始值3
    tf.Variable（3，name="my_variable"）
    # 2*2 的零矩阵
    tf.zeros([2, 2])
    # 2*2 的全1矩阵
    tf.ones([2, 2])
    # 3*3*3 的张量，其元素服从0~10的均匀分布
    tf.random_uniform([3, 3, 3], minval=0, maxval=10)
    # 3*3*3 的张量，其元素服从0均值、标准差为2的正太分布
    tf.random_normal([3, 3, 3], mean=0.0, stddev=2.0)
    # 为 Tensor 对象不会返回任何小于3.0或大于7.0的值
    tf.truncated_normal([2, 2], mean=5.0, stddev=1.0)
    ```

* 初始化

  * 通常通过将 tf.initialize_all_variables() 操作传给 Session.run() 完成
  * 使用 tf.initialize_variables() 初始化 Variable 对象的一个子集
* 对象的修改

  * Variable.assign() 操作重新赋值
  * Variable.assign_add() 和 Variable.assign_sub() 来实现 Variable 对象的简单的自增和自减

* trainable 参数

对于迭代计数器或者其他不涉及机器学习模型计算的 Variable 对象，通常将设置 trainable 值为 False。

## Session —— 会话

Session 类负责数据流图的执行。其构造方法接收3个可选参数：

    1 target: 指定了所要使用的执行引擎。
    2 graph：指定了将要在 Session 对象中加载的 Graph 对象，默认值为 None，表示使用默认的数据流图。
    3 config：允许用户指定配置 Session 对象所需的选项。

**Session.run()** 方法接收一个参数fethes，以及其他三个可选参数：feed_dict、options 和 run_metadata。
    1 fethes：接收任意的数据流图元素（Op 或 Tensor 对象【执行对象】）; 如为 Tensor 对象，则输出为 NumPy 数组， 如为 Op，则输出为 None。

    2 feed_dict：用于覆盖数据流图中的 Tensor 对象值

## Operation —— 操作

## Graph —— 图

## name scope —— 名称作用域

* 作用：组织数据流图，控制复杂度

* 用法：将 Op 添加到 with tf.name_scope(< name >) 语句块中

```python
import tensorflow as tf

with tf.name_scope("Scope_A"):
    a = tf.add(a, 2, name="A_add")
    b = tf.multiply(a, 3, name="A_mul")

with tf.name_scope("Scope_B"):
    c = tf.add(4, 5, name="B_add")
    d = tf.multiply(c, 6, name="B_mul")

e = tf.add(b, d, name="output")
```
