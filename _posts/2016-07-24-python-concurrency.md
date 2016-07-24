---
layout: post
title: 'Python并发－－Event Loop/Coroutine'
description:
category:
tags: [python, concurrency]
---

# Event Loop
实现并发，可以采用多线程／多进程方式，但是多线程／多进程有可能引起其它问题（如：数据同步，竞争条件，上下文切换的开销等）。
那么除了多线程／进程外，还有哪些实现并发的方式？－－event loop（即利用**IO多路复用**机制）。

首先，我们需要使用一个系统调用去轮询文件描述符（stdin） 查看是否有输入，根据不同的操作系统，有不同的系统调用如：poll, select, kqueue。在Python3.4中，selectors模块封装了这些系统调用的方法。

有了轮询，event loop 的实现就很简单了，在每次循环中，我们都会查看是否有可读事件，如果有便读入并处理。

例子：


```
import selectors
import sys
from time import time
from fib import timed_fib


def process_input(stream):
    text = stream.readline()
    n = int(text.strip())
    print('fib({}) = {}'.format(n, timed_fib(n)))


def print_hello():
    print("{} - Hello world!".format(int(time())))


def main():
    selector = selectors.DefaultSelector()
    # Register the selector to poll for "read" readiness on stdin
    selector.register(sys.stdin, selectors.EVENT_READ)
    last_hello = 0  # Setting to 0 means the timer will start right away
    while True:
        # Wait at most 100 milliseconds for input to be available
        for event, mask in selector.select(0.1):
            process_input(event.fileobj)
        if time() - last_hello > 3:
            last_hello = time()
            print_hello()


if __name__ == '__main__':
    main()
```

输出为

```
$ python3.4 hello_eventloop.py
1412376429 - Hello world!
1412376432 - Hello world!
1412376435 - Hello world!
37
Executing fib took 9.7 seconds.
fib(37) = 24157817
1412376447 - Hello world!
1412376450 - Hello world!
```
如上可见，由于是单线程，"读取并处理输入"事件阻塞了"打印Hello  World"事件。

##  Event  Loop with Callbacks
对于每一个事件类型，都注册一个回调函数。

![image](/img/in-post/concurrent2.png)

例子：

```
from bisect import insort
from collections import namedtuple
from fib import timed_fib
from time import time
import selectors
import sys

Timer = namedtuple('Timer', ['timestamp', 'handler'])

class EventLoop(object):
    """
    Implements a callback based single-threaded event loop as a simple
    demonstration.
    """
    def __init__(self, *tasks):
        self._running = False
        self._stdin_handlers = []
        self._timers = []
        self._selector = selectors.DefaultSelector()
        self._selector.register(sys.stdin, selectors.EVENT_READ)

    def run_forever(self):
        self._running = True
        while self._running:
            # First check for available IO input
            for key, mask in self._selector.select(0):
                line = key.fileobj.readline().strip()
                for callback in self._stdin_handlers:
                    callback(line)

            # Handle timer events
            while self._timers and self._timers[0].timestamp < time():
                handler = self._timers[0].handler
                del self._timers[0]
                handler()

    def add_stdin_handler(self, callback):
        self._stdin_handlers.append(callback)

    def add_timer(self, wait_time, callback):
        timer = Timer(timestamp=time() + wait_time, handler=callback)
        insort(self._timers, timer)

    def stop(self):
        self._running = False


def main():
    loop = EventLoop()

    def on_stdin_input(line):
        if line == 'exit':
            loop.stop()
            return
        n = int(line)
        print("fib({}) = {}".format(n, timed_fib(n)))

    def print_hello():
        print("{} - Hello world!".format(int(time())))
        loop.add_timer(3, print_hello)

    def f(x):
        def g():
            print(x)
        return g

    loop.add_stdin_handler(on_stdin_input)
    loop.add_timer(0, print_hello)
    loop.run_forever()


if __name__ == '__main__':
    main()
```

### IOLoop
```
We use epoll (Linux) or kqueue (BSD and Mac OS X) if they are available, or else we fall back on select(). 
```
IOLoop使用的就是多路复用IO机制，好多项目中都有这一块的封装，原理都差不多，它也是用python实现的基于多种不同操作系统IO多路复用的封装。

Tornado的ioloop也是类似的，记录了一个个文件描述符和handler的pair，每当有io事件发生，就会调用该文件描述符对应的handler。


## Event  Loop with Coroutines

### 生成器
python有yield这个关键字，yield能把一个函数变成一个generator

每当生成器函数碰到yield关键字时就会暂停，与return不同，yield在函数中返回值时会保存函数的状态，使下一次调用函数时会从上一次的状态继续执行，即从yield的下一条语句开始执行

这样做有许多好处，比如我们想要生成一个数列，若该数列的存储空间太大，而我们仅仅需要访问前面几个元素，那么yield就派上用场了，它实现了这种一边循环一边计算的机制，节省了存储空间，提高了运行效率

#### 生成器用处
1#  生成器函数常用于为for循环产生数据

```
def countdown(n):
	while n > 0:
		yield n
		n -= 1
for x in countdown(10):
	print x
```

2# 除了yield一个数据，生成器也能yield控制权
我们可以写一个调度器，在生成器之间循环，一个个执行它们直到碰到yield

一个调度器例子：

```
# 建立一堆任务
def countdown_task(n):
	while n > 0:
		print n
		yield
		n -= 1
		
# 任务列表，每个任务都是一个生成器函数
from collections import deque
tasks = deque([
	countdown_task(5),
	countdown_task(10),
	countdown_task(15)
])

# 现在，开启一个任务调度器
def scheduler(tasks):
	while tasks:
		task = tasks.popleft()
		try:
			next(task) # 运行直到遇到yield
			tasks.append(task) # 重新调度
		except StopIteration:
			pass
			
# 开启
scheduler(tasks)

# 输出
5
10
15
4
9
14
3
8
13
...
```

### 协程

协程是一个能在返回的同时记录下返回时的状态（如：局部变量，下一条指令的地址）的函数。

这个特性使得协程可以被再次调用，即从它上次返回处继续执行。这种形式的返回通常叫做 "yield"。

在Python中，yield关键字可以被用来创建协程。

我们可以用协程来实现异步。当我们需要等待一个异步操作的时候，我们就可以使用yield。

![image](/img/in-post/concurrent1.png)

例子：

```
from event_loop_coroutine import EventLoop
from event_loop_coroutine import print_every
import sys

def fib(n):
    if n <= 1:
        yield n
    else:
        a = yield fib(n - 1)
        b = yield fib(n - 2)
        yield a + b


def read_input(loop):
    while True:
        line = yield sys.stdin
        n = int(line)
        fib_n = yield fib(n)
        print("fib({}) = {}".format(n, fib_n))


def main():
    loop = EventLoop()
    hello_task = print_every('Hello world!', 3)
    fib_task = read_input(loop)
    loop.schedule(hello_task)
    loop.schedule(fib_task)
    loop.run_forever()


if __name__ == '__main__':
    main()
```

输出：

```
$ python3.4 fib_coroutine.py
1412727829 - Hello world!
1412727832 - Hello world!
28
1412727835 - Hello world!
1412727838 - Hello world!
fib(28) = 317811
1412727841 - Hello world!
1412727844 - Hello world!
```
我们需要一个 event loop 来处理协程。与使用回调函数不同的是，在这里，我们需要调度协程来响应IO。即，我们可以用 event loop 维护一个任务队列，每当有输入时，都有一个协程函数需要被执行。对每一个任务，都有一个 stack 变量来跟踪协程栈来一个个执行。






