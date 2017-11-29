---
title: TensorFlow学习——TensorFlow Core
date: 2017-11-27 19:17:04
category:
    - 机器学习
    - Tensorflow
tags:
    - Tensorflow
---
# Tensorflow core

## tf.train API

* optimizers —— 优化器

TensorFlow 提供了**优化器**来缓慢地更改每个变量，从而最大程度的降低损耗函数。
eg. gradient descent:

```pthon
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)
```

训练线性回归模型完整的代码：

```python
# 导入tensorflow
import tensorflow as tf

# 模型参数
W = tf.Variable([.3], dtype=tf.float32)
b = tf.Variable([-.3], dtype=tf.float32)
# 模型输入和输出
x = tf.placeholder(tf.float32)
# 定义线性模型
linear_model = W*x + b
y = tf.placeholder(tf.float32)

# 损耗
loss = tf.reduce_sum(tf.square(linear_model - y)) # sum of the squares
# 优化器：梯度下降
optimizer = tf.train.GradientDescentOptimizer(0.01)
train = optimizer.minimize(loss)

# 训练集数据
x_train = [1, 2, 3, 4]
y_train = [0, -1, -2, -3]
# 训练循环
init = tf.global_variables_initializer()
sess = tf.Session()
sess.run(init) # reset values to wrong
for i in range(1000):
  sess.run(train, {x: x_train, y: y_train})

# 计算训练的准确度
curr_W, curr_b, curr_loss = sess.run([W, b, loss], {x: x_train, y: y_train})
print("W: %s b: %s loss: %s"%(curr_W, curr_b, curr_loss))
```

## tf.estimator

tf.estimator 作为 TensorFlow 高级库，简化了机器学习的机制，其中包括：

* 运行训练循环
* 运行评估循环
* 管理数据集

tf.estimator 定义了徐福哦常见的模型

