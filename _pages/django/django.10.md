---
layout: single
permalink: /django/example1/
title: "Django By Example 第一章"
sidebar:
  nav: "django"
# toc: true
---

本例出自由Antonio Melé编写的《Django By Example》

<div ><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>

<p>在这本书中，你将学习如何创建完整的Django项目，可以在生产环境中使用。假如你还没有安装Django，在本章的第一部分你将学习如何安装。本章会覆盖如何使用Django去创建一个简单的blog应用。本章的目的是使你对该框架的工作有个基本概念，了解不同的组件之间是如何产生交互，并且教你一些技能通过使用一些基本功能方便地创建Djang项目。你会被引导创建一个完整的项目但是不会对所有的细节都进行详细说明。不同的框架组件将在本书接下来的章节中进行介绍。<br>
本章会覆盖以下几点：</p>
<ul>
<li>安装Django并创建你的第一个项目</li>
<li>设计模型（models）并且生成模型（model）数据库迁移</li>
<li>给你的模型（models）创建一个管理站点</li>
<li>使用查询集（QuerySet）和管理器（managers）</li>
<li>创建视图（views），模板（templates）和URLs</li>
<li>给列表视图（views）添加页码</li>
<li>使用Django内置的视图（views）</li>
</ul>

### 安装Django

<p>如果你已经安装好了Django，你可以直接略过这部分跳到<em>创建你的第一个项目</em>。Django是一个Python包因此可以安装在任何的Python的环境中。如果你还没有安装Django，这里有一个快速的指南帮助你安装Django用来本地开发。</p>
<p>Django需要在Python2.7或者3版本上才能更好的工作。在本书的例子中，我们将使用Python 3。如果你使用Linux或者Max OSX，你可能已经有安装好的Python。如果你不确定你的计算机中是否安装了Python，你可以在终端中输入 <em>python</em> 来确定。如果你看到以下类似的提示，说明你的计算机中已经安装好了Python:</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">Python</span> 3<span class="hljs-selector-class">.5</span><span class="hljs-selector-class">.0</span> (<span class="hljs-selector-tag">v3</span><span class="hljs-selector-class">.5</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-pseudo">:374f501f4567</span>, <span class="hljs-selector-tag">Sep</span> 12 2015, 11<span class="hljs-selector-pseudo">:00</span><span class="hljs-selector-pseudo">:19)</span>
<span class="hljs-selector-attr">[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)]</span> <span class="hljs-selector-tag">on</span> <span class="hljs-selector-tag">darwin</span>
<span class="hljs-selector-tag">Type</span> "<span class="hljs-selector-tag">help</span>", "<span class="hljs-selector-tag">copyright</span>", "<span class="hljs-selector-tag">credits</span>" <span class="hljs-selector-tag">or</span> "<span class="hljs-selector-tag">license</span>" <span class="hljs-selector-tag">for</span> <span class="hljs-selector-tag">more</span> <span class="hljs-selector-tag">information</span>.
&gt;&gt;&gt;</code></pre>
<p>如果你计算机中安装的Python版本低于3，或者没有安装，下载并安装Python 3.5.0 从http://www.python.org/download/ <strong>（译者注：最新已经是3.6.0了，Django2.0将不再支持pytyon2.7，所以大家都从3版本以上开始学习吧）</strong>。</p>
<p>由于你使用的是Python3，所以你没必要再安装一个数据库。这个Python版本自带SQLite数据库。SQLLite是一个轻量级的数据库，你可以在Django中进行使用用来开发。如果你准备在生产环境中部署你的应用，你应该使用一个更高级的数据库，比如PostgreSQL，MySQL或Oracle。你能获取到更多的信息关于数据库和Django的集成通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/install/#database-installation" class="uri">https://docs.djangoproject.com/en/1.8/topics/install/#database-installation</a> 。</p>

### 创建一个独立的Python环境
<p>强烈建议你使用virtualenv来创建独立的Python环境，这样你可以使用不同的包版本对应不同的项目，这比直接在真实系统中安装Python包更加的实用。另一个高级之处在于当你使用virtualenv你不需要任何管理员权限来安装Python包。在终端中运行以下命令来安装virtualenv：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> virtualenv</code></pre>
<p><strong>（译者注：如果你本地有多个python版本，注意Python3的pip命令可能是pip3）</strong></p>
<p>当你安装好virtualenv之后，通过以下命令来创建一个独立的环境：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">virtualenv</span> my_env</code></pre>
<p>以上命令会创建一个包含你的Python环境的my_env/目录。当你的virtualenv被激活的时候所有已经安装的Python库都会带入 my_env/lib/python3.5/site-packages 目录中。<br>
如果你的系统自带Python2.X然后你又安装了Python3.X，你必须告诉virtualenv使用后者Python3.X。通过以下命令你可以定位Python3的安装路径然后使用该安装路径来创建virtualenv：</p>
<pre class="shell"><code class="hljs groovy">zenx\$ *which python3* 
<span class="hljs-regexp">/Library/</span>Frameworks<span class="hljs-regexp">/Python.framework/</span>Versions<span class="hljs-regexp">/3.5/</span>bin/python3
zenx\$ *virtualenv my_env -p 
<span class="hljs-regexp">/Library/</span>Frameworks<span class="hljs-regexp">/Python.framework/</span>Versions<span class="hljs-regexp">/3.5/</span>bin/python3*</code></pre>
<p>通过以下命令来激活你的virtualenv：</p>
<pre><code class="hljs bash"><span class="hljs-built_in">source</span> my_env/bin/activate</code></pre>
<p>shell提示将会附上激活的virtualenv名，被包含在括号中，如下所示：</p>
<pre><code class="hljs groovy">(my_evn)<span class="hljs-string">laptop:</span>~ zenx$</code></pre>
<p>你可以使用<em>deactivate</em>命令随时停用你的virtualenv。</p>
<p>你可以获取更多的信息关于virtualenv通过访问 <a href="https://virtualenv.pypa.io/en/latest/" class="uri">https://virtualenv.pypa.io/en/latest/</a> 。</p>
<p>在virtualenv之上，你可以使用virtualenvwrapper工具。这个工具提供一些封装用来方便的创建和管理你的虚拟环境。你可以在 <a href="http://virtualenvwrapper.readthedocs.org/en/latest/" class="uri">http://virtualenvwrapper.readthedocs.org/en/latest/</a> 下载该工具。</p>

### 使用pip安装Django

<p><strong>（译者注：请注意以下的操作都在激活的虚拟环境中使用）</strong></p>
<p>pip是安装Django的第一选择。Python3.5自带预安装的pip，你可以找到pip的安装指令通过访问 <a href="https://pip.pypa.io/en/stable/installing/" class="uri">https://pip.pypa.io/en/stable/installing/</a> 。运行以下命令通过pip安装Django：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Django==<span class="hljs-number">1.8</span>.<span class="hljs-number">6</span></code></pre>
<p>Django将会被安装在你的虚拟环境的Python的<em>site-packages/</em>目录下。</p>
<p>现在检查Django是否成功安装。在终端中运行<em>python</em>并且导入Django来检查它的版本：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; import django
<span class="hljs-meta">&gt;&gt;</span>&gt; django.VERSION
DjangoVERSION（<span class="hljs-number">1</span>, <span class="hljs-number">8</span>, <span class="hljs-number">5</span>, <span class="hljs-string">'final'</span>, <span class="hljs-number">0</span>)</code></pre>
<p>如果你获得了以上输出，Django已经成功安装在你的机器中。</p>
<p>Django也可以使用其他方式来安装。你可以找到更多的信息通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/install/" class="uri">https://docs.djangoproject.com/en/1.8/topics/install/</a> 。</p>

### 创建你的第一个项目
<p>我们的第一个项目将会是一个完整的blog站点。Django提供了一个命令允许你方便的创建一个初始化的项目文件结构。在终端中运行以下命令：</p>
<pre><code class="hljs">django-admin startproject mysite</code></pre>
<p>该命令将会创建一个名为<em>mysite</em>的项目。<br>
让我们来看下生成的项目结构：</p>
<pre><code class="hljs">mysite/
    manage.py
    mysite/
        __init__.py
        settings.py
        urls.py
        wsgi.py</code></pre>
