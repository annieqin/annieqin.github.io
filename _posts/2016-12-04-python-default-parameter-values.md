---
layout: post
title: 'Python中的默认参数值'
description:
category:
tags: [python]
---

# Probelm

今天在写代码时，踩了一个坑，问题复现如下：

```
class ipNode(object):
	def __init__(self, name, children={}):
		self.name = name
		self.children = children
```
以上代码定义了一个ipNode类，其中的实例变量children被赋予了默认值，但是在后续的处理中发现，不同的实例的children竟然是同一个对象，即它们的children的id是一样的。

即

```
ip1 = ipNode("127.0.0.1")
ip2 = ipNode("192.168.1.1")
ip1.chlidren["127.0.0.1 192.168.1.1"] = ip2

print ip1.children["127.0.0.1 192.168.1.1"].name
print ip2.children["127.0.0.1 192.168.1.1"].name

print id(ip2.children)
print id(ip1.children)

output:
192.168.1.1
192.168.1.1

4349524304
4349524304

```
可见，ip1和ip2的children是同一个字典对象，改变ip1的children时，ip2的children也跟着变了。这样，改变任何一个实例的children值就相当于改变了所有实例的children值。

**在所有函数中给可变对象（如dict, list）的参数赋默认值都会遇到这个问题。**

# Reason

出现以上问题的原因是：

参数默认值是在函数定义时（而不是在函数被调用时）被赋值的，且只会被赋值这一次。我们每次操作的其实是同一个对象。

以下是python官方文档中的原话：
> Default parameter values are evaluated from left to right when the function definition is executed. This means that the expression is evaluated once, when the function is defined, and that the same “pre-computed” value is used for each call. This is especially important to understand when a default parameter is a mutable object, such as a list or a dictionary: if the function modifies the object (e.g. by appending an item to a list), the default value is in effect modified. This is generally not what was intended. A way around this is to use None as the default, and explicitly test for it in the body of the function.

**函数在Python中是一级对象。**当函数被定义时，便创建了一个函数对象，包括其中的参数默认值也一同被创建。
像其他的函数对象的属性一样，参数默认值可以通过以下方式获得或改变：

```
>>> function.func_name
'function'
>>> function.func_code
<code object function at 00BEC770, file "<stdin>", line 1>
>>> function.func_defaults
({"127.0.0.1 192.168.1.1": <web.ipNode object at 0x10652d950>},)
>>> function.func_globals
{'function': <function function at 0x00BF1C30>,
'__builtins__': <module '__builtin__' (built-in)>,
'__name__': '__main__', '__doc__': None}

>>> function.func_defaults[0][:] = {}
>>> function()
{}
>>> function.func_defaults
({},)

```



# Solution

解决方式如下：

```
class ipNode(object):
	def __init__(self, name, path, children=None):
		self.name = name
		self.path = path
		self.children = {} if children is None else children
```

