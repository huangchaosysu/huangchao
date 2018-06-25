---
layout: single
permalink: /django/firstapp1/
title: "第一个Django应用-1"
sidebar:
  nav: "django"
toc: true
---

<p>本节开始，将会以示例的形式讲解如何使用Django构建web应用。在接下来的章节中，将介绍一个简单的投票应用。这个应用包含2部分:</p>
<ul>
    <li>一个给人们查看票数并投票的公共的web站点</li>
    <li>一个管理员可见的用于管理投票的管理站点</li>
</ul>
<p>在学习后面的教程前，请再次确保您已经安装了Django。使用下面的命令来检查<code>python3 -m django --version</code>。如果Django正确安装，那么会看到屏幕显示Django的版本信息。本教程使用Django版本为1.10，python版本为3.4。</p>

### 创建项目-Creating a Project
<p>Django自带了一个命令行工具django-admin来管理项目的创建等等事情。使用该命令行工具可以省去不少重复的工作。下面就调用该工具来创建一个项目</p>
<pre>
    $ django-admin startproject mysite
</pre>
<p>上面这条命令会帮助我们建立项目的目录结构，初始化数据库配置等的一系列配置。命令执行完后在当前目录下应该看到一个<code>mysite</code>目录。目录的结构如下</p>
<pre>
mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py
</pre>
<p>在使用该命令创建项目时需要注意的是，避免使用python或者django内置的名字(例如django，test等)来作为项目的名称，这样会引发import错误或者其他奇怪的问题。接下来我们分别来讲解这些文件的作用</p>
<ul>
    <li>最外层的mysite目录为项目的根目录，目录名对于Django没有任何特殊含义，我们可以随意重命名</li>
    <li>manage.py，我们可以在命令行下通过这个文件来操作该Django项目。后面会有专门的章节来讲解。</li>
    <li>里面一层的mysite目录是这个项目实际会使用到的python包，我们通常都会import里面的一些模块</li>
    <li>__init__.py：告诉python这个目录应该被当做一个python包</li>
    <li>settings.py: 这个文件包含了此项目相关的Django配置项</li>
    <li>urls.py：此文件包含了该Django项目的url映射</li>
    <li>wsgi.py: wsgi协议兼容的web server(例如Nginx)与Django项目交互的入口</li>
</ul>

### 开发服务器
<p>django自带了一个开发服务器以允许用户方便的预览和调试web应用。它的启动方法为</p>
<pre>$ python3 manage.py runserver</pre>
<p>运行上面的命令将会看到类似的输出:</p>
<pre>
Performing system checks...

System check identified no issues (0 silenced).

You have unapplied migrations; your app may not work properly until they are applied.
Run 'python manage.py migrate' to apply them.

March 11, 2017 - 15:50:53
Django version 1.10, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
</pre>
<p>默认情况下，开发服务器启动后的监听地址为本机的8000端口，并且只能本机访问。如果需要做出修改可以使用如下命令：</p>
<pre>$ python3 manage.py runserver 8080  ＃ 只修改监听端口</pre>
<pre>$ python3 manage.py runserver 0.0.0.0:8080  ＃ 修改监听地址和端口</pre>

### 创建APP(Polls)
<p>经过前面的一些工作，我们已经有了一个项目环境，但是只有项目是没办法做任何事情的。我们还需要app，在django里，所有的工作逻辑都是由一个一个的app来完成的。编写Django的app需要遵守一定的规范。与创建项目一样，django也自带了工具来管理app，免去我们处理这些规范的麻烦。创建app的指令如下：</p>
<pre>$ python3 manage.py startapp polls</pre>
<p>运行完这个命令，django会自动的在当前目录下给我们创建一个polls目录，这个目录里面存放着polls这个app的内容。polls目录的结构是这样的：</p>
<pre>
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py
</pre>

### 编写view
<p>前面的章节我们讲过，Django里面的http请求的处理逻辑是通过view来完成的，也就是说，view是django处理外部http请求的入口。现在编辑polls/views.py</p>
<em>polls/views.py</em> 
<pre>
from django.http import HttpResponse

def index(request):
    return HttpResponse("Hello, world. You're at the polls index.")
</pre>
<p>上面这段代码演示了Django中最简单的View。要让这个view能够工作，我们还需要把它映射到一个url。在polls目录下新建一个urls.py，现在的目录结构看起来像这样</p>
<pre>
polls/
    __init__.py
    admin.py
    apps.py
    migrations/
        __init__.py
    models.py
    tests.py
    urls.py
    views.py
</pre>
<p>编辑polls/urls.py</p>
<em>polls/urls.py</em>
<pre>
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^$', views.index, name='index'),
]
</pre>
<p>下一步就是在root URLConf(mysite.urls)中添加一个对这个polls.urls的引用(这么做是为了代码结构考虑，如果不考虑代码结构，我们可以把polls.urls的内容直接写到root URLconf中)。这个引用是通过django.conf.urls.include来完成的。代码如下:</p>
<pre>
from django.conf.urls import include, url
from django.contrib import admin

urlpatterns = [
    url(r'^polls/', include('polls.urls')),
    url(r'^admin/', admin.site.urls),
]
</pre>
<p>留意一下<code>include()</code>的使用方式，它前面的这个正则表达式是以／结尾的，而不是$。include的作用是，当匹配到前面的正则表达式时，include把url中与正则表达式匹配的部分砍掉，只保留后面的部分，然后用剩下的部分再去跟所指向的这个urlconf进行匹配。例如：<code>example.com/polls/abc/</code>的url部分为<code>polls/abc/</code>，它会匹配到<code>url(r'^polls/', include('polls.urls'))</code>这条规则，这时django会把<code>polls/abc/</code>中已经匹配到的部分<code>polls/</code>去掉，剩下的<code>abc/</code>再用polls.urls里面的配置进行匹配</p>
<p>现在启动开发服务器<code>python3 manage.py runserver</code></p>
<p>打开浏览器，输入地址http://localhost:8000/polls/， 就会看到这样的输出“Hello, world. You’re at the polls index.”</p>

### 关于url()
<p>url()函数总共有4个参数，其中2个是必须的，它们是regex和view， 另外两个是可选的，分别是kwargs和name</p>
<p>regex: 这个参数是一个正则表达式，Django会按照顺序，从前往后对每个url()里面的正则表达式进行匹配，一旦找到第一个匹配，那么Django就停止匹配，调用匹配到的url()里面的view参数所指向的view方法。需要注意的是，这个匹配过程会忽略域名部分，get参数和post参数等。例如<code>https://www.example.com/myapp/?page=3</code>，参与匹配的url为<code>myapp/</code>,page=3部分和www.example.com部分被忽略了。有关正则表达式的内容可以参考python的re模块或者其他资料</p>
<p>view: 这个参数相当于往django注册了这条url对应的处理函数，而这个函数我们称之为view。每当django匹配到一条url，那么django就会自动调用这个url对应的view，并且会传入一个HttpRequest对象作为第一个参数。url匹配中捕获的参数会依据捕获的方式分别作为位置参数(positional argument)和关键字参数(keyword argument)传入</p>
<p>kwargs: 通过这个参数可以给view传递额外的关键字参数, 在讲解polls这个应用的过程中不会用到，将在更后面的章节介绍</p>
<p>name: 这个参数可以用来给这个url起一个名字，会在url反向解析的时候用到，后面才会介绍</p>
<p>这一节讲解了如何编写一个最基本的django应用，it's so easy！！！</p>