<p>让我们来了解一下这些文件：</p>
<ul>
<li><em>manage.py</em>:一个实用的命令行，用来与你的项目进行交互。它是一个对<em>django-admin.py</em>工具的简单封装。你不需要编辑这个文件。</li>
<li><em>mysite/</em>:你的项目目录，由以下的文件组成：
<ul>
<li><strong>init</strong>.py:一个空文件用来告诉Python这个<em>mysite</em>目录是一个Python模块。</li>
<li>settings.py:你的项目的设置和配置。里面包含一些初始化的设置。</li>
<li>urls.py:你的URL模式存放的地方。这里定义的每一个URL都映射一个视图（view）。</li>
<li>wsgi.py:配置你的项目运行如同一个WSGI应用。</li>
</ul></li>
</ul>
<p>默认生成的<em>settings.py</em>文件包含一个使用一个SQLite数据库的基础配置以及一个Django应用列表，这些应用会默认添加到你的项目中。我们需要为这些初始应用在数据库中创建表。</p>
<p>打开终端执行以下命令：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">cd</span> <span class="hljs-selector-tag">mysite</span>
<span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你将会看到以下的类似输出：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">Rendering</span> <span class="hljs-selector-tag">model</span> <span class="hljs-selector-tag">states</span>... <span class="hljs-selector-tag">DONE</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">contenttypes</span><span class="hljs-selector-class">.ooo1_initial</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">admin</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">contenttypes</span><span class="hljs-selector-class">.0002_remove_content_type_name</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0002_alter_permission_name_max_length</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0003_alter_user_email_max_length</span>..<span class="hljs-selector-class">.OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0004_alter_user_username_opts</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0005_alter_user_last_login_null</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">auth</span><span class="hljs-selector-class">.0006_require_contenttypes_0002</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">sessions</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>这些初始应用表将会在数据库中创建。过一会儿你就会学习到一些关于<em>migrate</em>的管理命令。</p>

### 运行开发服务器
<p>Django自带一个轻量级的web服务器来快速运行你的代码，不需要花费额外的时间来配置一个生产服务器。当你运行Django的开发服务器，它会一直检查你的代码变化。当代码有改变，它会自动重启，将你从手动重启中解放出来。但是，它可能无法注意到一些操作，例如在项目中添加了一个新文件，所以你在某些场景下还是需要手动重启。</p>
<p>打开终端，在你的项目主目录下运行以下代码来开启开发服务器：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">runserver</span></code></pre>
<p>你会看到以下类似的输出：</p>
<pre class="shell"><code class="hljs sql">Performing system checks...
    
System <span class="hljs-keyword">check</span> <span class="hljs-keyword">identified</span> <span class="hljs-keyword">no</span> issues (<span class="hljs-number">0</span> silenced).
November <span class="hljs-number">5</span>, <span class="hljs-number">2015</span> - <span class="hljs-number">19</span>:<span class="hljs-number">10</span>:<span class="hljs-number">54</span>
Django <span class="hljs-keyword">version</span> <span class="hljs-number">1.8</span><span class="hljs-number">.6</span>, <span class="hljs-keyword">using</span> <span class="hljs-keyword">settings</span> <span class="hljs-string">'mysite.settings'</span>
<span class="hljs-keyword">Starting</span> development <span class="hljs-keyword">server</span> <span class="hljs-keyword">at</span> <span class="hljs-keyword">http</span>://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8000</span>/
Quit the <span class="hljs-keyword">server</span> <span class="hljs-keyword">with</span> CONTROL-<span class="hljs-keyword">C</span>.</code></pre>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，你会看到一个告诉你项目成功运行的页面，如下图所示：<br>
<iframe id="iframe_0.45448383937082903" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-e5ab238aa9acbef9.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.45448383937082903',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 616px; height: 150px;" frameborder="0" scrolling="no"></iframe></p>
<p>你可以指定Django在定制的host和端口上运行开发服务，或者告诉它你想要运行你的项目通过读取一个不同的配置文件。例如：你可以运行以下 manage.py命令：</p>
<pre class="shell"><code class="hljs nginx"><span class="hljs-attribute">python</span> manage.py runserver <span class="hljs-number">127.0.0.1:8001</span> \
--settings=mysite.settings</code></pre>
<p>这个命令迟早会对处理需要不同设置的多套环境启到作用。记住，这个服务器只是单纯用来开发，不适合在生产环境中使用。为了在生产环境中部署Django，你需要使用真实的web服务让它运行成一个WSGI应用例如Apache，Gunicorn或者uWSGI<strong>（译者注：强烈推荐 nginx+uwsgi+Django）</strong>。你能够获取到更多关于如何在不同的web服务中部署Django的信息，访问 <a href="https://docs.djangoproject.com/en/1.8/howto/deployment/wsgi/" class="uri">https://docs.djangoproject.com/en/1.8/howto/deployment/wsgi/</a> 。</p>
<p>本书外额外的需要下载的章节<em>第十三章,Going Live</em>包含为你的Django项目设置一个生产环境。</p>

### 项目设置

<p>让我们打开settings.py文件来看看你的项目的配置。在该文件中有许多设置是Django内置的，但这些只是所有Django可用配置的一部分。你可以通过访问 <a href="https://docs.djangoproject.com/en/1.8/ref/settings/" class="uri">https://docs.djangoproject.com/en/1.8/ref/settings/</a> 看到所有的设置和它们默认的值。</p>
<p>以下列出的设置非常值得一看：</p>
<ul>
<li>DEBUG 一个布尔型用来开启或关闭项目的debug模式。如果设置为True，当你的应用抛出一个未被捕获的异常时Django将会显示一个详细的错误页面。当你准备部署项目到生产环境，请记住一定要关闭debug模式。永远不要在生产环境中部署一个打开debug模式的站点因为那会暴露你的项目中的敏感数据。</li>
<li>ALLOWED_HOSTS 当debug模式开启或者运行测试的时候不会起作用<strong>（译者注：最新的Django版本中，不管有没有开启debug模式该设置都会启作用）</strong>。一旦你准备部署你的项目到生产环境并且关闭了debug模式，为了允许访问你的Django项目你就必须添加你的域或host在这个设置中。</li>
<li>INSTALLED_APPS 这个设置你在所有的项目中都需要编辑。这个设置告诉Django有哪些应用会在这个项目中激活。默认的，Django包含以下应用：</li>
<li>django.contrib.admin：这是一个管理站点。</li>
<li>django.contrib.auth：这是一个权限框架。</li>
<li>django.contrib.contenttypes：这是一个内容类型的框架。</li>
<li>django.contrib.sessions：这是一个会话（session）框架。</li>
<li>django.contrib.messages：这是一个消息框架。</li>
<li>django.contrib.staticfiles：这是一个用来管理静态文件的框架</li>
<li>MIDDLEWARE_CLASSES 是一个包含可执行中间件的元组。</li>
<li>ROOT_URLCONF 指明你的应用定义的主URL模式存放在哪个Python模块中。</li>
<li>DATABASES 是一个包含了所有在项目中使用的数据库的设置的字典。里面一定有一个默认的数据库。默认的配置使用的是SQLite3数据库。</li>
<li>LANGUAGE_CODE 定义Django站点的默认语言编码。</li>
</ul>
<p>不要担心你目前还看不懂这些设置的含义。你将会在之后的章节中熟悉这些设置。</p>

### 项目和应用
<p>贯穿全书，你会反复的读到项目和应用的地位。在Django中，一个项目被认为是一个安装了一些设置的Django；一个应用是一个包含模型（models），视图（views），模板（templates）以及URLs的组合。应用之间的交互通过Django框架提供的一些特定功能，并且应用可能被各种各样的项目重复使用。你可以认为项目就是你的网站，这个网站包含多个应用，例如blog，wiki或者论坛，这些应用都可以被其他的项目使用。<strong>（译者注：我去，我竟然漏翻了这一节- -|||，罪过罪过，阿米头发）</strong></p>

### 创建一个应用

<p>现在让我们创建你的第一个Django应用。我们将要创建一个勉强凑合的blog应用。在你的项目主目录下，运行以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">blog</span></code></pre>
<p>这个命令会创建blog应用的基本目录结构，如下所示：</p>
<pre class="shell"><code class="hljs">blog/
    __init__.py
    admin.py
    migrations/
        __init__.py
    models.py
    tests.py
    views.py</code></pre>
<p>这些文件的含义：</p>
<ul>
<li>admin.py: 在这儿你可以注册你的模型（models）并将它们包含到Django的管理页面中。使用Django的管理页面是可选的。</li>
<li>migrations: 这个目录将会包含你的应用的数据库迁移。Migrations允许Django跟踪你的模型（model）变化并因此来同步数据库。</li>
<li>models.py: 你的应用的数据模型（models）。所有的Django应用都需要拥有一个<em>models.py</em>文件，但是这个文件可以是空的。</li>
<li>tests.py：在这儿你可以为你的应用创建测试。</li>
<li>views.py：你的应用逻辑将会放在这儿。每一个视图（view）都会接受一个HTTP请求，处理该请求，最后返回一个响应。</li>
</ul>

