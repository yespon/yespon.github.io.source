---
title: TensorFlow-api(1)
date: 2018-01-31 23:24:08
tags:
---
# TensorFlow-api(1)：tf.reduce_mean()这类函数

在tensor的某一维度上，有一类求值的函数，如tf.reduce_max( )，tf.reduce_mean( )，tf.reduce_sum( )
<!--more-->

## tf.reduce_mean( )

* 函数作用：

沿着tensor的某一维度，计算元素的平均值。由于输出tensor的维度比原tensor的低，这类操作也叫降维。

* 参数：

  * reduce_mean(input_tensor,axis=None,keep_dims=False,name=None, reduction_indices=None)
  * input_tensor：需要降维的tensor。
  * axis：axis=none, 求全部元素的平均值；axis=0, 按列降维，求每列平均值；axis=1，按行降维，求每行平均值。
  * keep_dims：若值为True，可多行输出平均值。
  * name：自定义操作的名称。 
  * reduction_indices：axis的旧名，已停用。

* 返回：

降维后的tensor

* 例子：

```python
import tensorflow as tf

x = tf.constant([[1., 2., 3.], [4., 5., 6.]])

with tf.Session() as sess:
    x = sess.run(x)

    mean1 = sess.run(tf.reduce_mean(x))
    mean2 = sess.run(tf.reduce_mean(x, 0))
    mean3 = sess.run(tf.reduce_mean(x, 1))

    print(x)
    print()
    print(mean1)
    print()
    print(mean2)
    print()
    print(mean3)
```

输出

```
[[ 1.  2.  3.]
 [ 4.  5.  6.]]

3.5

[ 2.5  3.5  4.5]

[ 2.  5.]
```

## tf.reduce_sum( )

* 函数作用：

沿着tensor的某一维度，计算元素的和。由于输出tensor的维度比原tensor的低，这类操作也叫降维。

* 参数:

  * reduce_sum(input_tensor, axis=None,keep_dims=False, name=None, reduction_indices=None) 
  * input_tensor：需要降维的tensor。 
  * axis：axis=none, 求全部元素的和；axis=0, 按列降维，求每列元素的和；axis=1，按行降维，求每行元素的和。 
  * keep_dims：若值为True，则用长度为1的tensor形式，输出平均值。 
  * name：自定义操作的名称。 
  * reduction_indices：axis的旧名，已停用。

* 返回： 

降维后的tensor

* 例子：

```python
import tensorflow as tf

x = tf.constant([[1, 1, 1], [1, 1, 1]])                    

# axis = none
reduce_sum = tf.reduce_sum(x)                               

# axis = 0
reduce_sum_axis0 = tf.reduce_sum(x, axis = 0)                
resuce_sum_axis0_axis0 = tf.reduce_sum(reduce_sum_axis0)

# axis = 1
reduce_sum_axis1 = tf.reduce_sum(x, 1)
reduce_sum_axis1_axis0 = tf.reduce_sum(reduce_sum_axis1, 0)

# axis = [0, 1] or axis = [1, 0] 
reduce_sum_axis_01 = tf.reduce_sum(x, [0, 1])
reduce_sum_axis_10 = tf.reduce_sum(x, [1, 0])


# 构建session
with tf.Session() as sess:

    # 运行op定义的运算
    x = sess.run(x)

    result = sess.run(reduce_sum)

    result_0 = sess.run(reduce_sum_axis0)
    result_0_0 = sess.run(resuce_sum_axis0_axis0) 

    result_1 = sess.run(reduce_sum_axis1)
    result_1_0 = sess.run(reduce_sum_axis1_axis0)

    result_01 = sess.run(reduce_sum_axis_01)
    result_10 = sess.run(reduce_sum_axis_10)

    # 输出运算结果
    print(x)
    print()
    print('shape of input_tensor x')
    print(type(x))
    print()

    print('不传入参数axis，默认x reduce到0维')
    print('result =', result)
    print()

    print('传入参数axis = 0')
    print('result_0 =', result_0)
    print()
    print('把 %s 沿0轴，再次reduce' % str(result_0))    
    print('result_0_0 =', result_0_0)
    print()

    print('传入参数axis = 1')
    print('result_1 =', result_1)
    print()
    print('把 %s 沿0轴，再次reduce' % str(result_1))
    print('result_1_0 =', result_1_0)
    print()

    print('输入参数axis = [0, 1]')
    print('result_01 =', result_01)
    print()
    print('输入参数axis = [1, 0]')
    print('result_10 =', result_10)
```

输出

```
[[ 1.  1.  1.]
 [ 1.  1.  1.]]

shape of input_tensor x
<class 'numpy.ndarray'>

不传入参数axis，默认x reduce到0维
result = 6.0

传入参数axis = 0
result_0 = [ 2.  2.  2.]

把 [ 2.  2.  2.] 沿0轴，再次reduce
result_0_0 = 6.0

传入参数axis = 1
result_1 = [ 3.  3.]

把 [ 3.  3.] 沿0轴，再次reduce
result_1_0 = 6.0

输入参数axis = [0, 1]
result_01 = 6.0

输入参数axis = [1, 0]
result_10 = 6.0
```

## tf.reduce_max( )

* 函数作用：

沿着tensor的某一维度，计算元素的最大值。

* 参数： 

同以上函数

* 返回： 

降维后的tensor

## tf.reduce_min( )

* 函数作用：

沿着tensor的某一维度，计算元素的最小值。

* 参数： 

同以上函数

* 返回： 

降维后的tensor

## tf.reduce_prod( )

* 函数作用：

沿着tensor的某一维度，计算输入tensor元素的乘积。

* 参数： 

同以上函数

* 返回： 

降维后的tensor

## tf.reduce_all( )

* 函数作用： 

对tensor中各个元素求逻辑‘与’。

* 参数： 

同以上函数

* 返回： 

降维后的tensor

## tf.reduce_any( )

* 函数作用： 

对tensor中各个元素求逻辑‘或’。

* 参数： 

同以上函数

* 返回： 

降维后的tensor
