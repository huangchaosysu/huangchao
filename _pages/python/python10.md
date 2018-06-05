---
layout: single
permalink: /python/if/
title: "条件判断"
excerpt: "条件判断"
sidebar:
  nav: "python"
# toc: true
---

计算机之所以能做很多自动化的任务，因为它可以自己做条件判断。

比如，输入用户年龄，根据年龄打印不同的内容，在Python程序中，用if语句实现：
```
age = 20
if age >= 18:
    print('your age is', age)
    print('adult')
```
根据Python的缩进规则，如果if语句判断是True，就把缩进的两行print语句执行了，否则，什么也不做。

也可以给if添加一个else语句，意思是，如果if判断是False，不要执行if的内容，去把else执行了：
```
age = 3
if age >= 18:
    print('your age is', age)
    print('adult')
else:
    print('your age is', age)
    print('teenager')
```
注意不要少写了冒号:。

当然上面的判断是很粗略的，完全可以用elif做更细致的判断：
```
age = 3
if age >= 18:
    print('adult')
elif age >= 6:
    print('teenager')
else:
    print('kid')
```
elif是else if的缩写，完全可以有多个elif，所以if语句的完整形式就是：
```
if <条件判断1>:
    <执行1>
elif <条件判断2>:
    <执行2>
elif <条件判断3>:
    <执行3>
else:
    <执行4>
```
if语句执行有个特点，它是从上往下判断，如果在某个判断上是True，把该判断对应的语句执行后，就忽略掉剩下的elif和else，所以，请测试并解释为什么下面的程序打印的是teenager：
```
age = 20
if age >= 6:
    print('teenager')
elif age >= 18:
    print('adult')
else:
    print('kid')
```
if判断条件还可以简写，比如写：
```
if x:
    print('True')
```
只要x是非零数值、非空字符串、非空list等，就判断为True，否则为False。

再议 input
最后看一个有问题的条件判断。很多同学会用input()读取用户的输入，这样可以自己输入，程序运行得更有意思：
```
birth = input('birth: ')
if birth < 2000:
    print('00前')
else:
    print('00后')
```
输入1982，结果报错：
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
TypeError: unorderable types: str() > int()
```
这是因为input()返回的数据类型是str，str不能直接和整数比较，必须先把str转换成整数。Python提供了int()函数来完成这件事情：
```
s = input('birth: ')
birth = int(s)
if birth < 2000:
    print('00前')
else:
    print('00后')
```
再次运行，就可以得到正确地结果。但是，如果输入abc呢？又会得到一个错误信息：
```
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
ValueError: invalid literal for int() with base 10: 'abc'
```
原来int()函数发现一个字符串并不是合法的数字时就会报错，程序就退出了。

如何检查并捕获程序运行期的错误呢？后面的错误和调试会讲到。

练习
小明身高1.75，体重80.5kg。请根据BMI公式（体重除以身高的平方）帮小明计算他的BMI指数，并根据BMI指数：

低于18.5：过轻
18.5-25：正常
25-28：过重
28-32：肥胖
高于32：严重肥胖

批注: 

* 注意if x: 这种写法，x如果是个数值，那么只要x非0， 就是True。例如x=-1时也是True， 这里跟其他语言有点区别

* if语句还有个短路的特性，编写代码的时候可以应用以下来提高执行效率。例:
```
a = xxx
b = xxx
if a > 1 and b < 1:
    do somethind
```
短路特性是指， 如果a>1不成立，那么if语句就直接跳过了，后面的b<1就不会再判断了。同时这个例子也掩饰了一个if语句有多个判断条件的写法。上例是a>1和b<1同时成立才执行。
```
if a > 1 or b < 1:
    do somethind
```
这段代码是a>1和b<1 有一个成立就执行