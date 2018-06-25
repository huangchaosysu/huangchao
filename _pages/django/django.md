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