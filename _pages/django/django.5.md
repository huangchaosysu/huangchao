---
layout: single
permalink: /django/firstapp4/
title: "第一个Django应用-4"
sidebar:
  nav: "django"
toc: true
---

### Views

本章节将继续前面的学习。首先我们来讲解一下View  
在Django中，view就是一个网页的抽象，它实际上是一个可运行的python函数，在这个函数中产生我们想让用户看到的数据。当然，要实际产生一个网页出来还需要搭配使用template(模板)。在polls这个app中，我们会编写以下几个view:

* Question的index页面: 该页面展示最新的若干Question
* Question的详情(detail)页: 展示某个Question的详细信息
* Question的结果页(result): 展示某个Question的结果
* 投票功能: 处理投票计数

在Django中，每个view都对应一个url，Django通过URLconf来完成从url到view的映射。本章会对urlconf有一个简单的介绍，详情请参见Django进阶系列的文章

### 编写View

*polls/views.py*

```
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
```

接下来给每个view绑定一个url  

*mysite/urls.py*

```
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^polls/', include("polls.urls"))
]
```

*polls/urls.py*

```
from django.conf.urls import url

from . import views

urlpatterns = [
    # ex: /polls/
    url(r'^$', views.index, name='index'),
    # ex: /polls/5/
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    # ex: /polls/5/results/
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    # ex: /polls/5/vote/
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```

打开浏览器，访问```/polls/34/```，调用detail，会显示我们从url传入的任意id以及其提示信息(You're looking at question 34), 访问```/polls/34/results/```， 调用results, 会显示我们传入的任意id以及提示信息(You're looking at the results of question 34), 同理调用```/polls/34/vote/```的结果就很明显了

每当Django收到一个请求(假设请求到url为'/polls/34/'), Django会载入mysite.settings中的ROOT_URLCONF(本例中为mysite.urls), 在mysite.urls中寻找urlpatterns这个变量并遍历这个数组的内容并进行url匹配. 遍历到```polls/```发现了一个匹配，django会把匹配到的```polls/```部分截断，剩下的"34/"与"polls.urls"中的urlpatterns进行匹配, 直到找到```^(?P<question_id>[0-9]+)/```这个匹配项, 然后调用detail. 最后实际对view的调用为:

```
detail(request=<HttpRequest object>, question_id='34')
```

question_id这个参数来自于(?P<question_id>[0-9]+). 使用括号表示抓取参数，会把括号内部的正则表达式匹配的内容作为参数传递到view，```?P<question_id>```表示一个参数名为question_id的关键字参数，如果是非关键字参数，则把```?P<question_id>```去掉即可。在此强调一下，Django的url不需要在末尾加.html等后缀，也不提倡. 注意，这里是1.x版本中的参数截取方法，2.x的见文章末尾

### 完善View

上面的例子中，view只是返回了一串文字，实际应用中需要view做的事情肯定不会这么简单，接下来我们就把现有的view稍微改复杂一点。
编辑polls/views.py

```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# 其他view保持不变
```

这样,我们就把index修改成展示最近发布的5个问题了.但是,这里有个问题.现在的这个index页面展示的内容都是我们在index这个view里硬编码的,如果日后我们想修改一下这个页面的内容,那么我们还需要修改index这个view函数,这是不能接受的.好在Django提供了一种解决的方法,那就是使用模板(template)

### 使用模板

首先在polls目录下创建一个templates目录，这是存放模板文件的目录。mysite/settings.py中的TEMPLATES这个配置项指定了django如何搜寻模板文件。默认情况下是从每个已经激活的app(INSTALLED_APP)目录下的templates目录以及mysite目录下的templates目录下搜寻模板文件。

在我们刚才创建的templates目录下再创建一个polls目录，在polls目录下新建一个名为index.html的模板文件。完整的路径是这样的```polls/templates/polls/index.html```。鉴于我们刚才所说的Django搜寻模板的方式，我们只需要在Django种使用"polls/index.html"就可以了

有一点需要注意的事情: 在上面的例子中，我们的index.html是放在polls目录下的，为什么不直接把index.html放到templates目录下而是在templates目录下又建了一个polls目录呢?原因是这样的，假如一个Django项目有好多个app，每个app都有一个index.html文件，那么整个项目就有好多个index.html，Django搜索模板是根据INSTALLED_APP里面所列的app顺序搜索的，假设在第一个app的templates目录下找到了名为index.html的文件，那么Django就停止搜索了，然而，这个index.html可能并不是我们想要的，因此保险起见，最好再加一层以app的名字命名的目录。

