---
layout: single
permalink: /python/helloworld/
title: "Hello world"
excerpt: "Hello world"
sidebar:
  nav: "python"
toc: true
---

### 命令行模式
在Windows开始菜单选择“命令提示符”，就进入到命令行模式，它的提示符类似C:\>：  
linux或者mac打开terminal，进入到命令行模式, 提示符类似vegetta@vegetta-To-be-filled-by-O-E-M:~$
![](/assets/images/python/5)

### python交互模式
在命令行模式下，输入python3, 就进入到python交互模式下  
![](/assets/images/python/4)

在写代码之前，请千万不要用“复制”-“粘贴”把代码从页面粘贴到你自己的电脑上。写程序也讲究一个感觉，你需要一个字母一个字母地把代码自己敲进去，在敲代码的过程中，初学者经常会敲错代码：拼写不对，大小写不对，混用中英文标点，混用空格和Tab键，所以，你需要仔细地检查、对照，才能以最快的速度掌握如何写程序。

在交互模式的提示符>>>下，直接输入代码，按回车，就可以立刻得到代码执行结果。现在，试试输入100+200，看看计算结果是不是300：

    >>> 100+200
300
很简单吧，任何有效的数学计算都可以算出来。

如果要让Python打印出指定的文字，可以用print()函数，然后把希望打印的文字用单引号或者双引号括起来，但不能混用单引号和双引号：
```
>>> print('hello, world')
hello, world
```
这种用单引号或者双引号括起来的文本在程序中叫字符串，今后我们还会经常遇到。

最后，用exit()退出Python，我们的第一个Python程序完成！唯一的缺憾是没有保存下来，下次运行时还要再输入一遍代码.要解决这个问题，那么我们需要把代码保存为一个<code>.py</code>后缀的文件。
例如：我们新建一个文件hello.py, 内容为:

    print('hello wrold')

在命令行模式下输入命令: python3 hello.py, 屏幕上就会输出hello world