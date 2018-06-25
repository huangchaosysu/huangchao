---
layout: single
permalink: /django/overview/
title: "Django概览"
sidebar:
  nav: "django"
toc: true
---

Django诞生于新闻工作室这种快节奏的环境中，它的设计初衷就是让开发者能够更加快速更加简单地开发web应用。下面的内容将带你粗略的了解如何使用Django来开发一个基于数据库的Web app。目前django已经出到的2.x版本，本教程的少部分代码还基于1.x, 所以在输出结果上可能有少许不同。本教程是博主根据官方教程翻译得来，在原教程的基础上有所简化。如有疑问，以官方教程为准。  
该篇文档的目标是带你了解Django运作的原理

### 设计Model
虽然使用Django的时候并不一定要使用数据库，但是Django自带了一个[ORM](http://baike.baidu.com/link?url=RSaX7MasszQDsum0g-adzY1dmLJ_jNfm2s8VVJDjspfibg6HOXDktD6v70y3ByqY72AjfmcUPSJNkwjBjh2eiq) 模块来帮助你使用纯python代码来定义数据库的结构并访问数据库。Django的这一套数据建模语法提供了多种描述数据库中的表结构（数据模型）的方式。下面是一个示例:

*mysite/news/models.py*
```
from django.db import models

class Reporter(models.Model):
    full_name = models.CharField(max_length=70)

    def __str__(self):              # __unicode__ on Python 2, 等效于Python2中的__unicode__方法
        return self.full_name

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)

    def __str__(self):              # __unicode__ on Python 2, , 等效于Python2中的__unicode__方法
        return self.headline
```

### 生成数据库表
有了Model可以通过运行Django命令行工具来自动生成数据库的表结构。在命令行模式下运行如下命令将会自动的根据上面所定义的Model在数据库中生成对应的表。migrate命令会搜索所有可用的Model，根据这些Model的定义在数据库中创建对应的表，如果表已经存在那么会更新表

    $ python3 manage.py migrate

### 试用API
完成以上步骤后，你就可以使用Django提供的丰富的API来访问数据库了。以下代码展示了Django API的使用方法。因为涉及到app等概念，待后面我们讲到这些概念的时候再来运行这些代码会更好一点，这里先做个演示

```
# Import the models we created from our "news" app
>>> from news.models import Reporter, Article

＃数据库中暂时没有Reporter的记录
>>> Reporter.objects.all()
<QuerySet []>

# 创建一个Reporter
>>> r = Reporter(full_name='John Smith')

# 将Reporter保存到数据库中
>>> r.save()

# Django会自动生成一个id
>>> r.id
1

# 再次查询，数据库中现在有我们刚才保存到那个Reporter的记录
>>> Reporter.objects.all()
<QuerySet [<Reporter: John Smith>]>

# 数据库中的字段在Django中是用python对象的属性来表示的
>>> r.full_name
'John Smith'

# Django提供了丰富的数据库查询API
>>> Reporter.objects.get(id=1)
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__startswith='John')
<Reporter: John Smith>
>>> Reporter.objects.get(full_name__contains='mith')
<Reporter: John Smith>
>>> Reporter.objects.get(id=2)
Traceback (most recent call last):
    ...
DoesNotExist: Reporter matching query does not exist.
# 创建一个article并存入数据库
>>> from datetime import date
>>> a = Article(pub_date=date.today(), headline='Django is cool',
...     content='Yeah.', reporter=r)
>>> a.save()

>>> Article.objects.all()
<QuerySet [<Article: Django is cool>]>

# Article对象可以通过API来获取到相关联的Reporter对象
>>> r = a.reporter
>>> r.full_name
'John Smith'

# 反过来，Reporter对象也可以通过api来获取有关联的article对象
>>> r.article_set.all()
<QuerySet [<Article: Django is cool>]>

# API可以追踪任意深度的外键
>>> Article.objects.filter(reporter__full_name__startswith='John')
<QuerySet [<Article: Django is cool>]>
# Change an object by altering its attributes and calling save().
>>> r.full_name = 'Billy Goat'
>>> r.save()

# 删除一条数据库记录
>>> r.delete()
```

### admin接口
Django提供了一个可以用于生产环境的管理界面，它是一个基于web的应用，用户登录后，可以对数据库的内容进行增删改查等操作,这样就免去了在程序中编写专门的接口来更新数据库的麻烦

*mysite/news/models.py*
```
from django.db import models

class Article(models.Model):
    pub_date = models.DateField()
    headline = models.CharField(max_length=200)
    content = models.TextField()
    reporter = models.ForeignKey(Reporter, on_delete=models.CASCADE)
```
*mysite/news/admin.py*
```
from django.contrib import admin

from . import models

admin.site.register(models.Article)
```

### 设计url
简洁、漂亮的URL是高质量web应用的重要组成部分。Django提倡漂亮的url设计方式，反对将.html, .php等令人讨厌的东西放在url的末尾。想要给app定义url，我们需要创建一个叫做```URLConf```的Python模块.这个模块存储一个URL到callback的映射。这个模块的另外一个作用就是把URL从Python代码中解藕出来。下面是Reporter/Article的一个urlconf示例

```
from django.urls import path

from . import views

urlpatterns = [
    path('articles/<int:year>/', views.year_archive),
    path('articles/<int:year>/<int:month>/', views.month_archive),
    path('articles/<int:year>/<int:month>/<int:pk>/', views.article_detail),
]
```

上面的代码把使用简单的正则表达式表示的url映射到一个python函数（Django里叫做View，其实就是普通的python函数）并使用括号来从url里抓去参数的值。一旦匹配，Django就会调用指定的View。每个view在调用时都会传入一个request对象，这个对象保存了该次http请求相关的信息，前面提到的url抓取的值也会被当做参数传入view。例如“/articles/2005/05/39323/”这个url会调用```news.views.article_detail(request, '2005', '05', '39323')```

### 编写View
在Django中，每个View都必须做2件事中的一个，返回一个HttpResponse对象或者Http404，在满足这一条件的情况下，我们可以做其他任意想做的事.通常情况下一个View里面的处理流程为，获取参数，根据参数来查询数据并计算得到结果，把结果渲染成html并返回

```
mysite/news/views.py
from django.shortcuts import render

from .models import Article

def year_archive(request, year):
    a_list = Article.objects.filter(pub_date__year=year)
    context = {'year': year, 'article_list': a_list}
    return render(request, 'news/year_archive.html', context)
```

### 使用模板
上面的代码调用了```new/year_archive.html```这个模板。Django会根据settings文件里面的DIRS这个配置所指定的目录来搜索模板。下面是一个模板的示例

```
mysite/news/templates/news/year_archive.html

{% extends "base.html" %}

{% block title %}Articles for {{ year }}{% endblock %}

{% block content %}
<span><</span>h1<span>></span>Articles for {{ year }}<span><</span>/h1<span>></span>

{% for article in article_list %}
    <p>{{ article.headline }}</p>
    <p>By {{ article.reporter.full_name }}</p>
    <p>Published {{ article.pub_date|date:"F j, Y" }}</p>
{% endfor %}
{% endblock %}
```

变量由双大括号包围，```{{ article.headline }}```表示将article对象的headline的值替换在这里。这里，点号既可以表示对象属性查询也可以表示字典查询，索引查询甚至函数调用。上面的示例中，```{{ article.pub_date|date:"F j, Y" }}```，这段代码使用了很像unix（或者linux）系统中的管道的语法（'｜'这个字符），这是Django中的模板过滤器（template filter），过滤器可以对变量的值做进一步的处理。该例中，date这个过滤器把python的时间对象格式化为一定的格式。在Django的模板语法中，过滤器可以嵌套多层，也允许用户自己编写过滤器

关于Django的模板，最核心的是Django支持模板的继承。就像面向对象编程中的类继承一样，在Django中可以使用```{% extends "base.html" %}```来实现模板的继承。其逻辑是，先载入base.html这个模板，然后用当前模板的内容来填充base.html中定义的众多block。下面是一个示例：

```
{% load static %}
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
</head>
<body>
    <img src="{% static "images/sitelogo.png" %}" alt="Logo" />
    {% block content %}{% endblock %}
</body>
</html>
```

base.html这个模板定义了整个页面的框架，并且准备好了一个又一个的坑来给它的子模板（继承base.html的模板）来填，这些坑就是你看到的这些```block```。这样的好处就是代码复用，我们不用给相似的页面写重复的代码

<h4>这只是皮毛</h4>
<p>以上只是对Django的一个粗略的概述，Django还提供了其他一些有用的特性，比如缓存，消息订阅等。下一节开始，将从简到难讲解Django到方方面面</p>