### 设计blog数据架构
<p>我们将要开始为你的blog设计初始的数据模型（models）。一个模型（model）就是一个Python类，该类继承了<em>django.db.models.model</em>,在其中的每一个属性表示一个数据库字段。Django将会为<em>models.py</em>中的每一个定义的模型（model）创建一张表。当你创建好一个模型（model），Django会提供一个非常实用的API来方便的查询数据库。</p>
<p>首先，我们定义一个<em>POST</em>模型（model）。在blog应用下的<em>models.py</em>文件中添加以下内容：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> django.utils <span class="im"><span class="hljs-keyword">import</span></span> timezone
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.auth.models <span class="im"><span class="hljs-keyword">import</span></span> User


<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Post</span><span class="hljs-params">(models.Model)</span>:</span>
    STATUS_CHOICES <span class="op">=</span> (
        (<span class="st"><span class="hljs-string">'draft'</span></span>, <span class="st"><span class="hljs-string">'Draft'</span></span>),
        (<span class="st"><span class="hljs-string">'published'</span></span>, <span class="st"><span class="hljs-string">'Published'</span></span>),
    )
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">250</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">250</span></span>,
                            unique_for_date<span class="op">=</span><span class="st"><span class="hljs-string">'publish'</span></span>)
    author <span class="op">=</span> models.ForeignKey(User,
                                related_name<span class="op">=</span><span class="st"><span class="hljs-string">'blog_posts'</span></span>)
    body <span class="op">=</span> models.TextField()
    publish <span class="op">=</span> models.DateTimeField(default<span class="op">=</span>timezone.now)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    status <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">10</span></span>,
                                choices<span class="op">=</span>STATUS_CHOICES,
                                default<span class="op">=</span><span class="st"><span class="hljs-string">'draft'</span></span>)
                            
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'-publish'</span></span>,)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.title</code></pre></div>
<p>这就是我们给blog帖子使用的基础模型（model）。让我们来看下刚才在这个模型（model）中定义的各个字段含义：</p>
<ul>
<li>title： 这个字段对应帖子的标题。它是<em>CharField</em>，在SQL数据库中会被转化成VARCHAR。</li>
<li>slug：这个字段将会在URLs中使用。slug就是一个短标签，该标签只包含字母，数字，下划线或连接线。我们将通过使用slug字段给我们的blog帖子构建漂亮的，友好的URLs。我们给该字段添加了<em>unique_for_date</em>参数，这样我们就可以使用日期和帖子的slug来为所有帖子构建URLs。在相同的日期中Django会阻止多篇帖子拥有相同的slug。</li>
<li>author：这是一个<em>ForeignKey</em>。这个字段定义了一个多对一（many-to-one）的关系。我们告诉Django一篇帖子只能由一名用户编写，一名用户能编写多篇帖子。根据这个字段，Django将会在数据库中通过有关联的模型（model）主键来创建一个外键。在这个场景中，我们关联上了Django权限系统的<em>User</em>模型（model）。我们通过<em>related_name</em>属性指定了从<em>User</em>到<em>Post</em>的反向关系名。我们将会在之后学习到更多关于这方面的内容。</li>
<li>body：这是帖子的主体。它是<em>TextField</em>，在SQL数据库中被转化成<em>TEXT</em>。</li>
<li>publish：这个日期表明帖子什么时间发布。我们使用Djnago的<em>timezone</em>的<em>now</em>方法来设定默认值。This is just a timezone-aware datetime.now<strong>（译者注：这句该咋翻译好呢）</strong>。</li>
<li>created：这个日期表明帖子什么时间创建。因为我们在这儿使用了<em>auto_now_add</em>，当一个对象被创建的时候这个字段会自动保存当前日期。</li>
<li>updated：这个日期表明帖子什么时候更新。因为我们在这儿使用了<em>auto_now</em>，当我们更新保存一个对象的时候这个字段将会自动更新到当前日期。</li>
<li>status：这个字段表示当前帖子的展示状态。我们使用了一个<em>choices</em>参数，这样这个字段的值只能是给予的选择参数中的某一个值。<strong>（译者注：传入元组，比如<code>(1,2)</code>，那么该字段只能选择1或者2，没有其他值可以选择）</strong></li>
</ul>
<p>就像你所看到的的，Django内置了许多不同的字段类型给你使用，这样你就能够定义你自己的模型（models）。通过访问 <a href="https://docs.djangoproject.com/en/1.8/ref/models/fields/" class="uri">https://docs.djangoproject.com/en/1.8/ref/models/fields/</a> 你可以找到所有的字段类型。</p>
<p>在模型（model）中的类<em>Meta</em>包含元数据。我们告诉Django查询数据库的时候默认返回的是根据<em>publish</em>字段进行降序排列过的结果。我们使用负号来指定进行降序排列。</p>
<p><em><strong>str</strong>()</em>方法是当前对象默认的可读表现。Django将会在很多地方用到它例如管理站点中。</p>
<blockquote>
<p>如果你之前使用过Python2.X，请注意在Python3中所有的strings都使用unicode，因此我们只使用<em><strong>str</strong>()</em>方法。<em><strong>unicode</strong>()</em>方法已经废弃。<strong>（译者注：Python3大法好，Python2别再学了，直接学Python3吧）</strong></p>
</blockquote>
<p>在我们处理日期之前，我们需要下载<em>pytz</em>模块。这个模块给Python提供时区的定义并且SQLite也需要它来对日期进行操作。在终端中输入以下命令来安装<em>pytz</em>：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> pytz</code></pre>
<p>Django内置对时区日期处理的支持。你可以在你的项目中的<em>settings.py</em>文件中通过<em>USE_TZ</em>来设置激活或停用对时区的支持。当你通过<em>startproject</em>命令来创建一个新项目的时候这个设置默认为<em>True</em>。</p>

### 激活你的应用

<p>为了让Django能保持跟踪你的应用并且根据你的应用中的模型（models）来创建数据库表，我们必须激活你的应用。因此，编辑<em>settings.py</em>文件，在<em>INSTALLED_APPS</em>设置中添加<em>blog</em>。看上去如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> ( 
    <span class="st"><span class="hljs-string">'django.contrib.admin'</span></span>,    
    <span class="co"><span class="hljs-string">'django.contrib.auth'</span></span>, 
    <span class="co"><span class="hljs-string">'django.contrib.contenttypes'</span></span>, 
    <span class="co"><span class="hljs-string">'django.contrib.sessions'</span></span>, 
    <span class="co"><span class="hljs-string">'django.contrib.messages'</span></span>, 
    <span class="co"><span class="hljs-string">'django.contrib.staticfiles'</span></span>,
    <span class="co"><span class="hljs-string">'blog'</span></span>,
 )</code></pre></div>
<p><strong>（译者注：该设置中应用的排列顺序也会对项目的某些方面产生影响，具体情况后几章会有介绍，这里提醒下）</strong></p>
<p>现在Django已经知道在项目中的我们的应用是激活状态并且将会对其中的模型（models）进行自审。</p>

### 创建和进行数据库迁移

<p>让我们为我们的模型（model）在数据库中创建一张数据表格。Django自带一个数据库迁移（migration）系统来跟踪你对模型（models）的修改，然后会同步到数据库。<em>migrate</em>命令会应用到所有在<em>INSTALLED_APPS</em>中的应用，它会根据当前的模型（models）和数据库迁移（migrations）来同步数据库。</p>
<p>首先，我们需要为我们刚才创建的新模型（model）创建一个数据库迁移（migration）。在你的项目主目录下，执行以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span> <span class="hljs-selector-tag">blog</span></code></pre>
<p>你会看到以下输出:</p>
<pre class="shell"><code class="hljs python">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'blog'</span>:
    <span class="hljs-number">0001</span>_initial.py;
        - Create model Post</code></pre>
