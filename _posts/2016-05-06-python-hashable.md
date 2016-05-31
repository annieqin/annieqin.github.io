---
layout: post
title: 'Python中的可哈希对象'
description:
category:
tags: [python]
---

字典(dictionary)的key 和 集合(set)的元素都必须是可哈希的(hashable)

不可哈希(unhashable)的数据类型：

* list: 可用tule代替

* set: 可用frozenset 代替

* dictionary:  暂时没有较好的替代者

不过可以自己写一个可哈希的字典

```
class hashabledict(dict): 
	def __hash__(self): 
		return hash(tuple(sorted(self.items())))
```

参考：![](http://stackoverflow.com/questions/1306631/python-add-list-to-set)

