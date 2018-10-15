---
layout: single
author_profile: true
title: "python实用技法-02"
date: 2018-10-15 14:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---



* 现在有两个字典，我们想找出它们中间可能相同的地方（相同的键、相同的值）

### 解决方案

只需要用过keys()或者item()方法执行常见的集合操作（并集、交集、差集）即可。

```
a={
    'x':1,
    'y':2,
    'z':3
}
b={
    'w':10,
    'x':11,
    'y':2
}

#找出 在两个字典中都存在的键
print(a.keys() & b.keys())

#找出 存在a却不存在b的键
print(a.keys() -b.keys())

#找出两个字典中，键和值都同时相等的数据
print(a.items() & b.items())
```

运行结果

```
{'y', 'x'}
{'z'}
{('y', 2)
```

注意:字典中的values()不支持上面的集合操作，因为字典同一个值可能会对应多个键。

另外也可参考使用set操作