<p>Django在blog应用下的migrations目录中创建了一个<em>0001——initial.py</em>文件。你可以打开这个文件来看下一个数据库迁移的内容。</p>
<p>让我们来看下Django根据我们的模型（model）将会为在数据库中创建的表而执行的SQL代码。<em>sqlmigrate</em>命令带上数据库迁移（migration）的名字将会返回它们的SQL，但不会立即去执行。运行以下命令来看下输出：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">sqlmigrate</span> <span class="hljs-selector-tag">blog</span> 0001</code></pre>
<p>输出类似如下：</p>
<pre class="shell"><code class="hljs sql"><span class="hljs-keyword">BEGIN</span>;
<span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">TABLE</span> <span class="hljs-string">"blog_post"</span> (<span class="hljs-string">"id"</span> <span class="hljs-built_in">integer</span> <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span> PRIMARY <span class="hljs-keyword">KEY</span> AUTOINCREMENT, <span class="hljs-string">"title"</span> <span class="hljs-built_in">varchar</span>(<span class="hljs-number">250</span>) <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"slug"</span> <span class="hljs-built_in">varchar</span>(<span class="hljs-number">250</span>) <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"body"</span> <span class="hljs-built_in">text</span> <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"publish"</span> datetime <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"created"</span> datetime <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"updated"</span> datetime <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"status"</span> <span class="hljs-built_in">varchar</span>(<span class="hljs-number">10</span>) <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span>, <span class="hljs-string">"author_id"</span> <span class="hljs-built_in">integer</span> <span class="hljs-keyword">NOT</span> <span class="hljs-literal">NULL</span> <span class="hljs-keyword">REFERENCES</span> <span class="hljs-string">"auth_user"</span> (<span class="hljs-string">"id"</span>));
<span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">INDEX</span> <span class="hljs-string">"blog_post_2dbcba41"</span> <span class="hljs-keyword">ON</span> <span class="hljs-string">"blog_post"</span> (<span class="hljs-string">"slug"</span>);
<span class="hljs-keyword">CREATE</span> <span class="hljs-keyword">INDEX</span> <span class="hljs-string">"blog_post_4f331e2f"</span> <span class="hljs-keyword">ON</span> <span class="hljs-string">"blog_post"</span> (<span class="hljs-string">"author_id"</span>);
<span class="hljs-keyword">COMMIT</span>;</code></pre>
<p>Django会根据你正在使用的数据库进行以上精准的输出。以上SQL语句是为SQLite数据库准备的。如你所见，Django生成的表名前缀为应用名之后跟上模型（model）的小写（blog_post），但是你也可以通过在模型（models）的<em>Meta</em>类中使用<em>db_table</em>属性来指定表名。Django会自动为每个模型（model）创建一个主键，但是你也可以通过在模型（model）中的某个字段上设置<em>primarry_key=True</em>来指定主键。</p>
<p>让我们根据新模型（model）来同步数据库。运行以下的命令来应用已存在的数据迁移（migrations）：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你应该会看到以下行跟在输出的末尾：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">blog</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>我们刚刚为<em>INSTALLED_APPS</em>中所有的应用进行了数据库迁移（migrations），包括我们的<em>blog</em>应用。在进行了数据库迁移（migrations）之后，数据库会反映我们模型的当前状态。</p>
<p>如果为了添加，删除，或是改变了存在的模型（models）中字段，或者你添加了新的模型（models）而编辑了<em>models.py</em>文件，你都需要通过使用<em>makemigrations</em>命令做一次新的数据库迁移（migration）。数据库迁移（migration）允许Django来保持对模型（model）改变的跟踪。之后你必须通过<em>migrate</em>命令来保持数据库与我们的模型（models）同步。</p>

### 为你的模型（models）创建一个管理站点(Admin)

<p>现在我们已经定义好了<em>Post</em>模型（model），我们将要创建一个简单的管理站点来管理blog帖子。Django内置了一个管理接口，该接口对编辑内容非常的有用。这个Django管理站点会根据你的模型（model）元数据进行动态构建并且提供一个可读的接口来编辑内容。你可以对这个站点进行自由的定制，配置你的模型（models）在其中如何进行显示。</p>
<p>请记住，<em>django.contrib.admin</em>已经被包含在我们项目的<em>INSTALLED_APPS</em>设置中，我们不需要再额外添加。</p>

### 创建一个超级用户

<p>首先，我们需要创建一名用户来管理这个管理站点。运行以下的命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">createsuperuser</span></code></pre>
<p>你会看下以下输出。输入你想要的用户名，邮箱和密码：</p>
<pre class="shell"><code class="hljs sql">Username (leave blank to <span class="hljs-keyword">use</span> <span class="hljs-string">'admin'</span>): <span class="hljs-keyword">admin</span>
Email address: <span class="hljs-keyword">admin</span>@<span class="hljs-keyword">admin</span>.com
<span class="hljs-keyword">Password</span>: ********
<span class="hljs-keyword">Password</span> (again): ********
Superuser created successfully.</code></pre>

### Django管理站点
<p>现在，通过<code>python manage.py runserver</code>命令来启动开发服务器，之后在浏览器中打开 <a href="http://127.0.0.1:8000/admin/" class="uri">http://127.0.0.1:8000/admin/</a> 。你会看到管理站点的登录页面，如下所示：<br>
<iframe id="iframe_0.5611286702084075" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-e2479f1cb20b1d5f.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.5611286702084075',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 336px; height: 250px;" frameborder="0" scrolling="no"></iframe></p>
<p>使用你在上一步中创建的超级用户信息进行登录。你将会看到管理站点的首页，如下所示：<br>
<iframe id="iframe_0.3293238625245323" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-80c83f3a385f13c1.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.3293238625245323',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 615px; height: 119px;" frameborder="0" scrolling="no"></iframe></p>
<p><em>Group</em>和<em>User</em> 模型（models） 位于<em>django.contrib.auth</em>，是Django权限管理框架的一部分。如果你点击<em>Users</em>，你将会看到你之前创建的用户信息。你的blog应用的<em>Post</em>模型（model）和<em>User</em>（model）关联在了一起。记住，它们是通过<em>author</em>字段进行关联的。</p>

### 在管理站点中添加你的模型（models）

<p>让我们在管理站点中添加你的blog模型（models）。编辑blog应用下的<em>admin.py</em>文件，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Post

admin.site.register(Post)</code></pre></div>
<p>现在，在浏览器中刷新管理站点。你会看到你的<em>Post</em>模型（model）已经在页面中展示，如下所示：<br>
<iframe id="iframe_0.18670075318742563" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-52e408aa45e35c1f.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.18670075318742563',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 622px; height: 136px;" frameborder="0" scrolling="no"></iframe></p>
<p>这很简单，对吧？当你在Django的管理页面注册了一个模型（model），Django会通过对你的模型（models）进行内省然后提供给你一个非常友好有用的接口，这个接口允许你非常方便的排列，编辑，创建，以及删除对象。</p>
<p>点击<em>Posts</em>右侧的<em>Add</em>链接来添加一篇新帖子。你将会看到Django根据你的模型（model）动态生成了一个表单，如下所示：<br>
<iframe id="iframe_0.6397721579927427" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-392e41ac7cb34cda.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.6397721579927427',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 622px; height: 329px;" frameborder="0" scrolling="no"></iframe></p>
<p>Django给不同类型的字段使用了不同的表单控件。即使是复杂的字段例如<em>DateTimeField</em>也被展示成一个简单的接口类似一个JavaScript日期选择器。</p>
<p>填写好表单然后点击<em>Save</em>按钮。你会被重定向到帖子列页面并且得到一条帖子成功创建的提示，如下所示：<br>
<iframe id="iframe_0.2706711216611424" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-879a2158c1c80ae9.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.2706711216611424',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 624px; height: 170px;" frameborder="0" scrolling="no"></iframe></p>

### 定制models的展示形式
<p>现在我们来看下如何定制管理站点。编辑blog应用下的<em>admin.py</em>文件，使之如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Post

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">PostAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> (<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'author'</span></span>, <span class="st"><span class="hljs-string">'publish'</span></span>,
                    <span class="co"><span class="hljs-string">'status'</span></span>)
admin.site.register(Post, PostAdmin)</code></pre></div>
<p>我们使用继承了<em>ModelAdmin</em>的定制类来告诉Django管理站点中需要注册我们自己的模型（model）。在这个类中，我们可以包含一些关于如何在管理站点中展示模型（model）的信息以及如何与该模型（model）进行交互。<em>list_display</em>属性允许你在设置一些你想要在管理对象列表页面显示的模型（model）字段。</p>
<p>让我们通过更多的选项来定制管理模型（model），如使用以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">PostAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> (<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'author'</span></span>, <span class="st"><span class="hljs-string">'publish'</span></span>,
                    <span class="co"><span class="hljs-string">'status'</span></span>)
    list_filter <span class="op">=</span> (<span class="st"><span class="hljs-string">'status'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'publish'</span></span>, <span class="st"><span class="hljs-string">'author'</span></span>)
    search_fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'body'</span></span>)
    prepopulated_fields <span class="op">=</span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'title'</span></span>,)}
    raw_id_fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'author'</span></span>,)
    date_hierarchy <span class="op">=</span> <span class="st"><span class="hljs-string">'publish'</span></span>
    ordering <span class="op">=</span> [<span class="st"><span class="hljs-string">'status'</span></span>, <span class="st"><span class="hljs-string">'publish'</span></span>]</code></pre></div>
