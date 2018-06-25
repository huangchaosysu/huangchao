---
layout: single
permalink: /django/firstapp3/
title: "第一个Django应用-3"
sidebar:
  nav: "django"
toc: true
---

### Django Admin简介
<p>Django Admin是Django自带的一个基于web的管理工具，通过这个工具，可以实现对Model对应的数据库表的增删改查等工作</p>

### 创建管理员账户
<p>要使用Django admin，首先需要一个管理员的帐号，因为只有管理员才能使用Django admin。运行命令:</p>
<pre>$ python3 manage.py createsuperuser</pre>
<p>看到提示后填写用户名并回车</p>
<pre>Username: admin</pre>
<p>接下来是填写邮件地址:</p>
<pre>Email address: admin@example.com</pre>
<p>最后一步是填写密码:</p>
<pre>
Password: **********
Password (again): *********
Superuser created successfully.
</pre>

### 启动开发服务器
<p>Django admin默认状态下已经处于激活状态(INSTALLED_APP里面包含了django.contrib.admin), 下面我们就来试用一下。首先启动开发服务器</p>
<pre>$ python3 manage.py runserver</pre>
<p>打开浏览器，访问<code>http://127.0.0.1:8000/admin/</code>， 可以看到如下页面</p>

![](/assets/images/django/0)
<p>使用我们刚才创建的管理员帐号登录后可以看到Django admin的主页</p>

![](/assets/images/django/1)
<p>你可能会奇怪，在主页上并没有看到polls相关的任何东西，不是说Django admin可以用来管理数据吗？别急，看不到是因为我们没有把polls注册到Django admin中。那么怎么注册呢？很简单。编辑<code>polls/admin.py</code></p>
<pre>
from django.contrib import admin

from .models import Question

admin.site.register(Question)
</pre>
<p>重新启动一下开发服务器，再访问刚刚的页面, 就可以看到Question了。</p>

![](/assets/images/django/2)
<p>点击Questions，进入编辑列表页面。这个页面会显示数据库里的所有question，我们可以任意选择一个进行编辑</p>

![](/assets/images/django/3)
<p>点击我们前面创建的<code>What's up?</code>进入编辑页面:</p>

![](/assets/images/django/4)
<p>这里有几点需要说明一下:</p>
<ul>
    <li>编辑时用到的表单是根据我们定义的Question这个Model自动生成的</li>
    <li>不同的Field类型会自动的适配合适的html组件</li>
    <li>DateTimefield会生成快捷键，例如，Date的编辑框后面有一个today的快捷键以及一个日历按钮(点击会弹出日历)</li>
</ul>
<p>尝试修改某些字段掉值然后点击<code>Save and continue editing</code>。然后选择右上方的History就进入下图所示的修改历史页面。</p>

![](/assets/images/django/5)
<p>好了，到这里Django admin就介绍完了，如果你已经掌握了所有内容就赶紧进入下一章节的学习吧。</p>