![](/assets/images/django/6)

接下来，把index这个view修改为使用index.html这个模板

```
from django.http import HttpResponse
from django.template import loader

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    template = loader.get_template('polls/index.html')
    context = {
        'latest_question_list': latest_question_list,
    }
    return HttpResponse(template.render(context, request))

# 其他保持不变
```

现在，重启开发服务器，再访问```http://localhost:8000/polls/```, 现在的index就是调用polls/index.html这个模板了

### 使用快捷方式:render()

在使用Django编写view时，载入模板，组装context，返回HttpResponse对象是非常常用的一个流程，Django提供了一个render()方法来帮助我们完成这个流程，不用每次都写这么多代码  
再次修改polls/views.py

```
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
```

可以看到，最新的index已经不需要使用loader和HttpResponse了(detail等其他3个还是需要HttpResponse的)，render()方法需要3个参数，第一个是request，第二个是模板，第三个是我们要传递给模板的内容

### 模板系统

编辑polls/views.py, 修改detail这个view

```
from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        pass
    return render(request, 'polls/detail.html', {'question': question})
```

上面的代码把detail也修改成了使用模板的方式。创建polls/templates/polls/detail.html, 填入以下代码

![](/assets/images/django/7)

我们来讲解一下detail页面是怎么产生的。首先在detail这个view里，我们依据question_id来查询出一个Question(except分支暂时忽略了question_id不存在的情况)，调用render把得到的question对象传递给模板detail.html。在模板里，通过点号'.'来获取对象的属性，就跟python代码里一样。例如```{{ question.question_text }}```是获取我们传入的question的question_text这个属性的值，for开头这句代码实际调用了question.choice_set.all()这个方法，而 for那条语句是表示循环，后面我们会有专门的章节来讲解django模板的tag

### 消除硬编码的URL

在index.html里，有个URL是这么写的```<li><a href="/polls/{{ question.id }}/">{{ question.question_text }}</a></li>```，这么写程序虽然可以正常工作，但是有个问题就是，日后如果我们修改了index这个view的URL，那么我们还要修改index.html这个模板才行。好在Django提供了 **url** 这个tag来解决这个问题(其实就是反向解析url)

现在我们来修改一下index.html, 把之前的  
![](/assets/images/django/8)
修改为  
![](/assets/images/django/9)

这种写法的工作原理是，Django在模板里发现url这个tag的时候会到polls.urls这个文件所定义的url里面寻找对应名字的url，本例中就是寻找名字为'detail'的url，也就是

```
url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail')
```

日后如果我们需要修改detail这个view对应的url，我们只需要在polls/urls.py这个文件里修改对应的url映射就可以了

### URL的命名空间

本例中，我们只有polls这一个app，但是在实际的项目中，一般都会有多个app。如果每个app中都有一个叫做detail的url该怎么办呢？Django应该选择哪个呢？为了解决这个问题，django提供了url的命名空间支持。

修改polls/urls.py

```
from django.conf.urls import url

from . import views

app_name = 'polls'
urlpatterns = [
    url(r'^$', views.index, name='index'),
    url(r'^(?P<question_id>[0-9]+)/$', views.detail, name='detail'),
    url(r'^(?P<question_id>[0-9]+)/results/$', views.results, name='results'),
    url(r'^(?P<question_id>[0-9]+)/vote/$', views.vote, name='vote'),
]
```
注意到，我们设置了app_name这个属性，它的作用是告诉Django，此文件中的urlconf从属于命名空间polls. 那么怎么使用呢？  
修改index.html, 把

![](/assets/images/django/10)
<p>修改为</p>

![](/assets/images/django/11)

就可以了，命名空间的使用方式为```{% url 'namespace:name' %}```

如果你已经掌握了本章节的内容，那么赶紧进入下一章节的学习吧

注意：django2.0已经出了新的url的编写方法，像这样  

```
from django.urls import path

from . import views

urlpatterns = [
    # ex: /polls/
    path('', views.index, name='index'),
    # ex: /polls/5/
    path('<int:question_id>/', views.detail, name='detail'),
    # ex: /polls/5/results/
    path('<int:question_id>/results/', views.results, name='results'),
    # ex: /polls/5/vote/
    path('<int:question_id>/vote/', views.vote, name='vote'),
]
```