<p>回到浏览器刷新管理站点页面，现在应该如下所示：<br>
<iframe id="iframe_0.035608481350103416" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-3b8a79f28e1a04de.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.035608481350103416',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 607px; height: 300px;" frameborder="0" scrolling="no"></iframe></p>
<p>你可以看到帖子列页面中展示的字段都是你在<em>list-dispaly</em>属性中指定的。这个列页面现在包含了一个右侧边栏允许你根据<em>list_filter</em>属性中指定的字段来过滤返回结果。一个搜索框也应用在页面中。这是因为我们还通过使用<em>search_fields</em>属性定义了一个搜索字段列。在搜索框的下方，有个可以通过时间层快速导航的栏，该栏通过定义<em>date_hierarchy</em>属性出现。你还能看到这些帖子默认的通过<em>Status</em>和<em>Publish</em>列进行排序。这是因为你通过使用<em>ordering</em>属性指定了默认排序。</p>
<p>现在，点击<em>Add post</em>链接。你还会在这儿看到一些改变。当你输入完成新帖子的标题，<em>slug</em>字段将会自动填充。我们通过使用<em>prepoupulated_fields</em>属性告诉Django通过输入的标题来填充<em>slug</em>字段。同时，现在的<em>author</em>字段展示显示为了一个搜索控件，这样当你的用户量达到成千上万级别的时候比再使用下拉框进行选择更加的人性化，如下图所示：<br>
<iframe id="iframe_0.5999441371314831" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-f89c73f0b51aba4b.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.5999441371314831',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 305px; height: 39px;" frameborder="0" scrolling="no"></iframe></p>
<p>通过短短的几行代码，我们就在管理站点中自定义了我们的模型（model）的展示形式。还有更多的方式可以用来定制Django的管理站点。在这本书的后面，我们还会进一步讲述。</p>

### 使用查询集（QuerySet）和管理器（managers）
<p>现在，你已经有了一个完整功能的管理站点来管理你的blog内容，是时候学习如何从数据库中检索信息并且与这些信息进行交互了。Django自带了一个强大的数据库抽象API可以让你轻松的创建，检索，更新以及删除对象。Django的<em>Object-relational Mapper(ORM)</em>可以兼容MySQL,PostgreSQL,SQLite以及Oracle。请记住你可以在你项目下的<em>setting.py</em>中编辑<em>DATABASES</em>设置来指定数据库。Django可以同时与多个数据库进行工作，这样你可以编写数据库路由通过任何你喜欢的方式来操作数据。</p>
<p>一旦你创建好了你的数据模型（models），Django会提供你一个API来与它们进行交互。你可以找到数据模型（model）的官方参考文档通过访问 <a href="https://docs.djangoproject.com/en/1.8/ref/models/" class="uri">https://docs.djangoproject.com/en/1.8/ref/models/</a> 。</p>

### 创建对象
<p>打开终端运行以下命令来打开Python shell：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">shell</span></code></pre>
<p>然后依次输入以下内容：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; from django.contrib.auth.models import User
<span class="hljs-meta">&gt;&gt;</span>&gt; from blog.models import Post
<span class="hljs-meta">&gt;&gt;</span>&gt; user = User.objects.get(username=<span class="hljs-string">'admin'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; post = Post.objects.create(title=<span class="hljs-string">'One more post'</span>,
                        slug=<span class="hljs-string">'one-more-post'</span>,
                        body=<span class="hljs-string">'Post body.'</span>,
                        author=user)
<span class="hljs-meta">&gt;&gt;</span>&gt; post.save()</code></pre>
<p>让我们来研究下这些代码做了什么。首先，我们取回了一个username是<em>admin</em>的用户对象：</p>
<pre><code class="hljs ini"><span class="hljs-attr">user</span> = User.objects.get(username=<span class="hljs-string">'admin'</span>)</code></pre>
<p><code>get()</code>方法允许你从数据库取回一个单独的对象。注意这个方法只希望在查询中有唯一的一个匹配。如果在数据库中没有返回结果，这个方法会抛出一个<em>DoesNotExist</em>异常，如果数据库返回多个匹配结果，将会抛出一个<em>MultipleObjectsReturned</em>异常。当查询执行的时候，所有的异常都是模型（model）类的属性。</p>
<p>接着，我们来创建一个拥有定制标题标题，slug和内容的<em>Post</em>实例，然后我们设置之前取回的user胃这篇帖子的作者如下所示：</p>
<pre><code class="hljs lisp">post = Post(<span class="hljs-name">title=</span>'Another post', slug='another-post', body='Postbody.', author=user)</code></pre>
<blockquote>
<p>这个对象只是存在内存中不会执行到数据库中</p>
</blockquote>
<p>最后，我们通过使用<em>save()</em>方法来保存该对象到数据库中：</p>
<pre><code class="hljs fortran">post.<span class="hljs-keyword">save</span>()</code></pre>
<p>这步操作将会执行一段SQL的插入语句。我们已经知道如何在内存中创建一个对象并且之后才在数据库中进行插入，但是我们也可以通过使用<em>create()</em>方法直接在数据库中创建对象，如下所示：</p>
<pre><code class="hljs python">Post.objects.create(title=<span class="hljs-string">'One more post'</span>, slug=<span class="hljs-string">'one-more-post'</span>,body=<span class="hljs-string">'Post body.'</span>, author=user)</code></pre>

### 更新对象

<p>现在，改变这篇帖子的标题并且再次保存对象：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; post.title = <span class="hljs-string">'New title'</span>
<span class="hljs-meta">&gt;&gt;</span>&gt; post.save()</code></pre>
<p>这一次，<em>save()</em>方法执行了一条更新语句。</p>
<blockquote>
<p>你对对象的改变一直存在内存中直到你执行到<em>save()</em>方法。</p>
</blockquote>

### 取回对象

<p>Django的<em>Object-relational mapping(ORM)</em>是基于查询集（QuerySet）。查询集（QuerySet）是从你的数据库中根据一些过滤条件范围取回的结果对象进行的采集。你已经知道如何通过<em>get()</em>方法从数据库中取回单独的对象。如你所见：我们通过<code>Post.objects.get()</code>来使用这个方法。每一个Django模型（model）至少有一个管理器（manager），默认管理器（manager）叫做<em>objects</em>。你通过使用你的模型（models）的管理器（manager）就能获得一个查询集（QuerySet）对象。获取一张表中的所有对象，你只需要在默认的<em>objects</em>管理器（manager）上使用<em>all()</em>方法即可，如下所示：</p>
<pre><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; all_posts = Post.objects.all()</code></pre>
<p>这就是我们如何创建一个用于返回数据库中所有对象的查询集（QuerySet）。注意这个查询集（QuerySet）并还没有执行。Django的查询集（QuerySets）是惰性（lazy）的，它们只会被动的去执行。这样的行为可以保证查询集（QuerySet）非常有效率。如果我们没有把查询集（QuerySet）设置给一个变量，而是直接在Python shell中编写，因为我们迫使它输出结果，这样查询集（QuerySet）的SQL语句将立马执行：</p>
<pre><code class="hljs css">&gt;&gt;&gt; <span class="hljs-selector-tag">Post</span><span class="hljs-selector-class">.objects</span><span class="hljs-selector-class">.all</span>()</code></pre>

### 使用filter()方法

<p>为了过滤查询集（QuerySet），你可以在管理器（manager）上使用<code>filter()</code>方法。例如，我们可以返回所有在2015年发布的帖子，如下所示：</p>
<pre><code class="hljs swift"><span class="hljs-type">Post</span>.objects.<span class="hljs-built_in">filter</span>(publish__year=<span class="hljs-number">2015</span>)</code></pre>
<p>你也可以使用多个字段来进行过滤。例如，我们可以返回2015年发布的所有作者用户名为<em>admin</em>的帖子，如下所示：</p>
<pre><code class="hljs swift"><span class="hljs-type">Post</span>.objects.<span class="hljs-built_in">filter</span>(publish__year=<span class="hljs-number">2015</span>, author__username='admin')</code></pre>
<p>上面的写法和下面的写法产生的结果是一致的：</p>
<pre><code class="hljs swift"><span class="hljs-type">Post</span>.objects.<span class="hljs-built_in">filter</span>(publish__year=<span class="hljs-number">2015</span>).<span class="hljs-built_in">filter</span>(author__username='admin')</code></pre>
<blockquote>
<p>我们构建了字段的查找方法，通过使用两个下划线<code>(publish__year)</code>来查询，除此以外我们也可以通过使用两个下划线<code>(author__username)</code>访问关联的模型（model）字段。</p>
</blockquote>

### 使用exclude()

<p>你可以在管理器（manager）上使用<em>exclude()</em>方法来排除某些返回结果。例如：我们可以返回所有2015年发布的帖子但是这些帖子的题目开头不能是<em>Why</em>:</p>
<pre><code class="hljs swift"><span class="hljs-type">Post</span>.objects.<span class="hljs-built_in">filter</span>(publish__year=<span class="hljs-number">2015</span>).exclude(title__startswith='<span class="hljs-type">Why'</span>)</code></pre>

### 使用order_by()

<p>通过在管理器（manager）上使用<em>order_by()</em>方法来对不同的字段进行排序，你可以对结果进行排序。例如：你可以取回所有对象并通过它们的标题进行排序：</p>
<pre><code class="hljs python">Post.objects.order_by(<span class="hljs-string">'title'</span>)</code></pre>
<p>默认是升序。你可以通过负号来指定使用降序，如下所示：</p>
<pre><code class="hljs python">Post.objects.order_by(<span class="hljs-string">'-title'</span>)</code></pre>

### 删除对象

<p>如果你想删除一个对象，你可以对对象实例进行下面的操作：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">post <span class="op">=</span> Post.objects.get(<span class="bu">id</span><span class="op">=</span><span class="dv"><span class="hljs-number">1</span></span>)
post.delete()</code></pre></div>
<blockquote>
<p>请注意，删除对象也将删除任何的依赖关系</p>
</blockquote>

### 查询集（QuerySet）什么时候会执行

<p>只要你喜欢，你可以连接许多的过滤给查询集（QuerySet）而且不会立马在数据库中执行直到这个查询集（QuerySet）被执行。查询集（QuerySet）只有在以下情况中才会执行：<br>
* 在你第一次迭代它们的时候<br>
* 当你对它们的实例进行切片：例如<code>Post.objects.all()[:3]</code><br>
* 当你对它们进行了打包或缓存<br>
* 当你对它们调用了<code>repr()</code>或<code>len()</code>方法<br>
* 当你明确的对它们调用了<code>list()</code>方法<br>
* 当你在一个声明中测试它，例如<em>bool()</em>, or, and, or if</p>

### 创建model manager

<p>我们之前提到过, <em>objects</em>是每一个模型（models）的默认管理器（manager），它会返回数据库中所有的对象。但是我们也可以为我们的模型（models）定义一些定制的管理器（manager）。我们准备创建一个定制的管理器（manager）来返回所有状态为已发布的帖子。</p>
<p>有两种方式可以为你的模型（models）添加管理器（managers）：你可以添加额外的管理器（manager）方法或者继承管理器（manager）的查询集（QuerySets）进行修改。第一种方法类似<code>Post.objects.my_manager()</code>,第二种方法类似<code>Post.my_manager.all()</code>。我们的管理器（manager）将会允许我们返回所有帖子通过使用<code>Post.published</code>。</p>
<p>编辑你的blog应用下的<em>models.py</em>文件添加如下代码来创建一个管理器（manager）:</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">PublishedManager</span><span class="hljs-params">(models.Manager)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_queryset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(PublishedManager,
                    <span class="va">self</span>).get_queryset().<span class="bu">filter</span>(status<span class="op">=</span><span class="st"><span class="hljs-string">'published'</span></span>)
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Post</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="co"><span class="hljs-comment"># ...</span></span>
    objects <span class="op">=</span> models.Manager() <span class="co"><span class="hljs-comment"># The default manager.</span></span>
    published <span class="op">=</span> PublishedManager() <span class="co"><span class="hljs-comment"># Our custom manager.</span></span></code></pre></div>
