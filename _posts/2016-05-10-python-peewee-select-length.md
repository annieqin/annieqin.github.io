---
layout: post
title: 'Python Peewee Select Length'
description:
category:
tags: [python]
---

参考：[http://stackoverflow.com/questions/31694616/how-to-get-peewee-result-length](http://stackoverflow.com/questions/31694616/how-to-get-peewee-result-length)

result = Job.select().where(Job.position_name == 'python')

result_num = result.count()
vs
result_num = len(list(result))
?