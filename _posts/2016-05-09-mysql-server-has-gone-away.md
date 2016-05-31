---
layout: post
title: "OperationalError: (2006, 'MySQL server has gone away')"
description:
category:
tags: [mysql]
---

## 解决办法
找到my.cnf文件
我的my.cnf文件在：
```
/usr/local/opt/mysql
```

将max_allowed_packet值增大：
```
[mysqld]
max_allowed_packet=256M
```
