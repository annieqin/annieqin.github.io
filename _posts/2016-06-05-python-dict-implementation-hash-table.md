---
layout: post
title: 'Hash Table (Dict Implementation in Python)'
description:
category:
tags: [python]
---

Python中字典的内部实现是Hash Table

## Hash Table

Hash Table实际上是一个数组，数组中存放着键值对(Key Value Pairs)，数组的下标则是由键(Key)通过hash函数算出来的一个整数

#### 将键(Key)映射成数组下标(Index)通常经过两步

* **Step 1 预哈希(Prehash)得到哈希码(hashcode):**

  先将键(Key)映射成非负整数
  
  在Python中，hash(x)即是对x的prehash,如：
  
  hash('a') = 12416037344
  
  12416037344就是'a'的hashcode
  
* **Step 2  哈希(Hash)得到数组下标(index):**

  将第一步prehash得到的hashcode进一步映射成数组下标
  
  数组下标是［0，N-1］中的一个整数 (其中N是数组长度)
  
#### Python字典中用如下方法映射键与数组下标

假设现在用一个大小为**x**的数组来存放键值对，则**键(key)**与**数组下标(index)**的映射关系为：

```
index = hash(key) & mask

(mask = x-1

 x is the size of the array)
 
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
	但是链表法有可能破坏时间复杂度，使之不能维持在 O(1)。通常认为，当'''n / N''' （n为键值对数目，N为数组长度）> 0.75 时，应增大数组长度以使时间复杂度保持在O(1)。

* **开放定址法(Open Addressing)**：
	在开放定址的解决办法中，如果有冲突发生，那么就要尝试选择另外的单元，直到找到空的单元为止。

#### 在Python字典的实现中，用到了**开放定址法(Open Addressing)**来解决冲突
	