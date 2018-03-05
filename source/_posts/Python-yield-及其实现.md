---
title: Python yield 及其实现
date: 2018-02-06 20:15:15
category:
    - Python
tags: 
    - yield 产生器 生成器
---
# Python yield及其实现

刚开始接触python时，解接触到了 yield 关键字，在实际使用中，越来越觉得其用处的强大，遂感觉需整理一下自己的理解，做一个总结。yield 的功能类似于 return，但不同之处在于它返回的是生成器。
<!--more-->

## 生成器

生成器(generator)是通过一个或多个 yield 表达式构成的函数，每一个生成器都是一个迭代器（但是迭代器不一定是生成器）。

如果一个函数包含 yield 关键字，这个函数就会变为一个生成器。

生成器并不会一次返回所有结果，而是每次遇到 yield 关键字后返回相应结果，并保留函数当前的运行状态，等待下一次的调用。

由于生成器也是一个迭代器，那么它就应该支持 next 方法来获取下一个值。

## 基本操作

```python
# 通过`yield`来创建生成器
def func():
   for i in xrange(10);
        yield i
 
# 通过列表来创建生成器
[i for i in xrange(10)]

# 调用如下
>>> f = func()
>>> f # 此时生成器还没有运行
<generator object func at 0x7fe01a853820>
>>> f.__next__() # 当i=0时，遇到yield关键字，直接返回
0
>>> f.__next__() # 继续上一次执行的位置，进入下一层循环
1
...
>>> f.__next__()
9
>>> f.__next__() # 当执行完最后一次循环后，结束yield语句，生成StopIteration异常
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
StopIteration
>>>
```

除了__next__函数，生成器还支持send函数。该函数可以向生成器传递参数。

```python
>>> def func():
...     n = 0
...     while 1:
...         n = yield n #可以通过send函数向n赋值
...
>>> f = func()
>>> f.__next__() # 默认情况下n为0
0
>>> f.send(1) #n赋值1
1
>>> f.send(2)
2
>>>
```

## 应用

最经典的例子，生成无限序列。

常规的解决方法是，生成一个满足要求的很大的列表，这个列表需要保存在内存中，很明显内存限制了这个问题。

```python
def get_primes(start):
    for element in magical_infinite_range(start):
        if is_prime(element):
            return element
```

如果使用生成器就不需要返回整个列表，每次都只是返回一个数据，避免了内存的限制问题。

```python
def get_primes(number):
    while True:
        if is_prime(number):
            yield number
        number += 1
```

## 生成器源码分析

生成器的源码在Objects/genobject.c。

### 调用栈

在解释生成器之前，需要讲解一下Python虚拟机的调用原理。

Python虚拟机有一个栈帧的调用栈，其中栈帧的是PyFrameObject，位于Include/frameobject.h。

```c
typedef struct _frame {
    PyObject_VAR_HEAD
    struct _frame *f_back;  /* previous frame, or NULL */
    PyCodeObject *f_code;   /* code segment */
    PyObject *f_builtins;   /* builtin symbol table (PyDictObject) */
    PyObject *f_globals;    /* global symbol table (PyDictObject) */
    PyObject *f_locals;     /* local symbol table (any mapping) */
    PyObject **f_valuestack;    /* points after the last local */
    /* Next free slot in f_valuestack.  Frame creation sets to f_valuestack.
       Frame evaluation usually NULLs it, but a frame that yields sets it
       to the current stack top. */
    PyObject **f_stacktop;
    PyObject *f_trace;      /* Trace function */
 
    /* If an exception is raised in this frame, the next three are used to
     * record the exception info (if any) originally in the thread state.  See
     * comments before set_exc_info() -- it's not obvious.
     * Invariant:  if _type is NULL, then so are _value and _traceback.
     * Desired invariant:  all three are NULL, or all three are non-NULL.  That
     * one isn't currently true, but "should be".
     */
    PyObject *f_exc_type, *f_exc_value, *f_exc_traceback;
 
    PyThreadState *f_tstate;
    int f_lasti;        /* Last instruction if called */
    /* Call PyFrame_GetLineNumber() instead of reading this field
       directly.  As of 2.3 f_lineno is only valid when tracing is
       active (i.e. when f_trace is set).  At other times we use
       PyCode_Addr2Line to calculate the line from the current
       bytecode index. */
    int f_lineno;       /* Current line number */
    int f_iblock;       /* index in f_blockstack */
    PyTryBlock f_blockstack[CO_MAXBLOCKS]; /* for try and loop blocks */
    PyObject *f_localsplus[1];  /* locals+stack, dynamically sized */
} PyFrameObject;
```

