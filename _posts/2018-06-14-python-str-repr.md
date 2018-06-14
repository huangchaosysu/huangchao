---
layout: single
author_profile: true
title: "Python__str__和__repr__的区别"
excerpt: " Python__str__ __repr__的区别"
date: 2018-06-14 15:11:52
tags:
  - Python
categories:
  - Python文章
# toc: true
---

\_\_str_\_和\_\_repr\_\_的区别
这是python中两个magic method，很容易让新手迷糊，因为很多时候，二者的实现是一样的，但是这两个函数是用在不同的地方
\_\_str\_\_， 主要是用于展示，str(obj)或者print(obj)的时候调用，返回值一定是一个str对象
\_\_repr\_\_， 是被repr(obj)， 或者在终端直接打obj的时候调用
```
>>> us = u'严'  # __repr__
>>> us
u'\u4e25'
>>> print(us)  # __str__
严
```
可以看到，不使用print返回的是一个更能反映对象本质的结果，即us是一个unicode对象（最前面的u表示，以及unicode编码是用的u），且“严”的unicode编码确实是4E25。而print调用可us.__str__，等价于print str(us)，使得结果对用户更友好。

说白了，__str__是给人看的，__repr__是给机器看的。