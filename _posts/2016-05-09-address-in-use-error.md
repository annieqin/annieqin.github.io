---
layout: post
title: 'socket.error: [Errno 48] Address already in use'
description:
category:
tags: [terminal]
---


## 报错原因
曾启动过相同或者类似的服务占用了这个端口，一般来讲，在Mac上直接用Python启动的话，会导致退出不完整，你不能通过点击GUI的“退出”按钮来一步到位，后台的Python进程还是存在的，而它就是一直占用端口不释放的元凶

## 解决办法
找到占用这个端口的进程

查看当前python进程：
```
ps aux | grep python
```

杀掉进程：
```
sudo kill [进程号]
```






