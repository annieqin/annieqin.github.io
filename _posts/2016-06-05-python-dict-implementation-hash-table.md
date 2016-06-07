---
layout: post
title: 'Hash Table (Dict Implementation in Python)'
description:
category:
tags: [python]
---

Python中字典的内部实现是Hash Table

## Hash Table

#### 字典实际上是一个hash table，而hash table实际上是一个数组

Hash Table实际上是一个数组，数组中存放着键值对(Key Value Pairs)，数组的下标则是由键(Key)通过hash函数算出来的一个整数

一个空字典是一个长度为8的数组(这个数组拥有连续的内存地址)

#### 将键(Key)映射成数组下标(Index)通常经过两步

* **Step 1 预哈希(Prehash)得到哈希码(hashcode):**

  先将键(Key)映射成非负整数
  
  在Python中，hash(x)即是对x的prehash,如：
  
  hash('a') = 12416037344
  
  12416037344就是'a'的hashcode
  
  相同的键对应的hashcode相同
  
* **Step 2  哈希(Hash)得到数组下标(index):**

  将第一步prehash得到的hashcode进一步映射成数组下标
  
  数组下标是［0，N-1］中的一个整数 (其中N是数组长度)
  
#### 字典中用如下方法映射键与数组下标

假设现在用一个大小为**x**的数组来存放键值对，则**键(key)**与**数组下标(index)**的映射关系为：

```
index = hash(key) & mask

(mask = x-1

 x is the size of the array)
 
```


#### 字典是无序的

由于字典实际上是存储在一个hash table中的，因此，输出字典中各元素时，其顺序并不会按照元素输入的顺序，而是按照在hash table中存放的顺序

例如：

```
>>> d = {}

>>> d['ftp'] = 21
>>> d['ssh'] = 22
>>> d['smtp'] = 25
>>> d['www'] = 80
>>> d['time'] = 37
```
此时，字典d对应的hash table如下：

![image](/img/in-post/python-dict-hash-table.png)

```
>>> print d
{'ftp': 21, 'www': 80, 'smtp': 25, 'ssh': 22, 'time': 37}

```



## 解决冲突

#### 冲突

在将键映射成数组下标时，通常会出现冲突，例如：

数组长度是8

hash(‘a’) & 7 = 0  －－ 'a'存储在下标0中

hash('b') & 7 = 3  －－ 'b'存储在下标3中

hash('c') & 7 = 2  －－ 'c'存储在下标2中

hash('z') & 7 = 3  －－ 'z'存储在下标3中，与'b'发生冲突

#### 解决冲突的方法

通常有2种解决冲突的方法：

* **链表法(Seperate Chaining)**：
	当两个键映射的下标冲突时，可用一个链表将这些冲突的键值对存储在同一个下标下，这就是链表的解决办法。	
	但是链表法有可能破坏时间复杂度，使之不能维持在 O(1)。通常认为，当```n / N > 0.75```（n为键值对数目，N为数组长度） 时，应增大数组长度以使时间复杂度保持在O(1)。

* **开放定址法(Open Addressing)**：
	在开放定址的解决办法中，如果有冲突发生，那么就要尝试选择另外的单元，直到找到空的单元为止。

#### 在Python字典的实现中，用到了**开放定址法(Open Addressing)**来解决冲突
	