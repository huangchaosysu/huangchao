---
layout: single
author_profile: true
title: "wthpython-奇怪的字符串"
date: 2018-11-30 10:30:53
# toc: true
tags:
  - python
  - 字符串
categories:
  - wtfpython
---

这篇文章是what the fuck python的第一篇， 在这个系列里面， 会给大家解释一些比较令人困惑的python现象或者比较少用到的python特性。 这篇文章里，献给大家介绍一个python字符串的冷门知识.

先来看几个例子:

#### example 01:
```
>>> a = "some_string"
>>> id(a)
140420665652016
>>> id("some" + "_" + "string") # Notice that both the ids are same.
140420665652016
```

#### example 02
```
>>> a = "wtf"
>>> b = "wtf"
>>> a is b
True

>>> a = "wtf!"
>>> b = "wtf!"
>>> a is b
False

>>> a, b = "wtf!", "wtf!"
>>> a is b
True
```

#### example 03
```
>>> 'a' * 20 is 'aaaaaaaaaaaaaaaaaaaa'
True
>>> 'a' * 21 is 'aaaaaaaaaaaaaaaaaaaaa'
False
```

有没有很神奇， 是不是很奇怪。为什么会这样呢？

原理解释:

* CPython会对字符串类型进行一种优化(string interning, 姑且叫做字符串驻留吧).其原理就是使用已经存在的不可变对象来代替创建一个新的对象的行为。在python里， 不可变对象在发生改变时会创建一个新的对象， 而这种优化机制就是使用一个已有的对象来替代这个机制。这种机制的结果就是， 多个对象引用的是内存中的同一块区域（这么做可以节省内存）。
* 在上面的代码中， 这些字符串就被隐式“驻留”了。那么在什么情况下字符串才会被驻留与python解释器的实现高度相关。不同的解释器是不同的。但是我们可以根据一些原则来猜测字符串是否被"驻留"
    * 所有长度为0或者1的字符串会被“驻留”
    * 字符串的驻留发生在编译时(py -> pyc), ("wtf"会， ''.join(['a', 'b', 'c'])就不会)
    * 含有ASCII字母（a-zA-Z），数字以及下划线以外的字符的字符串不会被“驻留”。这一条规则解释了'wtf!'为什么没有被驻留(example 02的第二个比较), 具体的cpython实现参考[这里](https://github.com/python/cpython/blob/3.6/Objects/codeobject.c#L19)
    ![](/assets/images/wtfpython/01.png)

    * 当我们在同一行里给把a，b都赋值为'wtf!'的时候，python解释器会创建一个新的对象， 并且把第二个变量也指向这个对象。这个时候解释器知道了已经有一个'wtf!'存在了。但是， 如果我们不在同一行里给a, b分别赋值'wtf!'， 由于'wtf!'没有驻留行为，解释器不知道已经有一个'wtf!'存在， 因此就会创建一个新的对象.
    * 对于'a'*20这种语法(constant folding), 如果长度不超过20, 那么编译时（py->pyc）会得到'aaaaaaaaaaaaaaaaaaaa'这个字符串， 但是如果长度超过20, 那么对于这个字符串的值的解析就要等到运行时， 因此有了example 03