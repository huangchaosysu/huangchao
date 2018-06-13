---
layout: post
title: "__new__()"
excerpt: "Python特殊方法之__new__()"
date: 2018-06-15 16:01:53
tags:
  - Python
categories:
  - Python文章
---

\_\_new\_\_() 是在新式类中新出现的方法，它作用在构造方法建造实例之前，用于空值如何创建类的实例。可以这么理解，在 Python 中存在于类里面的构造方法 \_\_init\_\_() 负责将类的实例化(其实叫初始化更准确)，而在 \_\_init\_\_() 启动之前，\_\_new\_\_() 决定如何创建一个未经初始化的实例以及是否要使用该 \_\_init\_\_() 方法对这个实例进行初始化，因为\_\_new\_\_() 可以调用其他类的构造方法或者直接返回别的对象来作为本类的实例。

\_\_new__() 方法的特性：
* \_\_new__() 方法是在类准备将自身实例化时调用。
* \_\_new__() 方法始终都是类的静态方法，即使没有被加上静态方法装饰器。

类的实例化和它的构造方法通常都是这个样子：
```
class MyClass(object):
    def __init__(self, *args, **kwargs):
        ...

# 实例化
myclass = MyClass(*args, **kwargs)
```

正如以上所示，一个类可以有多个位置参数和多个命名参数，而在实例化开始之后，在调用 \_\_init__() 方法之前，Python 首先调用 \_\_new__() 方法：
```
def __new__(cls, *args, **kwargs):
    ...
　　
```
第一个参数cls是当前正在实例化的类。

如果要得到当前类的实例，应当在当前类中的 \_\_new__() 方法语句中调用当前类的父类的 \_\_new__() 方法。

例如，如果当前类是直接继承自 object，那当前类的 __new__() 方法返回的对象应该为：
```
def __new__(cls, *args, **kwargs):
    ...
    return object.__new__(cls)
```
注意：  
事实上如果（新式）类中没有重写__new__()方法，即在定义新式类时没有重新定义__new__()时，Python默认是调用该类的直接父类的__new__()方法来构造该类的实例，如果该类的父类也没有重写__new__()，那么将一直按此规矩追溯至object的__new__()方法，因为object是所有新式类的基类。

而如果新式类中重写了__new__()方法，那么你可以自由选择任意一个的其他的新式类（必定要是新式类，只有新式类必定都有__new__()，因为所有新式类都是object的后代，而经典类则没有__new__()方法）的__new__()方法来制造实例，包括这个新式类的所有前代类和后代类，只要它们不会造成递归死循环。具体看以下代码解释：
```
class Foo(object):
    def __init__(self, *args, **kwargs):
        ...
    def __new__(cls, *args, **kwargs):
        return object.__new__(cls, *args, **kwargs)    

# 以上return等同于 
# return object.__new__(Foo, *args, **kwargs)
# return Stranger.__new__(cls, *args, **kwargs)
# return Child.__new__(cls, *args, **kwargs)

class Child(Foo):
    def __new__(cls, *args, **kwargs):
        return object.__new__(cls, *args, **kwargs)
# 如果Child中没有定义__new__()方法，那么会自动调用其父类的__new__()方法来制造实例，即 Foo.__new__(cls, *args, **kwargs)。
# 在任何新式类的__new__()方法，不能调用自身的__new__()来制造实例，因为这会造成死循环。因此必须避免类似以下的写法：
# 在Foo中避免：return Foo.__new__(cls, *args, **kwargs)或return cls.__new__(cls, *args, **kwargs)。Child同理。
# 使用object或者没有血缘关系的新式类的__new__()是安全的，但是如果是在有继承关系的两个类之间，应避免互调造成死循环，例如:(Foo)return Child.__new__(cls), (Child)return Foo.__new__(cls)。
class Stranger(object):
    pass
# 在制造Stranger实例时，会自动调用 object.__new__(cls)
```
 
 通常来说，新式类开始实例化时，__new__()方法会返回cls（cls指代当前类）的实例，然后该类的__init__()方法作为构造方法会接收这个实例（即self）作为自己的第一个参数，然后依次传入__new__()方法中接收的位置参数和命名参数。
 

注意：  
如果__new__()没有返回cls（即当前类）的实例，那么当前类的__init__()方法是不会被调用的。如果__new__()返回其他类（新式类或经典类均可）的实例，那么只会调用被返回的那个类的构造方法。
```
class Foo(object):
    def __init__(self, *args, **kwargs):
        ...
    def __new__(cls, *args, **kwargs):
        return object.__new__(Stranger, *args, **kwargs)  

class Stranger(object):
    ...

foo = Foo()
print type(foo)    

# 打印的结果显示foo其实是Stranger类的实例。

# 因此可以这么描述__new__()和__ini__()的区别，在新式类中__new__()才是真正的实例化方法，为类提供外壳制造出实例框架，然后调用该框架内的构造方法__init__()使其丰满。
# 如果以建房子做比喻，__new__()方法负责开发地皮，打下地基，并将原料存放在工地。而__init__()方法负责从工地取材料建造出地皮开发招标书中规定的大楼，__init__()负责大楼的细节设计，建造，装修使其可交付给客户。
```
