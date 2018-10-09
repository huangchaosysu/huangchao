---
layout: single
author_profile: true
title: "python字典操作"
date: 2018-10-09 15:30:53
# toc: true
tags:
  - python
categories:
  - python
---

Python 中的字典是Python中一个键值映射的数据结构,下面介绍一下字典的操作

### 创建字典

Python有两种方法可以创建字典,第一种是使用花括号,另一种是使用内建 函数dict

```
>>> info = {}
>>> info = dict()
``` 

### 初始化字典

Python可以在创建字典的时候初始化字典

```
>>> info = {"name" : 'cold'}
>>> info = dict(name = 'cold')
```
很明显第二种方法更加的优雅和减少一些特殊字符的输入,但是有种情况第二种不能胜任
```
>>> key = 'name'
>>> info = { key :'cold'}  # {'name':'cold'}
>>> info = dict(key = 'cold') # {'key': 'cold'}
```
明显第二种方法就会引发一个不容易找到的bug

Python字典还有一种初始化方式,就是使用字典的fromkeys方法可以从列表中获取元素作为键并用None或fromkeys方法的第二个参数初始化
```
>>> info = {}.fromkeys(['name', 'blog'])
>>> info
{'blog': None, 'name': None}
>>> info = dict().fromkeys(['name', 'blog'])
>>> info
{'blog': None, 'name': None}
>>> info = dict().fromkeys(['name', 'blog'], 'linuxzen.com')
>>> info
{'blog': 'linuxzen.com', 'name': 'linuxzen.com'}
```

### 优雅的获取键值

字典可以这样获取到键的值
```
>>> info = {'name':'cold', 'blog':'linuxzen.com'}
>>> info['name']
'cold'
```

但是如果获取不存在的键的值就会触发的一个KeyError异常,字典有一个get方法,可以使用字典get方法更加优雅的获取字典

```
>>> info = dict(name= 'cold', blog='www.linuxzen.com')
>>> info.get('name')
'cold'
>>> info.get('blogname')
None
>>> info.get('blogname', 'linuxzen')
'linuxzen'
```

我们看到使用get方法获取不存在的键值的时候不会触发异常,同时get方法接收两个参数,当不存在该键的时候就会返回第二个参数的值 我们可以看到使用get更加的优雅

### 更新字典

Python 字典可以使用键作为索引来访问/更新/添加值
```
>>> info = dict()
>>> info['name'] = 'cold'
>>> info['blog'] = 'linuxzen.com'
>>> info
{'blog': 'linuxzen.com', 'name': 'cold'}
>>> info
{'blog': 'linuxzen.com', 'name': 'cold night'}
```
同时Python字典的update方法也可以更新和添加字典
```
>>> info = dict(name='cold', blog='linuxzen.com')
>>> info.update({'name':'cold night', 'blogname':'linuxzen'})
>>> info
{'blog': 'linuxzen.com', 'name': 'cold night', 'blogname': 'linuxzen'}
>>> info.update(name='cold', blog='www.linuxzen.com') # 更优雅
>>> info
{'blog': 'www.linuxzen.com', 'name': 'cold', 'blogname': 'linuxzen'}
```

Python字典的update方法可以使用一个字典来更新字典,也可以使用参数传递类似dict函数一样的方式更新一个字典,上面代码中哦功能的第二个更加优雅,但是同样和dict函数类似,键是变量时也只取字面值


### 删除字典的元素

可以调用Python内置关键字del来删除一个键值
```
>>> info = dict(name='cold', blog='linuxzen.com')
>>> info
{'blog': 'linuxzen.com', 'name': 'cold'}
>>> del info['name']
>>> info
{'blog': 'linuxzen.com'}
```
同时也可以使用字典的pop方法来取出一个键值,并删除
```
>>> info = dict(name='cold', blog='linuxzen.com')
>>> info.pop('name')
'cold'
>>> info
{'blog': 'linuxzen.com'}
```

### 遍历

获取所有key
```
>>> info = dict(name='cold', blog='linuxzen.com')
>>> info.keys()
['blog', 'name']
```
获取key,value并循环
```
>>> info = dict(name='cold', blog='linuxzen.com')
>>> for key, value in info.items():
...     print() key, ':',  value)
... 
blog : linuxzen.com
name : cold
```