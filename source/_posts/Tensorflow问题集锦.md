---
title: Tensorflow问题集锦
date: 2017-11-29 15:21:55
category:
    - 机器学习
    - Tensorflow
tags:
    - Tensorflow
---
# Tensorflow问题集锦

## couldn’t open CUDA library cupti64_80.dll

问题一般出现在：Win10 TensorFlow（gpu）安装过程
<!--more-->
**原因：**在资源管理器中查询 cupti64_80.dll 的位置。如对于 windows 用户而言，如果将 nvidia 的显卡驱动安装在默认位置，该 dll 文件的路径在：

```cmd
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\extras\CUPTI\libx64
```

一种简单直接的方法即是不放将该路径下的文件全部复制到：

```cmd
C:\Program Files\NVIDIA GPU Computing Toolkit\CUDA\v8.0\bin
```

bin 目录下。

## TensorFlow: InternalError: Blas SGEMM launch failed

此问题会出现在执行 tf.matmul(a, b) 时，出现此问题的原因在于此时应当在多个应用程序中运行着其他的 Interactive Session，将它们关闭就好了。

或者在运行出异常的程序之前，添加如下代码：

```python
if 'session' in locals() and session is not None:
    print('Close interactive session')
    session.close()
```
