---
layout: post
title: 'Python 单线程/多线程/多进程 爬虫的比较'
description:
category:
tags: [python]
---

写了一个爬虫爬取拉勾网的数据，用了单线程，多线程，和多进程三种方式
我用这三种方式爬取了部分数据，比较了一下三者的爬取速度，结果如下：

* 单线程爬虫

![image](/img/in-post/sin-thread)

* 多线程爬虫

![image](/img/in-post/multi-thread)

* 多进程爬虫

![image](/img/in-post/multi-process)

可见 单线程用时 > 多线程用时 > 多进程用时

1.多进程爬虫效果 优于 多线程爬虫效果 优于 单线程爬虫效果

2.多线程的效果虽然由于CPython有GIL的原因受到了严重影响，但是对于爬虫这种IO密集型的任务还是有正面效应的
