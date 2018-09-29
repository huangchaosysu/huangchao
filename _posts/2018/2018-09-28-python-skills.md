---
layout: single
author_profile: true
title: "几个比较常用的笔记优雅的python编程技巧"
date: 2018-09-28 15:30:53
toc: true
tags:
  - python
  - 编程技巧
categories:
  - python
---

Python最大的优点之一就是语法简洁，好的代码就像伪代码一样，干净、整洁、一目了然。要写出 Pythonic（优雅的、地道的、整洁的）代码，需要多看多学大牛们写的代码，github 上有很多非常优秀的源代码值得阅读，比如：requests、flask、tornado，下面列举一些常见的Pythonic写法

### 交换赋值

```
##不推荐
temp = a
a = b
b = a  

##推荐下面的方法来交换连个对象的值
a, b = b, a  #  先生成一个元组(tuple)对象，然后unpack
```

### Unpacking(解构)

```
##不推荐
l = ['David', 'Pythonista', '+1-514-555-1234']
first_name = l[0]
last_name = l[1]
phone_number = l[2]  

##推荐
l = ['David', 'Pythonista', '+1-514-555-1234']
first_name, last_name, phone_number = l
# Python 3 Only
another_list = ['David', 'Pythonista', 'backham', '+1-514-555-1234']
first, *middle, last = another_list
```

### 使用in操作服

```
##不推荐写多个if
if fruit == "apple" or fruit == "orange" or fruit == "berry":
    # 多次判断  

##推荐
if fruit in ["apple", "orange", "berry"]:
    # 使用 in 更加简洁
```

### 字符串拼接

```
##不推荐
colors = ['red', 'blue', 'green', 'yellow']

result = ''
for s in colors:
    result += s  #  每次赋值都丢弃以前的字符串对象, 生成一个新对象  

# 推荐
colors = ['red', 'blue', 'green', 'yellow']
result = ''.join(colors)  #  没有额外的内存分配, 性能也更高
```

### 遍历字典

```
##不推荐
for key in my_dict.keys():
    #  my_dict[key] ...  

##推荐
for key in my_dict:
    #  my_dict[key] ...

# 只有当循环中需要更改key值的情况下，我们需要使用 my_dict.keys()
# 生成静态的键值列表。
```

### 字典 get 和 setdefault 方法
```
##不推荐
navs = {}
for (portfolio, equity, position) in data:
    if portfolio not in navs:
            navs[portfolio] = 0
    navs[portfolio] += position * prices[equity]

##推荐, 可以避免exception
navs = {}
for (portfolio, equity, position) in data:
    # 使用 get 方法
    navs[portfolio] = navs.get(portfolio, 0) + position * prices[equity]
    # 或者使用 setdefault 方法
    navs.setdefault(portfolio, 0)
    navs[portfolio] += position * prices[equity]
```

### 遍历列表和索引

```
##不推荐
items = 'zero one two three'.split()
# method 1
i = 0
for item in items:
    print i, item
    i += 1
# method 2
for i in range(len(items)):
    print i, items[i]

##推荐
items = 'zero one two three'.split()
for i, item in enumerate(items):
    print i, item
```

### 列表解析语法

```
##不推荐
new_list = []
for item in a_list:
    if condition(item):
        new_list.append(fn(item))  

##推荐
new_list = [fn(item) for item in a_list if condition(item)]
```

### 多层循环

```
##不推荐
for x in x_list:
    for y in y_list:
        for z in z_list:
            # do something for x &amp;amp; y  

##推荐
from itertools import product
for x, y, z in product(x_list, y_list, z_list):
    # do something for x, y, z
```

### 使用生成器

```
##不推荐
def my_range(n):
    i = 0
    result = []
    while i &amp;lt; n:
        result.append(fn(i))
        i += 1
    return result  #  返回列表

##推荐
def my_range(n):
    i = 0
    result = []
    while i &amp;lt; n:
        yield fn(i)  #  使用生成器代替列表
        i += 1
*尽量用生成器代替列表(尤其是列表长度很长的时候)，除非必须用到列表特有的函数。
```

### 使用any/all函数

```
##不推荐
found = False
for item in a_list:
    if condition(item):
        found = True
        break
if found:
    # do something if found...  

##推荐
if any(condition(item) for item in a_list):
    # do something if found...
```

### 使用Property(属性)

```
##不推荐
class Clock(object):
    def __init__(self):
        self.__hour = 1
    def setHour(self, hour):
        if 25 &amp;gt; hour &amp;gt; 0: self.__hour = hour
        else: raise BadHourException
    def getHour(self):
        return self.__hour

##推荐
class Clock(object):
    def __init__(self):
        self.__hour = 1
    def __setHour(self, hour):
        if 25 &amp;gt; hour &amp;gt; 0: self.__hour = hour
        else: raise BadHourException
    def __getHour(self):
        return self.__hour
    hour = property(__getHour, __setHour)
```

### 使用with语句
```
##不推荐
f = open("some_file.txt")
try:
    data = f.read()
    # 其他文件操作..
finally:
    f.close()

##推荐
with open("some_file.txt") as f:
    data = f.read()
    # 其他文件操作...

## 处理锁
##不推荐
import threading
lock = threading.Lock()

lock.acquire()
try:
    # 互斥操作...
finally:
    lock.release()

##推荐
import threading
lock = threading.Lock()

with lock:
    # 互斥操作...
```