<p><code>get_queryset()</code>是返回执行过的查询集（QuerySet）的方法。我们通过使用它来包含我们定制的过滤到完整的查询集（QuerySet）中。我们定义我们定制的管理器（manager）然后添加它到<em>Post</em> 模型（model）中。我们现在可以来执行它。例如，我们可以返回所有标题开头为<em>Who</em>的并且是已经发布的帖子:</p>
<pre><code class="hljs delphi">Post.<span class="hljs-keyword">published</span>.filter(title__startswith=<span class="hljs-string">'Who'</span>)</code></pre>

### 构建列和详情视图（views）

<p>现在你已经学会了一些如何使用ORM的基本知识，你已经准备好为blog应用创建视图（views）了。一个Django视图（view）就是一个Python方法，它可以接收一个web请求然后返回一个web响应。在视图（views）中通过所有的逻辑处理返回期望的响应。</p>
<p>首先我们会创建我们的应用视图（views），然后我们将会为每个视图（view）定义一个URL模式，我们将会创建HTML模板（templates）来渲染这些视图（views）生成的数据。每一个视图（view）都会渲染模板（template）传递变量给它然后会返回一个经过渲染输出的HTTP响应。</p>

### 创建列和详情views

<p>让我们开始创建一个视图（view）来展示帖子列。编辑你的blog应用下中<em>views.py</em>文件，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Post
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_list</span><span class="hljs-params">(request)</span>:</span>
    posts <span class="op">=</span> Post.published.<span class="bu">all</span>()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                  <span class="st"><span class="hljs-string">'blog/post/list.html'</span></span>,
                  {<span class="st"><span class="hljs-string">'posts'</span></span>: posts})</code></pre></div>
<p>你刚创建了你的第一个Django视图（view）。<em>post_list</em>视图（view）将<em>request</em>对象作为唯一的参数。记住所有的的视图（views）都有需要这个参数。在这个视图（view）中，我们获取到了所有状态为已发布的帖子通过使用我们之前创建的<em>published</em>管理器（manager）。</p>
<p>最后，我们使用Django提供的快捷方法<em>render()</em>通过给予的模板（template）来渲染帖子列。这个函数将<em>request</em>对象作为参数，模板（template）路径以及变量来渲染的给予的模板（template）。它返回一个渲染文本（一般是HTML代码）<em>HttpResponse</em>对象。<em>render()</em>方法考虑到了请求内容，这样任何模板（template）内容处理器设置的变量都可以带入给予的模板（template）中。你会在<em>第三章，扩展你的blog应用</em>学习到如何使用它们。</p>
<p>让我们创建第二个视图（view）来展示一篇单独的帖子。添加如下代码到<em>views.py</em>文件中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_detail</span><span class="hljs-params">(request, year, month, day, post)</span>:</span>
    post <span class="op">=</span> get_object_or_404(Post, slug<span class="op">=</span>post,
                                   status<span class="op">=</span><span class="st"><span class="hljs-string">'published'</span></span>,
                                   publish__year<span class="op">=</span>year,
                                   publish__month<span class="op">=</span>month,
                                   publish__day<span class="op">=</span>day)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                  <span class="st"><span class="hljs-string">'blog/post/detail.html'</span></span>,
                  {<span class="st"><span class="hljs-string">'post'</span></span>: post})</code></pre></div>
<p>这是一个帖子详情视图（view）。这个视图（view）使用<em>year，month，day</em>以及<em>post</em>作为参数通过给予slug和日期来获取到一篇已经发布的帖子。请注意，当我们创建<em>Post</em>模型（model）的时候，我们给slgu字段添加了<em>unique_for_date</em>参数。这样我们可以确保在给予的日期中只有一个帖子会带有一个slug，因此，我们能通过日期和slug取回单独的帖子。在这个详情视图（view）中，我们通过使用<em>get_object_or_404()</em>快捷方法来检索期望的<em>Post</em>。这个函数能取回匹配给予的参数的对象，或者当没有匹配的对象时返回一个HTTP 404（Not found）异常。最后，我们使用<em>render()</em>快捷方法来使用一个模板（template）去渲染取回的帖子。</p>

