---
layout: single
permalink: /django/firstapp6/
title: "第一个Django应用-6"
sidebar:
  nav: "django"
toc: true
---

一个典型的web应用，除了最基础的html文件，通常还需要一些其他的文件，例如css文件或者JavaScript文件来辅助完成一些前端功能。Django把这些文件统一归类为静态文件。本章节将介绍Django是如何管理和使用静态文件的。

在django中，这些静态文件是通过django.contrib.staticfiles这个组件来管理的。下面就通过一个小小的实例来讲解应该怎么用

首先要做的是创建存放静态文件的目录,类似于template，Django默认也会从当前app的目录下的static(polls/static)目录下寻找静态文件并且也是在找到第一个名字匹配的文件就停止寻找.

Django提供了<code>STATICFILES_FINDERS</code>这个配置项来允许我们定制如何搜寻静态文件。其中默认的AppDirectoriesFinder这个组件就是从当前app目录的static目录来搜寻静态文件。

讲完了在哪里存静态文件，下面我们来创建一个静态文件。创建文件```polls/static/polls/style.css```并填入以下代码
```
li a {
    color: green;
}
```
编辑<code>polls/templates/polls/index.html</code>，在head标签内加入以下代码

<pre><span></span><span class="cp">{</span><span>%</span> <span class="k">load</span> <span class="nv">static</span> <span class="cp">%</span><span>}</span>

<span class="p">&lt;</span><span class="nt">link</span> <span class="na">rel</span><span class="o">=</span><span class="s">"stylesheet"</span> <span class="na">type</span><span class="o">=</span><span class="s">"text/css"</span> <span class="na">href</span><span class="o">=</span><span class="s">"</span><span class="cp">{</span><span>%</span> <span class="k">static</span> <span class="s1">'polls/style.css'</span> <span class="cp">%</span><span>}</span><span class="s">"</span> <span class="p">/&gt;</span>
</pre>

static这个标签会自动生成静态文件的url，对于开发调试来说，到这里就已经可以正确的使用这些静态文件了

<p>重启开发服务器，访问<code>http://localhost:8000/polls/</code>, 如果看到所有的question的链接都变成了绿色，那么证明我们的程序是正常工作的</p>

<p>下一个例子我们将演示添加背景图片</p>
<p>新建目录<code>polls/static/polls/images</code>并把background.gif放到该目录下(polls/static/polls/images/background.gif)，编辑<code>polls/static/polls/style.css</code>，加入以下代码</p>
<pre>
body {
    background: white url("images/background.gif") no-repeat right bottom;
}
</pre>
<p>重新载入页面<code>http://localhost:8000/polls/</code>, 如果程序正常工作，我们将会看到背景图片</p>
<p>本节介绍了Django如何管理静态文件，更多详情参见后续django详解部分</p>
