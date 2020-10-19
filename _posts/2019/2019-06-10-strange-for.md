---
layout: single
author_profile: true
title: "wtfpython-For语句的奇特作用"
date: 2019-06-10 10:30:53
# toc: true
tags:
  - python
  - python for
categories:
  - wtfpython
---

这篇文章是what the fuck python系列的第三篇， 这里我们将介绍关于For语句的一个奇特作用.

示例如下:

```
some_string = "wtf"
some_dict = {}
for i, some_dict[i] in enumerate(some_string):
    pass
```
Output:
```
>>> some_dict # An indexed dict is created.
{0: 'w', 1: 't', 2: 'f'}
```

原理：  
python语法对For语句的定义如下
```for_stmt: 'for' exprlist 'in' testlist ':' suite ['else' ':' suite]```

这里 exprlist 就是赋值对象. 对于testlist中的每一个元素， {exprlist} = {next_value}都会被执行一遍. 本例中就是对于some_string的每一个字符k，都执行了一遍some_dict[i]=k, 因此这里我们看到的现象就是字典被填充了数据。