### 为你的视图（views）添加URL模式
<p>一个URL模式是由一个Python正则表达，一个视图（view），一个全项目范围内的命名组成。Django在运行中会遍历所有URL模式直到第一个匹配的请求URL才停止。之后，Django导入匹配的URL模式中的视图（view）并执行它，使用关键字或指定参数来执行一个<em>HttpRequest</em>类的实例。<br>
如果你之前没有接触过正则表达式，你需要去稍微了解下，通过访问 <a href="https://docs.python.org/3/howto/regex.html" class="uri">https://docs.python.org/3/howto/regex.html</a> 。</p>
<p>在blog应用目录下创建一个<em>urls.py</em>文件，输入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views
urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># post views</span></span>
    url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, views.post_list, name<span class="op">=</span><span class="st"><span class="hljs-string">'post_list'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;year&gt;\d</span></span><span class="sc"><span class="hljs-string">{4}</span></span><span class="vs"><span class="hljs-string">)/(?P&lt;month&gt;\d</span></span><span class="sc"><span class="hljs-string">{2}</span></span><span class="vs"><span class="hljs-string">)/(?P&lt;day&gt;\d</span></span><span class="sc"><span class="hljs-string">{2}</span></span><span class="vs"><span class="hljs-string">)/'</span></span><span class="op">\</span>
        <span class="co"><span class="hljs-string">r'(?P&lt;post&gt;[-\w]+)/$'</span></span>,
        views.post_detail,
        name<span class="op">=</span><span class="st"><span class="hljs-string">'post_detail'</span></span>),
]</code></pre></div>
<p>第一条URL模式没有带入任何参数，它映射到<em>post_list</em>视图（view）。第二条URL模式带上了以下4个参数映射到<em>post_detail</em>视图（view）中。让我们看下这个URL模式中的正则表达式：</p>
<ul>
<li>year：需要四位数</li>
<li>month：需要两位数。不及两位数，开头带上0，比如 01，02</li>
<li>day：需要两位数。不及两位数开头带上0</li>
<li>post：可以由单词和连字符组成</li>
</ul>
<blockquote>
<p>为每一个应用创建单独的<em>urls.py</em>文件是最好的方法，可以保证你的应用能给别的项目再度使用。</p>
</blockquote>
<p>现在你需要将你blog中的URL模式包含到项目的主URL模式中。编辑你的项目中的<em>mysite</em>文件夹中的<em>urls.py</em>文件，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> include, url
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)), 
    url(<span class="vs"><span class="hljs-string">r'^blog/'</span></span>, include(<span class="st"><span class="hljs-string">'blog.urls'</span></span>,
        namespace<span class="op">=</span><span class="st"><span class="hljs-string">'blog'</span></span>,
        app_name<span class="op">=</span><span class="st"><span class="hljs-string">'blog'</span></span>)),
]</code></pre></div>
<p>通过这样的方式，你告诉Django在<em>blog/</em>路径下包含了blog应用中的<em>urls.py</em>定义的URL模式。你可以给它们一个命名空间叫做<em>blog</em>，这样你可以方便的引用这个URLs组。</p>

### 模型（models）的标准URLs
<p>你可以使用之前定义的<em>post_detail</em> URL给<em>Post</em>对象构建标准URL。Django的惯例是给模型（model）添加<em>get_absolute_url()</em>方法用来返回一个对象的标准URL。在这个方法中，我们使用<em>reverse()</em>方法允许你通过它们的名字和可选的参数来构建URLS。编辑你的<em>models.py</em>文件添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse
Class Post(models.Model):
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_absolute_url</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> reverse(<span class="st"><span class="hljs-string">'blog:post_detail'</span></span>,
                        args<span class="op">=</span>[<span class="va">self</span>.publish.year,
                              <span class="va">self</span>.publish.strftime(<span class="st"><span class="hljs-string">'%m'</span></span>),
                              <span class="va">self</span>.publish.strftime(<span class="st"><span class="hljs-string">'</span></span><span class="sc"><span class="hljs-string">%d</span></span><span class="st"><span class="hljs-string">'</span></span>),
                              <span class="va">self</span>.slug])</code></pre></div>
<p>请注意，我们通过使用<em>strftime()</em>方法来保证个位数的月份和日期需要带上0来构建URL<strong>（译者注：也就是01,02,03）</strong>。我们将会在我们的模板（templates）中使用<em>get_absolute_url()</em>方法。</p>

### 为你的视图（views）创建模板（templates）
<p>我们为我们的应用创建了视图（views）和URL模式。现在该添加模板（templates）来展示界面友好的帖子了。</p>
<p>在你的blog应用目录下创建以下目录结构和文件：</p>
<pre><code class="hljs cpp">templates/
    blog/
        base.html
        post/
            <span class="hljs-built_in">list</span>.html
            detail.html</code></pre>