栈帧保存了给出代码的的信息和上下文，其中包含最后执行的指令，全局和局部命名空间，异常状态等信息。f_valueblock保存了数据，b_blockstack保存了异常和循环控制方法。

举一个例子来说明，

```c
def foo():
    x = 1
    def bar(y):
        z = y + 2  #
```

那么，相应的调用栈如下，一个py文件，一个类，一个函数都是一个代码块，对应者一个Frame，保存着上下文环境以及字节码指令。

```
c   ---------------------------
a  | bar Frame                 | -> block stack: []
l  |     (newest)              | -> data stack: [1, 2]
l   ---------------------------
   | foo Frame                 | -> block stack: []
s  |                           | -> data stack: [.bar at 0x10d389680>, 1]
t   ---------------------------
a  | main (module) Frame       | -> block stack: []
c  |       (oldest)            | -> data stack: []
k   ---------------------------
```

每一个栈帧都拥有自己的数据栈和block栈，独立的数据栈和block栈使得解释器可以中断和恢复栈帧（生成器正式利用这点）。

Python代码首先被编译为字节码，再由Python虚拟机来执行。一般来说，一条Python语句对应着多条字节码（由于每条字节码对应着一条C语句，而不是一个机器指令，所以不能按照字节码的数量来判断代码性能）。

调用dis模块可以分析字节码，

```
from dis import dis
 
dis(foo)
 
  5           0 LOAD_CONST               1 (1) # 加载常量1
              3 STORE_FAST               0 (x) # x赋值为1
 
  6           6 LOAD_CONST               2 (<code>) # 加载常量2
              9 MAKE_FUNCTION            0 # 创建函数
             12 STORE_FAST               1 (bar)
 
  9          15 LOAD_FAST                1 (bar)
             18 LOAD_FAST                0 (x)
             21 CALL_FUNCTION            1  # 调用函数
             24 RETURN_VALUE        </code>
```

其中，

1. 第一行为代码行号；
2. 第二行为偏移地址；
3. 第三行为字节码指令；
4. 第四行为指令参数；
5. 第五行为参数解释。

由了上面对于调用栈的理解，就可以很容易的明白生成器的具体实现。

### 生成器的创建

```c
PyObject *
PyGen_New(PyFrameObject *f)
{
    PyGenObject *gen = PyObject_GC_New(PyGenObject, &PyGen_Type); # 创建生成器对象
    if (gen == NULL) {
        Py_DECREF(f);
        return NULL;
    }
    gen->gi_frame = f; # 赋予代码块
    Py_INCREF(f->f_code); # 引用计数+1
    gen->gi_code = (PyObject *)(f->f_code);
    gen->gi_running = 0; # 0表示为执行，也就是生成器的初始状态
    gen->gi_weakreflist = NULL;
    _PyObject_GC_TRACK(gen); # GC跟踪
    return (PyObject *)gen;
}
```

### send与next

next与send函数，如下

```c
static PyObject *
gen_iternext(PyGenObject *gen)
{
    return gen_send_ex(gen, NULL, 0);
}
 
 
static PyObject *
gen_send(PyGenObject *gen, PyObject *arg)
{
    return gen_send_ex(gen, arg, 0);
}
```

从上面的代码中可以看到，send和next都是调用的同一函数gen_send_ex，区别在于是否带有参数。

