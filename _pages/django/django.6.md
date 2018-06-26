---
layout: single
permalink: /django/firstapp5/
title: "第一个Django应用-5"
sidebar:
  nav: "django"
toc: true
---

<h4>使用Form</h4>
<p>本章节将介绍Django是如何处理表单数据的</p>
<p>修改<code>polls/templates/polls/detail.htm</code>，先给detail页面添加一个表单</p>

![](/assets/images/django/12)

<p>现在来讲解一下上面的这段代码</p>

* 上面的代码给每一个选项前面都添加了一个单选的radio按钮，每个radio关联的值为其所代表的choice的ID，每个video按钮关联的文字描述为choice的文本内容。
* 上面我们把form的action设置为\{% url 'polls:vote' question.id %\}，请求方法为method=post，这里请务必使用post，因为此表单要修改服务器数据，在web开发中，凡是涉及到修改服务器数据的，请使用post。
* forloop.counter记录了for循环已经运行了多少次
* \{% csrf_token %\}是为了预防csrf攻击而设的，后面会有文章单独讲。有兴趣的读者可以先百度一下。

<p>接下来我们修改vote这个view来处理表单数据。编辑polls/views.py</p>

```
from django.shortcuts import get_object_or_404, render
from django.http import HttpResponseRedirect, HttpResponse
from django.urls import reverse

from .models import Choice, Question

def vote(request, question_id):
    question = get_object_or_404(Question, pk=question_id)
    try:
        selected_choice = question.choice_set.get(pk=request.POST['choice'])
    except (KeyError, Choice.DoesNotExist):
        # Redisplay the question voting form.
        return render(request, 'polls/detail.html', {
            'question': question,
            'error_message': "You didn't select a choice.",
        })
    else:
        selected_choice.votes += 1
        selected_choice.save()
        # Always return an HttpResponseRedirect after successfully dealing
        # with POST data. This prevents data from being posted twice if a
        # user hits the Back button.
        return HttpResponseRedirect(reverse('polls:results', args=(question.id,)))
```

<p>代码讲解:</p>

* request.POST是一个类似于python字典的对象，这个对象里面存储了post请求传过来的所有参数(仅限post参数)的值。可以像操作字典一样操作该对象。本例中，<code>request.POST['choice']</code>就是获取表单里面choice这个字段的值，同理，request.GET存储了所有get请求传递过来的参数。
* 如果choice这个key在request.POST中不存在，那么代码会抛出一个KeyError
* HttpResponseRedirect的作用是返回一个redirect，将浏览器重定向到另外一个地址。本例中我们把票数+1以后就返回了一个重定向，重定向的目标地址为result页面
* reverse方法的作用跟我们在模板里使用\{% url %\}基本相同，它返回一个url地址

<p>注:练习本章节的代码时请先使用django admin给question添加几个choice选项，不然在投票页面看到的会是一个空列表，看不到投票选项</p>