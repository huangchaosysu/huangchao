---
layout: single
author_profile: true
title: "python实用技法-04"
date: 2018-10-22 14:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---

需求: 我们想在某个集合中找出最大或最小的N个元素

### 解决方案

heapq模块中有两个函数：nlargest()和nsmallest()
代码：
```
import heapq

nums=[1,444,66,77,34,67,2,6,8,2,4,9,556]
print(heapq.nlargest(3,nums))
print(heapq.nsmallest(3,nums))
```
结果：
```
[556, 444, 77]
[1, 2, 2]
```
这个两个函数都可以接受一个参数key，从而允许他们可以工作在更加复杂的数据结构上：

代码：
```
import heapq

portfolio=[
    {'name':'IBM','shares':100,'price':91.1},
    {'name':'AAPL','shares':50,'price':543.22},
    {'name':'FB','shares':200,'price':21.09},
    {'name':'HPQ','shares':35,'price':31.75},
    {'name':'YHOO','shares':45,'price':16.35},
]

cheap=heapq.nsmallest(3,portfolio,key=lambda s:s['price'])

expensive=heapq.nlargest(3,portfolio,key=lambda s:s['price'])

print(cheap)
print(expensive)
```
结果：
```
[{'name': 'YHOO', 'shares': 45, 'price': 16.35}, {'name': 'FB', 'shares': 200, 'price': 21.09}, {'name': 'HPQ', 'shares': 35, 'price': 31.75}]
[{'name': 'AAPL', 'shares': 50, 'price': 543.22}, {'name': 'IBM', 'shares': 100, 'price': 91.1}, {'name': 'HPQ', 'shares': 35, 'price': 31.75}]
```

如果只是简单的查找最小或者最大的元素（N=1），那么使用min()和max()会更快。