```c
static PyObject *
gen_send_ex(PyGenObject *gen, PyObject *arg, int exc)
{
    PyThreadState *tstate = PyThreadState_GET();
    PyFrameObject *f = gen->gi_frame;
    PyObject *result;
 
    if (gen->gi_running) { # 判断生成器是否已经运行
        PyErr_SetString(PyExc_ValueError,
                        "generator already executing");
        return NULL;
    }
    if (f==NULL || f->f_stacktop == NULL) { # 如果代码块为空或调用栈为空，则抛出StopIteration异常
        /* Only set exception if called from send() */
        if (arg && !exc)
            PyErr_SetNone(PyExc_StopIteration);
        return NULL;
    }
 
    if (f->f_lasti == -1) { # f_lasti=1 代表首次执行
        if (arg && arg != Py_None) { # 首次执行不允许带有参数
            PyErr_SetString(PyExc_TypeError,
                            "can't send non-None value to a "
                            "just-started generator");
            return NULL;
        }
    } else {
        /* Push arg onto the frame's value stack */
        result = arg ? arg : Py_None;
        Py_INCREF(result); # 该参数引用计数+1
        *(f->f_stacktop++) = result; # 参数压栈
    }
 
    /* Generators always return to their most recent caller, not
     * necessarily their creator. */
    f->f_tstate = tstate;
    Py_XINCREF(tstate->frame);
    assert(f->f_back == NULL);
    f->f_back = tstate->frame;
 
    gen->gi_running = 1; # 修改生成器执行状态
    result = PyEval_EvalFrameEx(f, exc); # 执行字节码
    gen->gi_running = 0; # 恢复为未执行状态
 
    /* Don't keep the reference to f_back any longer than necessary.  It
     * may keep a chain of frames alive or it could create a reference
     * cycle. */
    assert(f->f_back == tstate->frame);
    Py_CLEAR(f->f_back);
    /* Clear the borrowed reference to the thread state */
    f->f_tstate = NULL;
 
    /* If the generator just returned (as opposed to yielding), signal
     * that the generator is exhausted. */
    if (result == Py_None && f->f_stacktop == NULL) {
        Py_DECREF(result);
        result = NULL;
        /* Set exception if not called by gen_iternext() */
        if (arg)
            PyErr_SetNone(PyExc_StopIteration);
    }
 
    if (!result || f->f_stacktop == NULL) {
        /* generator can't be rerun, so release the frame */
        Py_DECREF(f);
        gen->gi_frame = NULL;
    }
 
    return result;
}
```

### 字节码的执行

PyEval_EvalFrameEx函数的功能为执行字节码并返回结果。

```c
# 主要流程如下，
for (;;) {
   switch(opcode) { # opcode为操作码，对应着各种操作
        case NOP:
            goto  fast_next_opcode;
        ...
        ...
        case YIELD_VALUE: # 如果操作码是yield
            retval = POP();
            f->f_stacktop = stack_pointer;
            why = WHY_YIELD;
            goto fast_yield; # 利用goto跳出循环
    }
}
 
fast_yield:
    ...
return vetval; # 返回结果
```

举一个例子，f_back上一个Frame，f_lasti上一次执行的指令的偏移量，

```python
import sys
from dis import dis
 
 
def func():
    f = sys._getframe(0)
    print f.f_lasti
    print f.f_back
    yield 1
 
    print f.f_lasti
    print f.f_back
    yield 2
 
 
a = func()
dis(func)
a.__next__()
a.__next__()

```

结果如下，其中第三行的英文为操作码，对应着上面的opcode，每次switch都是在不同的opcode之间进行选择。

```python
6           0 LOAD_GLOBAL              0 (sys)
              3 LOAD_ATTR                1 (_getframe)
              6 LOAD_CONST               1 (0)
              9 CALL_FUNCTION            1
             12 STORE_FAST               0 (f)
 
  7          15 LOAD_FAST                0 (f)
             18 LOAD_ATTR                2 (f_lasti)
             21 PRINT_ITEM          
             22 PRINT_NEWLINE      
 
  8          23 LOAD_FAST                0 (f)
             26 LOAD_ATTR                3 (f_back)
             29 PRINT_ITEM          
             30 PRINT_NEWLINE      
 
  9          31 LOAD_CONST               2 (1)
             34 YIELD_VALUE     # 此时操作码为YIELD_VALUE，直接跳转上述goto语句，此时f_lasti为当前指令，f_back为当前frame
             35 POP_TOP            
 
11          36 LOAD_FAST                0 (f)
             39 LOAD_ATTR                2 (f_lasti)
             42 PRINT_ITEM          
             43 PRINT_NEWLINE      
 
12          44 LOAD_FAST                0 (f)
             47 LOAD_ATTR                3 (f_back)
             50 PRINT_ITEM          
             51 PRINT_NEWLINE      
 
13          52 LOAD_CONST               3 (2)
             55 YIELD_VALUE        
             56 POP_TOP            
             57 LOAD_CONST               0 (None)
             60 RETURN_VALUE        
18
<frame object at 0x7fa75fcebc20> #和下面的frame相同，属于同一个frame，也就是说在同一个函数（命名空间）内，frame是同一个。
39
<frame object at 0x7fa75fcebc20>
```