<p>以上就是我们的模板（templates）的文件目录结构。<em>base.html</em>文件将会包含站点主要的HTML结构以及分割内容区域和一个导航栏。<em>list.html</em>和<em>detail.html</em>文件会继承<em>base.html</em>文件来渲染各自的blog帖子列和详情视图（view）。</p>
<p>Django有一个强大的模板（templates）语言允许你指定数据的如何进行展示。它基于模板标签（templates tags）， 例如 <code><span>{</span><span>%</span> tag <span>%</span><span>}</span></code>, <code></span>{</span><span>{</span> variable <span>}</span><span>}</span></code>以及模板过滤器（templates filters），可以对变量进行过滤，例如 <code>{{ variable|filter }}</code>。你可以通过访问 <a href="https://docs.djangoproject.com/en/1.8/" class="uri">https://docs.djangoproject.com/en/1.8/</a> ref/templates/builtins/ 找到所有的内置模板标签（templates tags）和过滤器（filters）。</p>
<p>让我们来编辑<em>base.html</em>文件并添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load staticfiles <span>%</span><span>}</span>
<span class="dt"><span class="hljs-meta">&lt;!DOCTYPE </span></span><span class="hljs-meta">html</span><span class="dt"><span class="hljs-meta">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span><span>{</span><span>%</span> block title <span>%</span><span>}</span><span>{</span><span>%</span> endblock <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">link</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> <span>static</span> "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">css</span>/<span class="hljs-attr">blog.css</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">rel</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"stylesheet"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"content"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span>{</span><span>%</span> endblock <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"sidebar"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>My blog<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>This is my blog.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre></div>
<p><code></span>{</span><span>%</span> load staticfiles <span>%</span><span>}</span></code>告诉Django去加载<em>django.contrib.staticfiles</em>应用提供的<em>staticfiles</em> 模板标签（temaplate tags）。通过加载它，你可以在这个模板（template）中使用<code><span>{</span><span>%</span> <span>static</span> <span>%</span><span>}</span></code>模板过滤器（template filter）。通过使用这个模板过滤器（template filter），你可以包含一些静态文件比如说<em>blog.css</em>文件，你可以在本书的范例代码例子中找到该文件，在blog应用的<code>static/</code>目录中<strong>（译者注：给大家个地址去拷贝 <a href="https://github.com/levelksk/django-by-example-book" class="uri">https://github.com/levelksk/django-by-example-book</a> ）</strong>拷贝这个目录到你的项目下的相同路径来使用这些静态文件。</p>
<p>你可以看到有两个<code><span>{</span><span>%</span> block <span>%</span><span>}</span></code>标签（tags）。这些是用来告诉Django我们想在这个区域中定义一个区块（block）。继承这个模板（template）的其他模板（templates）可以使用自定义的内容来填充区块（block）。我们定义了一个区块（block）叫做<em>title</em>，另一个区块（block）叫做<em>content</em>。</p>
<p>让我们编辑<em>post/list.html</em>文件使它如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "blog/base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>My Blog<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>My Blog<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span>{</span><span>%</span> for post in posts <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"{{ post.get_absolute_url }}"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        {{ post.title }}
      <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"date"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      Published {{ post.publish }} by {{ post.author }}
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
    {{ post.body|truncatewords:30|linebreaks }}
  <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>通过<code><span>{</span><span>%</span> extends <span>%</span><span>}</span></code>模板标签（template tag），我们告诉Django需要继承<em>blog/base.html</em> 模板（template）。然后我们在<em>title</em>和<em>content</em>区块（blocks）中填充内容。我们通过循环迭代帖子来展示它们的标题，日期，作者和内容，在标题中还集成了帖子的标准URL链接。在帖子的内容中，我们应用了两个模板过滤器（template filters）： <em>truncatewords</em>用来缩短内容限制在一定的字数内，<em>linebreaks</em>用来转换内容中的换行符为HTML的换行符。只要你喜欢你可以连接许多模板标签（tempalte filters），每一个都会应用到上个输出生成的结果上。</p>
<p>打开终端执行命令<code>python manage.py runserver</code>来启动开发服务器。在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 你会看到运行结果。注意，你需要添加一些发布状态的帖子才能在这儿看到它们。你会看到如下图所示：<br>
<iframe id="iframe_0.5919300372059573" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-a8f162f31d72f0dc.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.5919300372059573',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 626px; height: 297px;" frameborder="0" scrolling="no"></iframe></p>
<p>这之后，让我们来编辑<em>post/detail.html</em>文件使它如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "blog/base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>{{ post.title }}<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>{{ post.title }}<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"date"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    Published {{ post.publish }} by {{ post.author }}
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  {{ post.body|linebreaks }}
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>现在，你可以在浏览器中点击其中一篇帖子的标题来看帖子的详细视图（view）。你会看到类似以下页面：<br>
<iframe id="iframe_0.20138048564446032" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-6c9f869e3aaad43d.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.20138048564446032',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 626px; height: 142px;" frameborder="0" scrolling="no"></iframe></p>

### 添加页码
<p>当你开始给你的blog添加内容，你很快会意识到你需要将帖子分页显示。Django有一个内置的<em>Paginator</em>类允许你方便的管理分页。</p>
<p>编辑blog应用下的<em>views.py</em>文件导入Django的页码类修改<em>post_list</em>如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.paginator <span class="im"><span class="hljs-keyword">import</span></span> Paginator, EmptyPage, PageNotAnInteger

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_list</span><span class="hljs-params">(request)</span>:</span>
    object_list <span class="op">=</span> Post.published.<span class="bu">all</span>()
    paginator <span class="op">=</span> Paginator(object_list, <span class="dv"><span class="hljs-number">3</span></span>) <span class="co"><span class="hljs-comment"># 3 posts in each page</span></span>
    page <span class="op">=</span> request.GET.get(<span class="st"><span class="hljs-string">'page'</span></span>)
    <span class="cf"><span class="hljs-keyword">try</span></span>:
        posts <span class="op">=</span> paginator.page(page)
    <span class="cf"><span class="hljs-keyword">except</span></span> PageNotAnInteger:
        <span class="co"><span class="hljs-comment"># If page is not an integer deliver the first page</span></span>
        posts <span class="op">=</span> paginator.page(<span class="dv"><span class="hljs-number">1</span></span>)
    <span class="cf"><span class="hljs-keyword">except</span></span> EmptyPage:
        <span class="co"><span class="hljs-comment"># If page is out of range deliver last page of results</span></span>
        posts <span class="op">=</span> paginator.page(paginator.num_pages)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                  <span class="st"><span class="hljs-string">'blog/post/list.html'</span></span>,
                  {<span class="st"><span class="hljs-string">'page'</span></span>: page, 
                   <span class="co"><span class="hljs-string">'posts'</span></span>: posts})</code></pre></div>
<p><em>Paginator</em>是如何工作的：</p>
<ul>
<li>我们使用希望在每页中显示的对象的数量来实例化<em>Paginator</em>类。</li>
<li>我们获取到<em>page</em> GET参数来指明页数<br>
</li>
<li>我们通过调用<em>Paginator</em>的 <em>page()</em>方法在期望的页面中获得了对象。</li>
<li>如果<em>page</em>参数不是一个整数，我们就返回第一页的结果。如果这个参数数字超出了最大的页数，我们就展示最后一页的结果。</li>
<li>我们传递页数并且获取对象给这个模板（template）。</li>
</ul>
<p>现在，我们必须创建一个模板（template）来展示分页处理，它可以被任意的模板（template）包含来使用分页。在blog应用的<em>templates</em>文件夹下创建一个新文件命名为<em>pagination.html</em>。在该文件中添加如下HTML代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"pagination"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"step-links"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>%</span> if page.has_previous <span>%</span><span>}</span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"?page={{ page.previous_page_number }}"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Previous<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span>{</span><span>%</span> endif <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"current"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      Page {{ page.number }} of {{ page.paginator.num_pages }}.
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
      <span>{</span><span>%</span> if page.has_next <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"?page={{ page.next_page_number }}"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Next<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
      <span>{</span><span>%</span> endif <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>    </code></pre></div>
<p>为了渲染上一页与下一页的链接并且展示当前页面和所有页面的结果，这个分页模板（template）期望一个<em>Page</em>对象。让我们回到<em>blog/post/list.html</em>模板（tempalte）中将<em>pagination.html</em>模板（template）包含在<code><span>{</span><span>%</span> content <span>%</span><span>}</span></code>区块（block）中，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> block content <span>%</span><span>}</span>
  ...
  <span>{</span><span>%</span> include "pagination.html" with page=posts <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>我们传递给模板（template）的<em>Page</em>对象叫做<em>posts</em>，我们将分页模板（tempalte）包含在帖子列模板（template）中指定参数来对它进行正确的渲染。这种方法你可以反复使用，用你的分页模板（template）对不同的模型（models）视图（views）进行分页处理。</p>
<p>现在，在你的浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a>。 你会看到帖子列的底部已经有分页处理：<br>
<iframe id="iframe_0.6678382213709215" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-6fbd1a4b2e409533.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6152893%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.6678382213709215',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 622px; height: 332px;" frameborder="0" scrolling="no"></iframe></p>

### 使用基于类的视图（views）
<p>因为一个视图（view）的调用就是得到一个web请求并且返回一个web响应，你可以将你的视图（views）定义成类方法。Django为此定义了基础的视图（view）类。它们都从<em>View</em>类继承而来，<em>View</em>类可以操控HTTP方法调度以及其他的功能。这是一个可替代的方法来创建你的视图（views）。</p>
<p>我们准备通过使用Django提供的通用<em>ListView</em>使我们的<em>post_list</em>视图（view）转变为一个基于类的视图。这个基础视图（view）允许你对任意的对象进行排列。</p>
<p>编辑你的blog应用下的<em>views.py</em>文件，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic <span class="im"><span class="hljs-keyword">import</span></span> ListView
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">PostListView</span><span class="hljs-params">(ListView)</span>:</span>
    queryset <span class="op">=</span> Post.published.<span class="bu">all</span>()
    context_object_name <span class="op">=</span> <span class="st"><span class="hljs-string">'posts'</span></span>
    paginate_by <span class="op">=</span> <span class="dv"><span class="hljs-number">3</span></span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'blog/post/list.html'</span></span></code></pre></div>
<p>这个基于类的的视图（view）类似与之前的<em>post_list</em>视图（view）。在这儿，我们告诉<em>ListView</em>做了以下操作：</p>
<ul>
<li>使用一个特定的查询集（QuerySet）代替取回所有的对象。代替定义一个<em>queryset</em>属性，我们可以指定<code>model = Post</code>然后Django将会构建<em>Post.objects.all()</em> 查询集（QuerySet）给我们。</li>
<li>使用环境变量<em>posts</em>给查询结果。如果我们不指定任意的<em>context_object_name</em>默认的变量将会是<em>object_list</em>。</li>
<li>对结果进行分页处理每页只显示3个对象。</li>
<li>使用定制的模板（template）来渲染页面。如果我们不设置默认的模板（template），<em>ListView</em>将会使用<code>blog/post_list.html</code>。</li>
</ul>
<p>现在，打开你的blog应用下的<em>urls.py</em>文件，注释到之前的<em>post_list</em>URL模式，在之后添加一个新的URL模式来使用<em>PostlistView</em>类，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># post views</span></span>
    <span class="co"><span class="hljs-comment"># url(r'^$', views.post_list, name='post_list'),</span></span>
    url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, views.PostListView.as_view(),name<span class="op">=</span><span class="st"><span class="hljs-string">'post_list'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;year&gt;\d</span></span><span class="sc"><span class="hljs-string">{4}</span></span><span class="vs"><span class="hljs-string">)/(?P&lt;month&gt;\d</span></span><span class="sc"><span class="hljs-string">{2}</span></span><span class="vs"><span class="hljs-string">)/(?P&lt;day&gt;\d</span></span><span class="sc"><span class="hljs-string">{2}</span></span><span class="vs"><span class="hljs-string">)/'</span></span><span class="op">\</span>
        <span class="co"><span class="hljs-string">r'(?P&lt;post&gt;[-\w]+)/$'</span></span>,
        views.post_detail,
        name<span class="op">=</span><span class="st"><span class="hljs-string">'post_detail'</span></span>),
]</code></pre></div>
<p>为了保持分页处理能工作，我们必须将正确的页面对象传递给模板（tempalte）。Django的<em>ListView</em>通过叫做<em>page_obj</em>的变量来传递被选择的页面，所以你必须编辑你的<em>post_list_html</em>模板（template）去包含使用了正确的变量的分页处理，如下所示：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">include</span></span> "pagination.html" with page=page_obj <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 然后检查每一样功能是否都和之前的<em>post_list</em>视图（view）一样工作。这是一个简单的，通过使用Django提供的通用类的基于类视图（view）的例子。你将在<em>第十章，创建一个在线学习平台</em>以及相关的章节中学到更多的基于类的视图（views）。</p>

### 总结
<p>在本章中，你通过创建一个基础的blog应用学习了Django web框架的基础。你为你的项目设计了数据模型（models）并且进行了数据库迁移（migration）。你为你的blog创建了视图（views），模板（templates）以及URLs，还包括对象分页。</p>
<p>在下一章中，你会学习到如何增强你的blog应用，例如评论系统，标签（tag）功能，并且允许你的用户通过邮件来分享帖子。</p>
</div>