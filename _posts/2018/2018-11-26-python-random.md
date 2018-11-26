---
layout: single
author_profile: true
title: "python模块介绍-random"
date: 2018-11-26 15:30:53
# toc: true
tags:
  - python
  - random
categories:
  - python模块
---

random是Python产生伪随机数的模块，随机种子默认为系统时钟。下面分析模块中的方法：

### random.randint(start,stop)

这是一个产生整数随机数的函数，参数start代表最小值，参数stop代表最大值，两端的数值都可以取到；

函数算法时间复杂度为：O(1)

核心源码：

    return self.randrange(a,b+1) #调用randrange函数来处理

实例：
```
import random

for i in range(20):
    print(random.randint(0,10),end=' ')
```
结果：

    1 1 7 5 10 1 4 1 0 8 7 7 2 10 6 8 6 0 3 1

### random.randrange(start,stop,step)
也是一个随机整数函数，参数可选

只有一个参数时，默认随机范围是0到该参数，前闭后开；  
两个参数时，表示最小值和最大值，前闭后开  
三个参数时，表示最小值，最大值和步长，前闭后开 

函数算法时间复杂度：O(1)

核心源代码：

    return istart+istep*self._randbelow(n) #调用randbelow函数处理
实例：
```
import random

for i in range(10):
    print(random.randrange(10),end=' ') #产生0到10（不包括10）的随机数

print("")

for i in range(10):
    print(random.randrange(5,10),end=' ') #产生5到10（不包括10）的随机数

print("")

for i in range(10):
    print(random.randrange(5,100,5),end=' ') #产生5到100（不包括100）范围内的5倍整数的随机数
```
结果：
```
1 1 2 4 4 3 4 6 1 4
6 6 5 7 8 9 6 6 6 5
30 50 20 40 75 85 25 65 80 95
```

### random.choice(seq)
一个随机选择函数，seq是一个非空的集合，在集合中随机选择了一个元素输出，元素的类型没有限制。

核心源代码：
```
i=self._randbelow(len(seq)) #由randbelow函数得到随机地下标
return seq[i]
```
函数算法时间负责度：O(1)

实例：
```
import random

list3=["mark","帅",18,[183,138]]
for j in range(10):
    print(random.choice(list3),end=' ')
```
结果：

    mark 帅 [183, 138] 18 mark 18 mark 帅 帅 [183, 138]

### random.random()
这个函数形成从0.0到1.0之间的任意浮点数，左闭右开，没有参数。

实例：
```
import random

for j in range(5):
    print(random.random(),end=' ')
```
运行结果：

    0.357486615834809 0.5928029747238529 0.37053940107869987 0.3802224543848519 0.9741990956161711

### random.send(n=None)
一个可以对随机数生成器进行初始化的函数，n代表随机种子；当n=None时，随机种子为系统时间，当n为其他的数据，如int，str等时，则以提供的数据作为随机种子，此时生成的随机数列固定。

实例：
```
import random

random.seed("mark")
for j in range(20):#无论启动多少次程序，输出的序列不变
    print(random.randint(0,10),end=' ')
```
结果：

    4 1 10 5 6 2 8 5 5 10 7 2 9 6 2 6 0 5 10 10

### random.getstate() 和 random.setstate(state):
getstate()函数用来记录随机数生成器的状态，setstate(state)函数用来将生成器恢复到上次记录的状态。

实例：
```
import random

tuple1=random.getstate()#记录生成器的状态
for i in range(20):
    print(random.randint(0,10),end=' ')
print()
random.setstate(tuple1)#传入参数回复之间的状态
for i in range(20):
    print(random.randint(0,10),end=' ')#两次输出的结果一致
```
结果：
```
5 7 9 9 10 10 2 3 7 1 1 6 1 7 1 1 7 4 2 2
5 7 9 9 10 10 2 3 7 1 1 6 1 7 1 1 7 4 2 2
```
### random.shuffle(seq,random=None):
对传入的集合进行乱序操作。只能针对可变序列，如字符串、列表，对于元组等不可变序列会报错，random用来选择乱序操作的方式，如random=random。

核心源代码：
```
for i in reversed(range(1,len(x))):
    j=randbelow(i+1)
    x[i],x[j]=x[k],x[i]
```
函数算法的时间复杂度：O(n)

实例：
```
import random

lists=['mark','帅哥',18,[183,138]]
print(lists)
random.shuffle(lists,random=None)
print(lists)
```
结果：
```
['mark', '帅哥', 18, [183, 138]]
['帅哥', 18, 'mark', [183, 138]]
```

### random.sample(population,k):
population参数是一个序列，如列表、元组、集合、字符串等；从集合中随机抽取k个元素形成新的序列，不会改变原有的序列。

最坏时间复杂度：O(n*n)

实例：
```
import random

lists=['mark','帅哥',18,[183,138]]
lists2=random.sample(lists,3)
print(lists)
print(lists2)
```
结果：
```
['mark', '帅哥', 18, [183, 138]]
['mark', [183, 138], '帅哥']
```

### random.uniform(a,b)
生成参数a到b之间的浮点数的函数，如果a>b ,则生成b到a之间的浮点数。

核心源码：

    return a+(b-a)*self.random()
时间复杂度：0(1)

实例：
```
import random

for i in range(5):
    print(random.uniform(10,1))
```
结果：
```
2.8826090956524606
1.5211191352548408
3.2397454278562794
4.147879756524251
6.532545391009419
```
