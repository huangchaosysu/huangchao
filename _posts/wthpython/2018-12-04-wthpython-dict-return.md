---
layout: single
author_profile: true
title: "wthpython-奇怪的字典和return"
date: 2018-12-04 10:30:53
# toc: true
tags:
  - python
  - 字符串
categories:
  - wtfpython
---

这篇文章是what the fuck python系列的第二篇， 这里我们将介绍关于字典和return的内容.

先来看字典， 示例如下:

```
some_dict = {}
some_dict[5.5] = "Ruby"
some_dict[5.0] = "JavaScript"
some_dict[5] = "Python"
```
Output:
```
>>> some_dict[5.5]
"Ruby"
>>> some_dict[5.0]
"Python"
>>> some_dict[5]
"Python"
```

直观的现象是python把javascript那个数值给删掉了。那么为什么呢？

原理：  
* python的字典的键是依据其hash值来判定两个键是否相等的
* 对于不可变对象来说，如果其值相等， 那么其hash值也是相同的
  ```
  >>> 5 == 5.0  # 5和5.0都是不可变对象
  True
  >>> hash(5) == hash(5.0)
  True
  ```

* 上面的例子中， 因为5和5.0的hash值相同， 所以在执行```some_dict[5] = "Python"```的时候，python就把'javascript'覆盖了。
* 详情参考[这里](https://stackoverflow.com/a/32211042/4354153)


接下来我们看看return语句的一个奇怪现象, 示例:

```
def some_func():
    try:
        return 'from_try'
    finally:
        return 'from_finally'
```
Output:
```
>>> some_func()
'from_finally'
```

原理：  
* 在'try...finally'语句中， 不论执行结果如何， 最后的finally语句都会执行， 包括在try内部执行return, break, continue的情况.
* 函数的返回值是有最后执行的return语句决定的。 因此上例中的返回值永远都是'from_finally'