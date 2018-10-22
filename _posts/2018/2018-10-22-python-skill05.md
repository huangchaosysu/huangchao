---
layout: single
author_profile: true
title: "python实用技法-05"
date: 2018-10-22 15:30:53
# toc: true
tags:
  - python
categories:
  - python
  - python实用技法
---

需求: 我们想要实现一个队列，它能够以给定的优先级来对元素排序，且每次pop操作时都会返回优先级最高的那个元素

利用heapq模块实现
代码：
```
import heapq

#利用heapq实现一个简单的优先级队列
class PriorityQueue:
    def __init__(self):
        self._queue=[]
        self._index=0
    def push(self,item,priority):
        heapq.heappush(self._queue,(-priority,self._index,item))
        self._index+=1
    def pop(self):
        return heapq.heappop(self._queue)[-1]

class Item:
    def __init__(self,name):
        self.name=name

    def __repr__(self):
        return 'Item({!r})'.format(self.name)

if __name__ == '__main__':
    q=PriorityQueue()
    q.push(Item('foo'),1)
    q.push(Item('bar'),5)
    q.push(Item('spam'),4)
    q.push(Item('grok'),1)

    print(q.pop())
    print(q.pop())
    #具有相同优先级的两个元素，返回的顺序同它们插入到队列时的顺序相同
    print(q.pop())
    print(q.pop())
```

运行结果：
```
Item('bar')
Item('spam')
Item('foo')
Item('grok')
```
上面的代码核心在于heapq模块的使用。函数heapq.heapqpush()以及heapq.heapqpop()分别实现将元素从列表_queue中插入和移除，且保证列表中第一个元素的优先级最低。heappop()方法总是返回【最小】的元素，因此这就是让队列能弹出正确元素的关键。此外，由于push和pop操作的复杂度都是O(logN),其中N代表堆中元素的数量，因此就算N的值很大，这些操作的效率也非常高。
上面代码中，队列以元组(-priority ,index,item)的形式组成。把priority取负值是为了让队列能够按照元素的优先级从高到底的顺序排列。

变量index的作用是为了将具有相同优先级的元素以适当的顺序排列。通过维护一个不断递增的索引，元素将以它们如队列时的顺序来排列。为了说明index的作用，看下面实例：

```
class Item:
    def __init__(self,name):
        self.name=name

    def __repr__(self):
        return 'Item({!r})'.format(self.name)

if __name__ == '__main__':
    a=(1,Item('foo'))
    b=(5,Item('bar'))
    #下面一句打印True
    print(a<b)


    c=(1,Item('grok'))
    #下面一句会报错：TypeError: '<' not supported between instances of 'Item' and 'Item'
    print(c<a)


    d=(1,0,Item('foo'))
    e=(5,1,Item('bar'))
    f=(1,2,Item('grok'))
    #下面一句打印True
    print(d<e)
    #下面一句打印True
    print(d<f)
```
