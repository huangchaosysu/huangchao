---
layout: single
author_profile: true
title: "python实用技法-05"
date: 2018-10-29 10:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---

需求: 我们有一个元素序列，想知道在序列中出现次数最多的元素是什么？

### 解决方案

collections模块中国的Counter类正是为此类问题而设计的。它甚至有一个非常方便的most_common()方法可以告诉我们答案。可以给Counter对象提供任何可哈希的对象序列作为输入。

假设一个列表，其中有一些列的单词，我们想找出哪些单词出现的最为频繁

```
from collections import Counter
words=[
'a','b','c','d','e','f',
'a','b','c','d','e','f',
'a','b','c',
'a','b',
'a'
]
#利用Counter统计每个元素出现的个数
words_counts=Counter(words)
#出现次数最多的3个元素
top_three=words_counts.most_common(3)
#返回元素和出现次数
print(top_three)

#Counter底层是一个字典，可以在元素和他们出现的次数之间做映射，例如：
#输出元素【f】出现的次数
print(words_counts['f'])

#如果想手动增加计数个数，只需要简单的自增
words_counts['f']+=1
print(words_counts['f'])

#如果想手动增加计数个数,还可以使用update()方法：
#只针对元素【f】增加一次计数
words_counts.update('f')
print(words_counts['f'])

#为所有计数增加一次
morewords=[
'a','b','c','d','e','f'
]
words_counts.update(morewords)
print(words_counts['f'])
```

运行结果:

```
[('a', 5), ('b', 4), ('c', 3)]
2
3
4
5
```

除了上面介绍的用法，Counter对象另一个不为人知的特性，那就是他们可以轻松地同各种数学运算操作结合起来使用。

```
from collections import Counter
words1=[
'a','b','c','d','e','f',
'a','b','c','d','e','f',
'a','b','c',
'a','b',
'a'
]

words2=[
'a','b','c','d','e','f',
'a','b','c',
'a','b',
'a'
]
one=Counter(words1)
two=Counter(words2)
print(one)
print(two)

three=one+two
print(three)

four=one-two
print(four)
```