---
layout: single
author_profile: true
title: "装饰器"
date: 2018-06-13 16:01:52
tags:
  - Python
categories:
  - Python文章
# toc: true
---

在Python中，装饰器实现是一种非常方便的功能，其可以看做是decorator模式的一种实现（有兴趣的可以看下装饰器模式，后面本人也会写一边关于这个设计模式的）。
python中有一个原则就是一切皆对象，因此：函数作为一种对象，是可以向普通变量一样做各种操作的。在python中，可以把函数被赋值给其他变量，可以作为返回值，也可以在其他函数内定义新的函数（闭包）。那么装饰器是什么呢？装饰器是一个用来包装函数的函数，装饰器在函数申明完成的时候被调用，调用之后返回一个修改之后的函数对象，将其重新赋值原来的标识符，并永久丧失对原始函数对象的访问(申明的函数被换成一个被装饰器装饰过后的函数)
《python核心编程》中，对装饰器有如下的定义：  
**装饰器仅仅是用来装饰函数的一个包装，它返回一个修改后的函数对象，将其重新赋值给原来的标识符， 并永久失去对原始函数对象的访问。**

通过定义可以看出这么几点：
1. 装饰其是用来装饰被修饰的函数的，因此装饰器肯定有方法获取到被修饰的函数的引用，在实现上就是接受一个被修饰的函数的引用作为参数。
2. 既然是对源函数的修饰，那么我们在装饰器中肯定是做了一些源函数中没有做的事情。
3. 返回了一个函数对象，来替换原来的标识符，既然要替换原来的标识符，那么返回的这个函数所接受的参数必然也是同源函数相同啦。
4. 另外一点在这个定义里没有这么明显体现出来的，装饰器是个函数，那么它是可以带参数的。

根据装饰器是否带参数（这里说的参数是除了被包装的函数以外的参数），被修饰的函数是否带参数， 可以有装饰器有参/无参，函数有参/无参，共4种组合的写法：
关键的一点就是记住，上面说的第三点，返回一个与源函数的参数列表相同的函数， 记住这一点，装饰器就差不多会了80%了

下面是一些示例：  
无参数装饰器 - 包装无参数函数，也是最简单的装饰器：
```
def bar(func):  # func 就是被修饰的函数
    def f():
        print("bar")
        func()
    return f

@bar
def foo():
    print("foo")

foo()
```
上面的代码，每次调用foo()都会打印bar， foo
这里，已经不是
```
def foo：
    print("foo")
```
而是变成了与decorator(foo)相当的一个函数

无参数装饰器 – 包装带参数函数
```
def bar(func):  #func 被修饰的函数
    def f(*args, **kwargs):  # *args和**kwargs用来接收源函数的参数
        print("bar")
        func(*args, **kwargs)
    return f  #关键

@bar
def foo(arg):
    print("foo", arg)

foo("fooarg")
```
foo("fooarg") 等价与bar(foo)("fooarg")

其实，此种写法完全可以代替无参数装饰器-无参数函数的情况


带参数装饰器 – 包装无参数函数
```
def bar(arg): # arg用来接受装饰器参数，这里指代处理被包装的函数的参数
    print(arg)
    def b(func): # func接受被包装的函数
        def f():
            print("bar")
            func()
        return f  
    return b

@bar("bararg")
def foo():
    print("foo")

foo()
foo()
```
上面程序的输出是
bararg
bar
foo
bar
foo
与前面的不同在于：比上一层多了一层封装，先传递参数，再传递函数名
这里可以这么理解，我们在定义foo这个函数的时候，是这么用的，@bar('bararg'), 这里可以理解为先调用了一遍bar这个函数，并且返回了一个函数b，而这个b有什么特点呢？它返回了一个跟被修饰的函数有相同参数的函数，是不是很面熟了，对了，这里就返回的之前说过的最简单的装饰器的写法。个人认为，bar不是装饰其，它只是返回了一个装饰器而已，返回的函数b才是真正的装饰器，这种写法只是运用了闭包而已。
这里foo()等价与bar(arg)(foo)()

带参数装饰器– 包装带参数函数
```
def bar(arg):  # arg接受装饰器参数
    print(arg)
    def b(func): # func接受被包装的函数
        def f(*args, **kwargs):
            print("bar")
            func(*args, **kwargs)
        return f
    return b

@bar("bararg")
def foo(arg):
    print("foo", arg)

foo("fooarg")
foo("fooarg")
```
原理同上，就不多解释了
foo('fooarg')等价与bar(arg)(foo)('fooarg')

内置装饰器  
Python内置的装饰器有三个：staticmethod,classmethod, property
```
class A():
    @staticmethod
    def test_static():
        print "static"
    def test_normal(self):
        print "normal"
    @classmethod
    def test_class(cls):
        print "class", cls
``` 
test_static  
staticmethod 类中定义的实例方法变成静态方法  
基本上和一个全局函数差不多(不需要传入self，只有一般的参数)，只不过可以通过类或类的实例对象来调用，不会隐式地传入任何参数。类似于静态语言中的静态方法  

test_normal  
普通对象方法：普通对象方法至少需要一个self参数，代表类对象实例

test_class  
类中定义的实例方法变成类方法, classmethod需要传入类对象，可以通过实例和类对象进行调用。
这是一个和class相关的方法，可以通过类或类实例调用，并将该class对象（不是class的实例对象）隐式地当作第一个参数传入.

就这种方法可能会 比较奇怪一点，不过只要你搞清楚了python里class也是个真实地存在于内存中的对象，而不是静态语言中只存在于编译期间的类型，就好办了。正常的方法就是和一个类的实例对象相关的方法，通过类实例对象进行调用，并将该实例对象隐式地作为第一个参数传入，这个也和其它语言比较像。

staticmethod，classmethod相当于全局方法，一般用在抽象类或父类中。一般与具体的类无关。
类方法需要额外的类变量cls，当有子类继承时，调用类方法传入的类变量cls是子类，而不是父类。
类方法和静态方法都可以通过类对象和类的实例对象访问
定义方式，传入的参数，调用方式都不相同。

property  
对类属性的操作，类似于java中定义getter/setter
```
class B():
    def __init__(self):
        self.__prop = 1
    @property
    def prop(self):
        print "call get"
        return self.__prop
    @prop.setter
    def prop(self, value):
        print "call set"
        self.__prop = value
    @prop.deleter
    def prop(self):
        print "call del"
        del self.__prop
```
装饰器能干嘛：  
关于这个问题，根据上面的讨论可以看出，主要是对函数进行功能扩展，就是说，在已有函数的情况下，我们可以对它添加一些额外的功能，并且不改变原来的函数。  
例如，原来有功能是获取用户信息的函数get_user_info()，如果我们想在获取用户信息的时候顺便记录下是谁在什么时间调用了这个函数，通常情况下我们会在get_user_info这个函数里添加相关的代码，这就带来一个问题，我们需要修改get_user_info，可能对其他不需要这个记录功能的用户造成影响，如果使用装饰器，我们就可以在不改变源函数的情况下来完成这个功能。
