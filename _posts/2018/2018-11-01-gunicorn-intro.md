---
layout: single
author_profile: true
title: "gunicorn简介"
date: 2018-10-29 10:30:53
# toc: true
tags:
  - python
  - gunicorn
  - uwsgi
categories:
  - python
---

WSGI协议: Web框架致力于如何生成HTML代码，而Web服务器用于处理和响应HTTP请求。Web框架和Web服务器之间的通信，需要一套双方都遵守的接口协议。WSGI协议就是用来统一这两者的接口的。

WSGI容器——Gunicorn: 常用的WSGI容器有Gunicorn和uWSGI，但Gunicorn直接用命令启动，不需要编写配置文件，相对uWSGI要容易很多，所以这里我也选择用Gunicorn作为容器。

安装
    pip install gunicorn

启动
    $ gunicorn [options] module_name:variable_name

module_name对应python文件，variable_name对应web应用实例。 以最简单的flask应用为例：
```
#main.py
from flask import Flask
app = Flask(name) 
@app.route('/')
def index():    
    return 'hello world' 
if name == 'main':    
    app.run()
```

启动代码：

    gunicorn --worker=3 main:app -b 0.0.0.0:8080

Gunicorn特点

Gunicorn一个开源Python WSGI UNIX的HTTP服务器，Github仓库地址[在这](https://github.com/benoitc/gunicorn)，传说速度快（配置快、运行快）、简单，默认是同步工作，支持Gevent、Eventlet异步，支持Tornado，官方有很详细的文档可以参阅。

特点：

Gunicorn是基于prefork模式的Python wsgi应用服务器，支持 Unix like的系统

采用epoll (Linux下) 非阻塞网络I/O 模型

多种Worker类型可以选择 同步的，基于事件的（gevent tornado等），基于多线程的

高性能，比之uwsgi不相上下

配置使用非常简单

支持 Python 2.x >= 2.6 or Python 3.x >= 3.2

​

需要在你的Django项目的settings.py中的INSTALLED_APPS加入：gunicorn
```
gunicorn --worker-class=gevent isaced.wsgi:application

gunicorn wsgi:application 
#8个worker
gunicorn -w 8 wsgi:application
#指定端口号
gunicorn -w 8 -b 0.0.0.0:8888 wsgi:application
#unix socket
gunicorn -w 8 --bind unix:/xx/mysock.sock wsgi:application
#使用gevent做异步（默认worker是同步的）
gunicorn -w 8 --bind 0.0.0.0:8000 -k 'gevent' wsgi:application 
#选项挺多，看文档或者使用 --help都可以查看
--log-level=DEBUG  日志级别
--timeout=100  超时时间
----config=config.py  使用配置文件

///config.py

# -*- coding: utf-8 -*-

import os

bind = '0.0.0.0:80'
workers = os.cpu_count()
worker_class = 'tornado'  支持tornado时，配置选项
```

--worker-class 指定工作方式，这里我用的gevent

如果提示You need gevent installed to use this worker则表示你还没有安装 gevent。

isaced.wsgi:application 这里是指你的项目名，在Django创建项目的时候会自动生成对应名字文件夹中的wsgi.py，这里就是指的它。

官网简单教程
```
def app(environ, start_response):
    """Simplest possible application object"""
    data = 'Hello, World!\n'
    status = '200 OK'
    response_headers = [
        ('Content-type','text/plain'),
        ('Content-Length', str(len(data)))
    ]
    start_response(status, response_headers)
    return iter([data])

gunicorn --workers=2 test:app
```
nohup

nohup是一个 Linux 命令，搭配 & 来不挂断地运行某条命令达到后台执行的效果，默认会在根目录生成一个 nohup.out文件用来记录所有的 log 信息，也可以重定向到其他位置。这里我们用它来执行gunicorn，来保持 gunicorn 进程不会被挂断。

    nohup gunicorn --worker-class=gevent NSLoger.wsgi:application -b 127.0.0.1:8000&

--worker-class来指定工作方式为gevent，-b指定地址和端口号。

注意：在尾部加上&(and)字符表示后台运行

执行这条命令后可以用ps命令查看进程，就能看到gunicorn了

gunicorn架构

服务模型(Server Model)

Gunicorn是基于 pre-fork 模型的。也就意味着有一个中心管理进程( master process )用来管理 worker 进程集合。Master从不知道任何关于客户端的信息。所有的请求和响应处理都是由 worker 进程来处理的。

Master(管理者)

主程序是一个简单的循环,监听各种信号以及相应的响应进程。master管理着正在运行的worker集合,通过监听各种信号比如TTIN, TTOU, and CHLD. TTIN and TTOU响应的增加和减少worker的数目。CHLD信号表明一个子进程已经结束了,在这种情况下master会自动的重启失败的worker。

gunicorn支持的异步workers
```
greenlet
eventlet
gevent
tornado

sync, eventlet, gevent, tornado, gthread, gaiohttp默认同步sync
```

总结

最后，总结下这几个部分的关系
![](/assets/images/posts/gunicorn)

(nginx收到客户端发来的请求，根据nginx中配置的路由，将其转发给WSGI) nginx：”WSGI，找你的来了！” (WSGI服务器根据WSGI协议解析请求，配置好环境变量，调用start_response方法呼叫flask框架) WSGI服务器：”flask,快来接客，客户资料我都给你准备好了！” (flask根据env环境变量，请求参数和路径找到对应处理函数，生成html) flask：”!@#$%^……WSGI，html文档弄好了，拿去吧。” (WSGI拿到html，再组装根据env变量组装成一个http响应，发送给nginx) WSGI服务器：”nginx,刚才谁找我来着？回他个话，!@#$%^…..” (nginx再将响应发送给客户端)

