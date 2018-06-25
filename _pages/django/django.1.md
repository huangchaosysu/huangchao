---
layout: single
permalink: /django/install/
title: "安装Django"
sidebar:
  nav: "django"
# toc: true
---

<h4>安装django</h4>
<p>Django是一个开源的python web框架，它并没有包含在python的默认安装环境中。要使用Django，必须先安装。因为Django是依赖python的，因此必须先安装python。关于python的安装可参考<a href="/article_2_2/">python教程</a>，这里就不重复</p>
<h4>设置数据库</h4>
<p>Django支持多种数据库，包括mysql，postgresql，oracle， sqlite等。本教程中使用mysql进行讲解。安装mysql的过程请参考您对应的操作系统的具体的<a href="https://dev.mysql.com/doc/refman/5.7/en/installing.html">安装过程</a>。这里就不再多说。</p>
<p>要用Django操作mysql，则需要给python安装mysql的驱动，我们推荐使用mysqlclient，其安装方式为<code>pip3 install mysqlclient</code>, 如果你使用python2.x，那么只需要把<code>pip3</code>替换成<code>pip</code>就可以了。在本教程中，我们使用python3.4进行讲解</p> 
<h4>安装Django</h4>
<p>Django作为一个开源的python框架，已经被Python package index收录。因此我们推荐使用pip的方式安装Django。</p>
<p><code>pip3 install django</code></p>
<p>你也可以通过django源码的方式来安装，下载<a href="https://www.djangoproject.com/download/">django.1.xx.x.tar.gz</a>, 解压缩以后通过<code>python3 setup.py install</code>的方式安装，或者也可以直接把django的源码目录（django目录为顶级目录）直接丢到python path的目录下</p>
<h4>验证安装</h4>
<p>要验证是否已经为当前系统的python环境安装好了Django，打开python交互模式，运行以下代码</p>
<pre>
>>> import django
>>> print(django.get_version())
1.11
</pre>
<p>如果你看到如上所示的输出，那么恭喜你，django已经成功安装了.是不是很简单？</p>
