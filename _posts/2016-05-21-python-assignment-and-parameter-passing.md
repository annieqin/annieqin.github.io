---
layout: post
title: 'Python中的变量赋值和函数参数传递特性'
description:
category:
tags: [python]
---


>动态特性：引用与对象分离

>python会缓存整数和短小字符

## 变量赋值
在其他语言中，对一个变量赋值可以看作是把对象放到该变量所代表的盒子中（即将对象放到该变量所申请的内存空间中），如 a=1：
![image](/img/in-post/blog1.1.jpeg)
此时，盒子"a"装有整数1

当把另外一个对象赋值给同一个变量，将会替换到盒子中原来的内容（即将另一个对象放到原变量申请的内存空间中（新对象替换旧对象）），
如 a=2:
![image](/img/in-post/blog1.2.jpeg)
此时盒子"a"中装有整数2

两个变量间的赋值将会生成目标对象的副本，并将它放入新盒子中（即复制原变量内存空间中的对象，放到新变量申请的内存空间中），
如 int b=a:
![image](/img/in-post/blog1.3.jpeg)
盒子a和b中分别装着两个数值相同的不同对象

在Python中，变量是指向真正对象的引用，即变量像是套在对象上的标签，
如 a=1:
![image](/img/in-post/blog1.4.jpeg)
对象“1”拥有一个名为“a”的标签（即引用"a"指向对象"1"）

如果重新给“a”赋值，
如 a=2:
![image](/img/in-post/blog1.5.jpeg)
标签“a”从“1”上移到了“2”上（即引用"a"指向另一个对象"2"）
此时，没有引用指向对象"1"，"1"将被内存释放

如果将一个变量赋值给另一个变量，则是将被赋值变量标签与赋值变量标签套在同一个对象身上（即引用"a"和"b"指向同一个对象）
如 b=a:
![image](/img/in-post/blog1.6.jpeg)

## 函数参数传值
Python中的函数形参并不会生成实参的副本，而是形参与实参指向同一个对象

## 一些例子
```
def main():
     n=1
     x=[0,1,2,3]
     f(n,x)
     print id(n),id(x)
     print n

def f(n, x):
    n = 2
    x.append(4)
    print id(n),id(x)
    print n
    
>>>main()
140320226469472 4372006744
2
140320226469496 4372006744
1

def main():
    n=1
    f(n)
    print id(n)
    print n

def f(n):
    print id(n)
    n=n+1
    print id(n)
    print n

>>>main()
140320226469496
140320226469472
2
140320226469496
1

def list_changer(input_list): 
	input_list[0] = 10 
	input_list = range(1, 10) 
	print(input_list) 
	input_list[0] = 10 
	print(input_list) 
>>> test_list = [5, 5, 5] 
>>> list_changer(test_list) 
[1, 2, 3, 4, 5, 6, 7, 8, 9] [10, 2, 3, 4, 5, 6, 7, 8, 9] 
>>> print test_list 
[10, 5, 5]
```

## Python会缓存整数和短小字符
```
a=1
b=1
print(a is b)

True

a='hi'
b='hi'
print(a is b)

True

a='hi everyone welcome here'
b='hi everyone welcome here'
print(a is b)

False

a=[]
b=[]
print(a is b)

False
```



