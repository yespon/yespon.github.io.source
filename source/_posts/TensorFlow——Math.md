---
title: TensorFlow API——Math
date: 2018-02-26 15:03:42
category:
    - 机器学习
tags: 
    - TensorFlow
---
本文主要对tf的一些数学操作方法进行汇总。
<!--more-->

## Math

TensorFlow 将图形定义转换成分布式执行的操作, 以充分利用可用的计算资源(如 CPU 或 GPU。一般你不需要显式指定使用 CPU 还是 GPU, TensorFlow 能自动检测。如果检测到 GPU, TensorFlow 会尽可能地利用找到的第一个 GPU 来执行操作. 并行计算能让代价大的算法计算加速执行，TensorFlow也在实现上对复杂操作进行了有效的改进。大部分核相关的操作都是设备相关的实现，比如GPU。) 下面是一些重要的操作/核：

操作组 | 操作
-|-
Arithmetic Operators | Add, Subtract, Multiply, Scalar_mul, Div, Divide, truediv, floordiv, realdiv, truncatediv, floor_div, truncatemod, floormod, mod, cross
Basic Math Functions | add_n, abs, negative, sign, reciprocal, square, round, sqrt, rsqrt, pow, exp, expm1, log, log1p, ceil, floor, maximum, minimum, cos, sin, lbeta, tan, acos, asin, atan, sinh, asinh, acosh, atanh, lgamma, digamma, erf, erfc, squared_difference, igamma, igammac, zeta, ploygamma, betanic, rint
Matrix Math Functions | diag, diag_part, trace, transpose, eye, matrix_diag, matrix_diag_part, matrix_band_part, matrix_set_diag, matrix_transpose, matmul, norm, matrix_determinant, matrix_inverse, cholesky, cholesky_solve, matrix_solve, matrix_triangular_solve, matrix_solve_ls, qr, self_adjoint_eig, self_adjoint_eigvals, svd
Tensor Math Function | tensordot
Complex Number Functions | complex, conj, imag, angle, real
Reduction | reduce_sum, reduce_prod, reduce_min, reduce_max, reduce_mean, reduce_all, reduce_any, reduce_logsumexp, count_nonzero, accumulate_n, einsum
Scan | cumsum, cumprod
Segmentation | segment_sum, segment_prod, segment_min, segment_max, segment_mean, unsorted_segment_sum, sparse_segment_sum, sparse_segment_mean, sparse_segment_sqrt_n
Sequence Comparison and Indexing | argmin, argmax, setdiff1d, where, unique, edit_distance, invert_permutation

### 算术操作

操作 | 描述
-|-
tf.add | 加法
tf.subtract | 减法
tf.multiply | 乘法
tf.scalar_mul | 固定倍率缩放
tf.div | 
tf.divide | 
tf.truediv | 
tf.floordiv | 
tf.realdiv | 
tf.truncatediv | 
tf.floor_div | 
tf.truncatemod | 
tf.floormod | 
tf.mod | 
tf.cross | 
