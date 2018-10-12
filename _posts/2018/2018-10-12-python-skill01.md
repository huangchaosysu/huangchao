---
layout: single
author_profile: true
title: "python实用技法-01"
date: 2018-10-12 17:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---

### 需求

* 我们想去除列表中出现的重复元素，但仍然保持剩下的元素的顺序不变。

如果只是想要去重，那么通常足够简单的方法就是构建一个集合：

```
a=[1,5,4,36,7,8,2,3,5,7]
#结果为：{1, 2, 3, 4, 5, 36, 7, 8}
print(set(a))
```


### 解决方案

如果序列中的值是可哈希的（hashable），那么这个问题可以通过使用集合和生成器轻松解决。

如果一个对象是可哈希的，那么它的生存期内必须是不可变的，它需要有一个__hash__()方法。整数、浮点数、字符串、元素都是不可变的。关于什么是可哈希的，可以参考python核心编程这本书
```
def dedupe(items):
    seen=set()
    for item in items:
        if item not in seen:
            yield item
            seen.add(item)

a=[1,2,3,1,9,1,5,10]
print(list(dedupe(a)))

# 结果 [1, 2, 3, 9, 5, 10]
```

只有当序列中的元素是可哈希的时候才能这么做。如果想在不可哈希的对象序列中去除重复项，需要上述代码稍作修改：

```
# 这里添加了一个key参数，这个参数用来把非可哈希的元素转变为可哈兮的元素
def dedupe(items,key=None):
    seen=set()
    for item in items:
        value=item if key is None else key(item)
        if value not in seen:
            yield item
            seen.add(value)

a=[
    {'x':1,'y':2},
    {'x':1,'y':3},
    {'x':1,'y':4},
    {'x':1,'y':2},
    {'x':1,'y':3},
    {'x':1,'y':1},

]
print(list(dedupe(a,key=lambda d:(d['x'],d['y']))))

print(list(dedupe(a,key=lambda d:d['y'])))
```

运行结果:

```
[{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 1, 'y': 4}, {'x': 1, 'y': 1}]
[{'x': 1, 'y': 2}, {'x': 1, 'y': 3}, {'x': 1, 'y': 4}, {'x': 1, 'y': 1}]
```

这里的参数key的作用是指定一个函数用来将序列中的元素转换为可哈希的类型，那么做的目的是为了检测重复项。