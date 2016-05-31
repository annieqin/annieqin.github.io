---
layout: post
title: "Python中的全局变量"
description:
category:
tags: [mysql]
---

### 外部的不可变数据对象
对于外部的不可变数据对象，你的函数里如果只使用到了它的值，而没有对其赋值的话，就不需要声明global。相反，如果你对其赋了值的话，那么你就需要声明global。声明global的话，就表示你是在向一个全局变量赋值，而不是在向一个局部变量赋值（否则会被认为是在向局部变量赋值）

### 外部的可变数据对象
对于外部的可变数据对象，无论你的函数是使用它还是对它赋值，都不需要声明global

### 全局变量在多线程编程中的应用
在多线程编程中，常常用到“全局变量”来让多线程共享数据，如果是不可变数据对象，则需声明其为global来对其进行操作，如果是可变数据对象，则可直接对其操作，不需声明global，也可将该可变数据对象作为参数传递给线程函数，这些线程将共享该可变数据对象

例

* 当全局变量是不可变数据对象时

```
import threading
import os


def booth(tid):
    global i
    global lock
    while True:
        lock.acquire()
        if i!=0:
            i=i-1
            print(tid, ':now left:', i)
        else:
            print('Thread_id', tid, 'No more tickets')
            os._exit(0)
        lock.release()
i = 100# 不可变全局变量
lock = threading.Lock()# 不可变全局变量
for k in range(10):
    thread = threading.Thread(target=booth,args=(k,))
```

* 当全局变量是可变数据对象时

```
import threading
import os


def booth(tid, monitor):
    while True:
        monitor['lock'].acquire()
        if monitor['ticket']!=0:
            monitor['ticket']=monitor['ticket']-1
            print(tid, ':now left:', monitor['ticket'])
        else:
            print('Thread_id', tid, 'No more tickets')
            os._exit(0)
        monitor['lock'].release()
monitor = {'ticket': 100, 'lock': threading.Lock()}# 可变全局变量
for k in range(10):
    thread = threading.Thread(target=booth,
                              args=(k, monitor)) # 当可变数据对象被传递给函数时，函数所使用的依然是同一个对象，相当于被多个线程共享
                             
```

### 其他一些例子
```
In [1]: a=1

In [2]: def f():
   ...:     print a
   ...:

In [3]: f()
1



In [4]: def f():
   ...:     a=a+1
   ...:     print a
   ...:

In [5]: f()
UnboundLocalError: local variable 'a' referenced before assignment


In [8]: def f(a):
   ...:     a=a+1
   ...:     print(a)
   ...:
In [10]: f(a)
2

In [11]: a
Out[11]: 1
```

                                            

