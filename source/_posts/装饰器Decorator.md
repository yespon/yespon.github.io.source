---
title: Python学习——装饰器
date: 2017-11-07 13:29:44
category:
    - Python
tags:
    - 装饰器
    - decorator
---
在学习Pthon过程中，装饰器算是一个蛮难理解的问题（对我来说，尤其是其效果看起来甚像Java中的AOP切面的织入）。

AOP采用的是Proxy模式，再利用反射机制。

Python中，**函数** 也是一个 
**对象**
---
要理解装饰器，首先必须知道，在python中，函数也是对象，而且函数对象可以被赋值给变量，所以，通过变量也能调用该函数。

让我们用一个sample example来理解：

```
def shout():
    print("Yes!")
    
shout()
# outputs: 'Yes!'

# 作为对象，可以将函数赋值给另一个对象
scream = shout

# ！这里并没有使用括号：因为这里我们并不是调用函数，
# 而是将函数‘shout’赋值给‘scream’ 这意味着，
# 可以通过‘scream’ 调用 ‘shout’

scream()
# outputs: 'Yes!'

# 不仅如此，你甚至可以删除‘shout’，但是通过‘scream’依旧可以访问原有函数

del shout
try:
    shout()
except NameError, e:
    print e
# outputs: name 'shout' is not defined

scream()
# outputs: 'Yes!'

```

装饰模式有很多经典的使用场景，例如：插入日志，性能测试，事务处理等等。有了装饰器，我们就可以提取大量函数中与函数本身功能无关的代码，从而达到代码重用的目的。

**一个简单的需求**
现在，假设我们要增强shout()函数的功能，在调用shout()函数前后自动打印日志，但是我们不希望修改函数。这种在代码运行期间动态增加功能的方式，称之为“装饰器”（Decorator）。

本质上，decorator就是一个返回函数的高阶函数。所以，我们要定义一个能打印日志的decorator，可以定义如下：

```
def log(func): # 接受一个函数作为参数
    def wrapper(*args, **kw): # wrapper()函数的参数定义是(*args, **kw)，因此，wrapper()函数可以接受任意参数的调用
        print('start call %s(): ' % func.__name__)
        func(*args, **kw)
        print('end call %s(): ' % func.__name__)
    return wrapper # 同时，返回一个参数
```

**装饰器语法糖**

在Python中，可以使用“@”语法糖（把decorator置于函数定义处）来精简装饰器的代码：

```
@log
def shout():
    print("Yes!")
```
调用shout（）函数，不仅会运行shout（）函数本身，还会在运行函数shout前后各打印一行日志：
```
>>>shout()
start call shout():
Yes!
end call shout():
```
使用了“@”语法糖后，我们就不需要额外的代码来给“shout”重新赋值了，其实，@log的本质就是
```
shout=log（shout）
```
