---
layout: single
author_profile: true
title: "单例(Singleton)模式"
date: 2018-06-13 16:01:52
tags:
  - 设计模式
  - Singleton
categories:
  - 设计模式
---

首先来明确一个问题，那就是在某些情况下，有些对象，我们只需要一个就可以了，比如，一台计算机上可以连好几个打印机，但是这个计算机上的打印程序只能有一个，这里就可以通过单例模式来避免两个打印作业同时输出到打印机中，即在整个的打印过程中我只有一个打印程序的实例。  
简单说来，单例模式（也叫单件模式）的作用就是保证在整个应用程序的生命周期中，任何一个时刻，单例类的实例都只存在一个（当然也可以不存在）。
单例模式的要点有三个：
* 一是某个类只能有一个实例；
* 二是它必须自行创建这个实例；
* 三是它必须自行向整个系统提供这个实例。单例模式是一种对象创建型模式。单例模式又名单件模式或单态模式。

下面来看单例模式的UML示意图:  
![](/assets/images/design_pattern/singleton)


Python代码实现
```
#!/usr/bin/python
#coding:utf8
'''
Singleton
'''

class Singleton(object):
    ''''' A python style singleton '''

    def __new__(cls, *args, **kw):
        if not hasattr(cls, '_instance'):
            org = super(Singleton, cls)
            cls._instance = org.__new__(cls, *args, **kw)
            # cls._instance = object.__new__(cls, *args, **kw)
        return cls._instance


if __name__ == '__main__':
    class SingleSpam(Singleton):
        def __init__(self, s):
            self.s = s

        def __str__(self):
            return self.s

    s1 = SingleSpam('spam')
    print id(s1), s1
    s2 = SingleSpam('spa')
    print id(s2), s2
    print id(s1), s1

# 运行结果
140261440784976 spam
140261440784976 spa
140261440784976 spa
```

从上面的结果可以看出来，尽管我实例化了两次SingleSpam，但是s1和s2实际上是同一个实例对象，原理就是利用了python语言的一个特殊方法__new__()。

单例模式的优点  
* 提供了对唯一实例的受控访问。因为单例类封装了它的唯一实例，所以它可以严格控制客户怎样以及何时访问它，并为设计及开发团队提供了共享的概念。
* 由于在系统内存中只存在一个对象，因此可以节约系统资源，对于一些需要频繁创建和销毁的对象，单例模式无疑可以提高系统的性能。
* 允许可变数目的实例。我们可以基于单例模式进行扩展，使用与单例控制相似的方法来获得指定个数的对象实例。

模式适用环境
在以下情况下可以使用单例模式：
* 系统只需要一个实例对象，如系统要求提供一个唯一的序列号生成器，或者需要考虑资源消耗太大而只允许创建一个对象。
* 客户调用类的单个实例只允许使用一个公共访问点，除了该公共访问点，不能通过其他途径访问该实例。
