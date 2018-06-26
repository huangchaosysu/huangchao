---
layout: single
permalink: /django/firstapp4/
title: "第一个Django应用-4"
sidebar:
  nav: "django"
toc: true
---

### Views

<p>本章节将继续前面的学习。首先我们来讲解一下View</p>
<p>在Django中，view就是一个网页的抽象，它实际上是一个可运行的python函数，在这个函数中产生我们想让用户看到的数据。当然，要实际产生一个网页出来还需要搭配使用template(模板)。在polls这个app中，我们会编写以下几个view:</p>
<ul>
    <li>Question的index页面: 该页面展示最新的若干Question</li>
    <li>Question的详情(detail)页: 展示某个Question的详细信息</li>
    <li>Question的结果页(result): 展示某个Question的结果</li>
    <li>投票功能: 处理投票计数</li>
</ul>
<p>在Django中，每个view都对应一个url，Django通过URLconf来完成从url到view的映射。本章会对urlconf有一个简单的介绍，详情请参见Django进阶</p>

### 编写View
<em>polls/views.py</em>
<pre>
def detail(request, question_id):
    return HttpResponse("You're looking at question %s." % question_id)

def results(request, question_id):
    response = "You're looking at the results of question %s."
    return HttpResponse(response % question_id)

def vote(request, question_id):
    return HttpResponse("You're voting on question %s." % question_id)
</pre>

<p>接下来给每个view绑定一个url</p>
<em>mysite/urls.py</em>
<pre>
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^polls/', include("polls.urls"))
]
</pre>

<em>polls/urls.py</em>
<pre>
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
</pre>
<p>打开浏览器，访问<code>"/polls/34/"</code>，调用detail，会显示我们从url传入的任意id以及其提示信息(You're looking at question 34), 访问<code>"/polls/34/results/"</code>， 调用results, 会显示我们传入的任意id以及提示信息(You're looking at the results of question 34), 同理调用<code>"/polls/34/vote/"</code>的结果就很明显了</p>
<p>每当Django收到一个请求(假设请求到url为'/polls/34/'), Django会载入mysite.settings中的ROOT_URLCONF(本例中为mysite.urls), 在mysite.urls中寻找urlpatterns这个变量并遍历这个数组的内容并进行url匹配. 遍历到<code>'^polls/'</code>发现了一个匹配，django会把匹配到的"polls/"部分截断，剩下的"34/"与"polls.urls"中的urlpatterns进行匹配, 直到找到"'^(?P<question_id>[0-9]+)/"这个匹配项, 然后调用detail. 最后实际对view的调用为:</p>
<pre>
detail(request=<HttpRequest object>, question_id='34')
</pre>
<p>question_id这个参数来自于(?P<question_id>[0-9]+). 使用括号表示抓取参数，会把括号内部的正则表达式匹配的内容作为参数传递到view，?P<question_id>表示一个参数名为question_id的关键字参数，如果是非关键字参数，则把?P<question_id>去掉即可。在此强调一下，Django的url不需要在末尾加.html等后缀，也不提倡.</p>

### 完善View

<p>上面的例子中，view只是返回了一串文字，实际应用中需要view做的事情肯定不会这么简单，接下来我们就把现有的view稍微改复杂一点。</p>
<p>编辑polls/views.py</p>

```
from django.http import HttpResponse

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    output = ', '.join([q.question_text for q in latest_question_list])
    return HttpResponse(output)

# 其他view保持不变
```

<p>这样,我们就把index修改成展示最近发布的5个问题了.但是,这里有个问题.现在的这个index页面展示的内容都是我们在index这个view里硬编码的,如果日后我们想修改一下这个页面的内容,那么我们还需要修改index这个view函数,这是不能接受的.好在Django提供了一种解决的方法,那就是使用模板(template)</p>

### 使用模板
<p>首先在polls目录下创建一个templates目录，这是存放模板文件的目录。mysite/settings.py中的TEMPLATES这个配置项指定了django如何搜寻模板文件。默认情况下是从每个已经激活的app(INSTALLED_APP)目录下的templates目录以及mysite目录下的templates目录下搜寻模板文件。</p>

<p>在我们刚才创建的templates目录下再创建一个polls目录，在polls目录下新建一个名为index.html的模板文件。完整的路径是这样的<code>polls/templates/polls/index.html</code>。鉴于我们刚才所说的Django搜寻模板的方式，我们只需要在Django种使用"polls/index.html"就可以使用了</p>

<p>有一点需要注意的事情: 在上面的例子中，我们的index.html是放在polls目录下的，为什么不直接把index.html放到templates目录下而是在templates目录下又建了一个polls目录呢?原因是这样的，假如一个Django项目有好多个app，每个app都有一个index.html文件，那么整个项目就有好多个index.html，Django搜索模板是根据INSTALLED_APP里面所列的app顺序搜索的，假设在第一个app的templates目录下找到了名为index.html的文件，那么Django就停止搜索了，然而，这个index.html可能并不是我们想要的，因此保险起见，最好再加一层以app的名字命名的目录。</p>

![](/assets/images/django/6)

<p>接下来，把index这个view修改为使用index.html这个模板</p>

<pre>
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
</pre>

<p>现在，重启开发服务器，再访问<code>http://localhost:8000/polls/</code>, 现在的index就是调用polls/index.html这个模板了</p>

### 使用快捷方式:render()
<p>在使用Django编写view时，载入模板，组装context，返回HttpResponse对象是非常常用的一个流程，Django提供了一个render()方法来帮助我们完成这个流程，不用每次都写这么多代码</p>
<p>再次修改polls/views.py</p>

<pre>
from django.shortcuts import render

from .models import Question


def index(request):
    latest_question_list = Question.objects.order_by('-pub_date')[:5]
    context = {'latest_question_list': latest_question_list}
    return render(request, 'polls/index.html', context)
</pre>
<p>可以看到，最新的index已经不需要使用loader和HttpResponse了(detail等其他3个还是需要HttpResponse的)，render()方法需要3个参数，第一个是request，第二个是模板，第三个是我们要传递给模板的内容</p>

### 模板系统
<p>编辑polls/views.py, 修改detail这个view</p>

<pre>
from .models import Question
# ...
def detail(request, question_id):
    try:
        question = Question.objects.get(pk=question_id)
    except Question.DoesNotExist:
        pass
    return render(request, 'polls/detail.html', {'question': question})
</pre>

<p>上面的代码把detail也修改成了使用模板的方式。创建polls/templates/polls/detail.html, 填入以下代码</p>

![](/assets/images/django/7)
