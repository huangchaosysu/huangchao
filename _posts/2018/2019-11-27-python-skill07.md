---
layout: single
author_profile: true
title: "python实用技法-07"
date: 2018-11-27 10:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---

需求: 我们需要将字符串拆分为不同的字段，但是分隔符（以及分隔符之间的空格）在整个字符串中并不一致.

### 解决方案

字符串对象的split()方法只能处理非常简单的情况，而且不支持多个分隔符，对分隔符周围存在的多种空格情况也无能为力。当需要一些更为灵活地功能时，应该使用re.split()方法：
```
import re
line='abc def ; ghi, jkl,mno, pkr'
#分隔符：分号，都逗号，空格符，前后可以跟着任意数量的额外空格
result=re.split(r'\s*[;,\s]\s*',line)
print(result)
```
结果：

    ['abc', 'def', 'ghi', 'jkl', 'mno', 'pkr']

### 分析

re.split()是很有用的，因为可以为分隔符指定多个模式。例如，在上面的解决方案中，分隔符：分号，都逗号，空格符，前后可以跟着任意数量的额外空格。同str.split()得到的结果一样，最终得到的结果是字段列表。

当使用re.split()时，需要小心正则表达式模式中的捕获组是否包含在了括号里。

如果用到了捕获组，那么匹配的文本也会包含在最终结果中。比如，看看下面的案例：
```
import re
line='abc def ; ghi, jkl,mno, pkr'
result=re.split(r'\s*(;|,|\s)\s*',line)
print(result)
```
结果：

    ['abc', ' ', 'def', ';', 'ghi', ',', 'jkl', ',', 'mno', ',', 'pkr']

在特定的上下文中获取到分隔符也可能是有用的。例如，用分隔字符来改进字符串的输出：
```
import re
line='abc def ; ghi, jkl,mno, pkr'
result=re.split(r'\s*(;|,|\s)\s*',line)

values=result[::2]
delimiters=result[1::2]+['']

print(values)
print(delimiters)

last=''.join(v+d for v,d in zip(values,delimiters))
print(last)
```
结果：
```
['abc', 'def', 'ghi', 'jkl', 'mno', 'pkr']
[' ', ';', ',', ',', ',', '']
abc def;ghi,jkl,mno,pkr
```
如果不想在结果中看到分隔字符，但仍然想用符号来对正则表达式模式进行分组，请确保用的是非捕获组，以(?:...)的形式指定。示例如下：
```
import re
line='abc def ; ghi, jkl,mno, pkr'
result=re.split(r'\s*(?:;|,|\s)\s*',line)

print(result)
```
结果：

    ['abc', 'def', 'ghi', 'jkl', 'mno', 'pkr']