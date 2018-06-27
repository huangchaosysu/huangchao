---
layout: single
permalink: /django/example10/
title: "Django By Example 第十章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第十章">第十章</h1>
<h2 id="创建一个在线学习平台e-learning-platform">创建一个在线学习平台（e-Learning Platform）</h2>
<p>在上一章，你添加国际化到你的在线商店项目中。你还构建了一个优惠券系统和一个商品推荐引擎。在这章中，你会创建一个新项目。你将构建一个在线学习平台创建一个定制内容管理系统。</p>
<p>在这章中，你会学习以下操作：</p>
<ul>
<li>创建fixtures给你的模型</li>
<li>使用模型继承</li>
<li>创建定制模型字段</li>
<li>使用基于类的视图和mixins</li>
<li>构建formsets</li>
<li>管理组合权限</li>
<li>创建一个内容管理系统</li>
</ul>
<h2 id="创建一个在线学习平台">创建一个在线学习平台</h2>
<p>我们最实际的项目将会是一个在线学习平台。在本章中，我们将要构建一个灵活的<strong>内容管理系统（CMS）</strong>用来允许教师来创建课程和管理它们的内容。</p>
<p>首先，创建一个虚拟环境给你的新项目并且激活它通过以下命令：</p>
<pre class="shell"><code class="hljs perl"><span class="hljs-keyword">mkdir</span> env
virtualenv env/educa
source env/educa/bin/activate</code></pre>
<p>安装Django到你的虚拟环境中通过以下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Django==<span class="hljs-number">1.8</span>.<span class="hljs-number">6</span></code></pre>
<p>我们将要管理图片上传在我们的项目中，所以我们还需要安装Pillow通过以下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Pillow==<span class="hljs-number">2.9</span>.<span class="hljs-number">0</span></code></pre>
<p>创建一个新项目使用以下命令：</p>
<pre><code class="hljs">django-admin startproject educa</code></pre>
<p>进入这个新的<em>educa</em>目录并且创建一个新应用使用以下命令：</p>
<pre class="shell"><code class="hljs dos"><span class="hljs-built_in">cd</span> educa
django-admin startapp courese</code></pre>
<p>编辑<em>educa</em>项目的<em>settings.py</em>文件并且添加<em>courses</em>到<code>INSTALLED_APPS</code>设置中如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> ( 
    <span class="st"><span class="hljs-string">'courses'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.admin'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.auth'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.contenttypes'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.sessions'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.messages'</span></span>,
    <span class="co"><span class="hljs-string">'django.contrib.staticfiles'</span></span>,
)</code></pre></div>
<p><em>courses</em>应用现在已经在这个项目中激活。让我们定义模型给课程以及课程内容。</p>
<h2 id="构建课程模型">构建课程模型</h2>
<p>我们的在线学习平台将会提供课程在许多主题中。每一个课程都将会划分为一个可配置的模块编号，并且每个模块将会包含一个可配置的内容编号。将会有许多类型的内容：文本，文件，图片，或者视频。下面的例子展示了我们的课程目录的数据结构：</p>
<pre><code class="hljs vbnet">Subject <span class="hljs-number">1</span>
    Course <span class="hljs-number">1</span>
        <span class="hljs-keyword">Module</span> <span class="hljs-number">1</span>
            Content <span class="hljs-number">1</span> (images)
            Content <span class="hljs-number">3</span> (<span class="hljs-keyword">text</span>)
        <span class="hljs-keyword">Module</span> <span class="hljs-number">2</span>
            Content <span class="hljs-number">4</span> (<span class="hljs-keyword">text</span>)
            Content <span class="hljs-number">5</span> (file)
            Content <span class="hljs-number">6</span> (video)
            ...</code></pre>
<p>让我们来构建课程模型。编辑<em>courses</em>应用的<em>models.py</em>文件并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.auth.models <span class="im"><span class="hljs-keyword">import</span></span> User
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Subject</span><span class="hljs-params">(models.Model)</span>:</span>
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'title'</span></span>,)
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.title
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Course</span><span class="hljs-params">(models.Model)</span>:</span>
    owner <span class="op">=</span> models.ForeignKey(User,
                                 related_name<span class="op">=</span><span class="st"><span class="hljs-string">'courses_created'</span></span>)
    subject <span class="op">=</span> models.ForeignKey(Subject,
                                   related_name<span class="op">=</span><span class="st"><span class="hljs-string">'courses'</span></span>)
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    overview <span class="op">=</span> models.TextField()
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'-created'</span></span>,)
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.title
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Module</span><span class="hljs-params">(models.Model)</span>:</span>
    course <span class="op">=</span> models.ForeignKey(Course, related_name<span class="op">=</span><span class="st"><span class="hljs-string">'modules'</span></span>)
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>)
    description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.title</code></pre></div>
<p>这些是最初的<em>Subject,Course,</em>以及<em>Module</em>模型。<em>Course</em>模型字段如下所示：</p>
<ul>
<li>owner：创建这个课程的教师。</li>
<li>subject：这个课程属于的主题。这是一个<em>ForeingnKey</em>字段指向<em>Subject</em>模型。</li>
<li>title：课程标题。</li>
<li>slug：课程的slug。之后它将会被用在URLs中。</li>
<li>overview：这是一个<em>TextFied</em>列用来包含一个关于课程的概述。</li>
<li>created：课程被创建的日期和时间。它将会被Django自动设置当创建一个新的对象，因为<code>auto_now_add=True</code>。</li>
</ul>
<p>每一个课程都被划分为多个模块。因此，<em>Module</em>模型包含一个<em>ForeignKey</em>字段用来指向<em>Course</em>模型。</p>
<p>打开shell并且运行一下命令来给这个应用创建最初的迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manange</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你将会看到以下输出：</p>
<pre><code class="hljs sql">Migrations for 'courses':
     0001_initial.py:
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> Course
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-keyword">Module</span>
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> Subject
       - <span class="hljs-keyword">Add</span> <span class="hljs-keyword">field</span> subject <span class="hljs-keyword">to</span> course</code></pre>
<p>之后，运行一下命令来应用所有的迁移到数据库中：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你将会看到一个输出包含所有应用的迁移，包括Django的那些。这个输出将会包含以下行：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">courses</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>这告诉我们那个我们的<em>courses</em>引用模型已经同步到了数据库中。</p>
<h2 id="注册模型到管理平台中">注册模型到管理平台中</h2>
<p>我们将要添加课程模型到管理平台中。编辑<em>courses</em>应用目录下的<em>admin.py</em>文件并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Subject, Course, Module

<span class="at"><span class="hljs-meta">@admin.register</span></span><span class="hljs-meta">(Subject)</span>
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">SubjectAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>]
    prepopulated_fields <span class="op">=</span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'title'</span></span>,)}
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ModuleInline</span><span class="hljs-params">(admin.StackedInline)</span>:</span>
    model <span class="op">=</span> Module
<span class="at"><span class="hljs-meta">@admin.register</span></span><span class="hljs-meta">(Course)</span>

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>]
    list_filter <span class="op">=</span> [<span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'subject'</span></span>]
    search_fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'overview'</span></span>]
    prepopulated_fields <span class="op">=</span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'title'</span></span>,)}
    inlines <span class="op">=</span> [ModuleInline]</code></pre></div>
<p>课程应用的模型现在已经在管理平台中注册。我们使用<code>@admin.register()</code>装饰器替代了<code>admin.site.register()</code>方法。它们都提供了相同的功能。</p>
<h2 id="提供最初数据给模型">提供最初数据给模型</h2>
<p>有时候你可能想要预装你的数据库通过使用硬编码数据。这是很有用的，当自动包含最初数据在项目设置中用来替代手工去添加数据。Django自带一个简单的方法来加载以及转储数据库中的数据到字段中，这被称为fixtures。</p>
<p>Django支持fixtures在JSON,XML,或者YAML格式中。我们将要创建一个fixture用来包含一些最初的<em>Subject</em>对象给我们的项目。</p>
<p>首先，创建一个超级用户使用如下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">createsuperuser</span></code></pre>
<p>之后，运行开发服务器使用以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">runserver</span></code></pre>
<p>现在，打开 <a href="http://127.0.0.1:8000/admin/courses/subject/" class="uri">http://127.0.0.1:8000/admin/courses/subject/</a> 在你的浏览器中。创建一些主题通过使用管理平台。列页面看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-1.png" alt="django-10-1"></p>
<p>运行一下命令在shell中：</p>
<pre><code class="hljs lua">python manage.py dumpdata courese <span class="hljs-comment">--indent=2</span></code></pre>
<p>你会看到类似以下的输出：</p>
<div class="sourceCode"><pre class="sourceCode json"><code class="sourceCode json hljs"><span class="ot">[</span>
<span class="fu">{</span>
  <span class="dt">"<span class="hljs-attr">fileld:</span>"</span><span class="fu">:</span> <span class="fu">{</span>
    <span class="dt">"<span class="hljs-attr">title</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"Programming"</span></span><span class="fu">,</span>
    <span class="dt">"<span class="hljs-attr">slug</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"programming"</span></span>
  <span class="fu">},</span>
  <span class="dt">"<span class="hljs-attr">model</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"courses.subject"</span></span><span class="fu">,</span>
  <span class="dt">"<span class="hljs-attr">pk</span>"</span><span class="fu">:</span> <span class="dv"><span class="hljs-number">1</span></span>
<span class="fu">}</span><span class="ot">,</span>
<span class="fu">{</span>
<span class="dt">"<span class="hljs-attr">fields</span>"</span><span class="fu">:</span> <span class="fu">{</span>
    <span class="dt">"<span class="hljs-attr">title</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"Mathematics"</span></span><span class="fu">,</span>
    <span class="dt">"<span class="hljs-attr">slug</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"mathematics"</span></span>
  <span class="fu">},</span>
  <span class="dt">"<span class="hljs-attr">model</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"courses.subject"</span></span><span class="fu">,</span>
  <span class="dt">"<span class="hljs-attr">pk</span>"</span><span class="fu">:</span> <span class="dv"><span class="hljs-number">2</span></span>
<span class="fu">}</span><span class="ot">,</span> 
<span class="fu">{</span>
<span class="dt">"<span class="hljs-attr">fields</span>"</span><span class="fu">:</span> <span class="fu">{</span>
    <span class="dt">"<span class="hljs-attr">title</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"Physics"</span></span><span class="fu">,</span>
    <span class="dt">"<span class="hljs-attr">slug</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"physics"</span></span>
  <span class="fu">},</span>
  <span class="dt">"<span class="hljs-attr">model</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"courses.subject"</span></span><span class="fu">,</span>
  <span class="dt">"<span class="hljs-attr">pk</span>"</span><span class="fu">:</span> <span class="dv"><span class="hljs-number">3</span></span>
<span class="fu">}</span><span class="ot">,</span> <span class="fu">{</span>
  <span class="dt">"<span class="hljs-attr">fields</span>"</span><span class="fu">:</span> <span class="fu">{</span>
    <span class="dt">"<span class="hljs-attr">title</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"Music"</span></span><span class="fu">,</span>
    <span class="dt">"<span class="hljs-attr">slug</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"music"</span></span>
  <span class="fu">},</span>
  <span class="dt">"<span class="hljs-attr">model</span>"</span><span class="fu">:</span> <span class="st"><span class="hljs-string">"courses.subject"</span></span><span class="fu">,</span>
  <span class="dt">"<span class="hljs-attr">pk</span>"</span><span class="fu">:</span> <span class="dv"><span class="hljs-number">4</span></span>
<span class="fu">}</span> 
<span class="ot">]</span></code></pre></div>
<p><em>dumpdata</em>命令从数据库中转储数据到标准输出中，默认序列化为JSON。这串数据结构包含的信息关于这个模型以及它的字段将会被Django用来加载它到数据库中。</p>
<p>你可以提供应用名给这命令或者指定模型给输出数据使用<code>app.Model</code>格式。你还可以指定格式通过使用<code>--format</code>标记。默认的，<em>dumpdata</em>输出序列化数据给标准输出。当然，你可以表明一个输出文件通过使用<code>--output</code>标记。<code>--indent</code>标记允许你指定缩进。更多信息关于<em>udmpdata</em>参数，运行<code>python manage.py dumpdata --help</code>。</p>
<p>保存这个转储为一个fixtures文件到<em>orders</em>应用的<em>fixtures/</em>目录中，通过使用如下命令：</p>
<pre class="shell"><code class="hljs lua">mkdir courses/fixtures
python manage.py dumpdata courses <span class="hljs-comment">--indent=2 --output=courses/fixtures/</span>
subjects.json</code></pre>
<p>使用管理平台去移除你之前创建的主题。之后加载fixture到数据库中通过使用以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">loaddata</span> <span class="hljs-selector-tag">subjects</span><span class="hljs-selector-class">.json</span></code></pre>
<p>所有包含在fixture中的<code>subject</code>对象都会加载到数据库中。</p>
<p>默认的，Django会寻找每一个应用的<em>fixtures/</em>目录下的文件，但是你可以指定fixture文件的完整路径给<em>loaddata</em>命令。你还可以使用<code>FIXTURE_DIRS</code>设置来告诉Django去额外的目录寻找fixtures。</p>
<blockquote>
<p>Fixtures并不只对初始化数据有用，还可以提供简单的数据给你的应用或者数据请求给你的测试用例。</p>
</blockquote>
<p>你可以找到更多关于如何使用fixtures在测试中，通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/testing/tools/#topics-testing-fixtures" class="uri">https://docs.djangoproject.com/en/1.8/topics/testing/tools/#topics-testing-fixtures</a> 。</p>
<p>如果你想要加载fixturres在模型迁移中，去看下Django的文档关于数据迁移。请记住，我们创建了一个定制迁移在<strong>第九章，扩展你的商店</strong>来迁移存在的数据在修改给翻译的模型之后。你可以找到迁移数据的文档，通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/migrations/#data-migrations" class="uri">https://docs.djangoproject.com/en/1.8/topics/migrations/#data-migrations</a> 。</p>
<h2 id="给不同的内容创建模型">给不同的内容创建模型</h2>
<p>我们打算添加各种不同的内容类型给课程模块，例如文本，图片，文件以及视屏。我们需要一个通用的数据模型可以允许我们去存储不同的内容。在<strong>第六章，跟踪用户行为</strong>中，你已经学习过有关使用通用关系方便的创建外键能够指向任何模型的对象。我们将要创建一个<em>content</em>模型相当于模块内容以及定义一个通用关系来连接任意种类的内容。</p>
<p>编辑<em>courses</em>应用下的<em>models.py</em>文件并且添加如下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib.contenttypes.models <span class="im"><span class="hljs-keyword">import</span></span> ContentType
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.contenttypes.fields <span class="im"><span class="hljs-keyword">import</span></span> GenericForeignKey</code></pre></div>
<p>之后添加如下代码到文件后面：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Content</span><span class="hljs-params">(models.Model)</span>:</span>
    module <span class="op">=</span> models.ForeignKey(Module, related_name<span class="op">=</span><span class="st"><span class="hljs-string">'contents'</span></span>)
    content_type <span class="op">=</span> models.ForeignKey(ContentType)
    object_id <span class="op">=</span> models.PositiveIntegerField()
    item <span class="op">=</span> GenericForeignKey(<span class="st"><span class="hljs-string">'content_type'</span></span>, <span class="st"><span class="hljs-string">'object_id'</span></span>)</code></pre></div>
<p>这就是一个<em>Content</em>模型。一个模块包含多种内容，所有我们定义了一个<code>ForeignKey</code>字段给<em>module</em>模型。我们还设置了一个通用关系来连接对象从不同的模型中相当于不同的内容类型。请记住，我们需要三种不同的字段来设置一个通用关系。在我们的<em>Content</em>模型中，它们是：</p>
<ul>
<li>content_type：一个<em>ForeignKey</em>字段指向<em>ContentType</em>模型</li>
<li>object_id：这是<em>PositiveIntegerField</em>用来存储有关联对象的关键字</li>
<li>item：一个<em>GenericForeignKey</em>字段指向被关联的对象通过结合前两个字段</li>
</ul>
<p>只有<code>content_type</code>和<code>object_id</code>字段有一个对应列在这个模型的数据库表中。<code>item</code>字段允许你去检索或者直接设置关联对象，并且它的功能是简历在其他两个字段之上。</p>
<p>我们将要使用一个不同的模型给每一种内容。我们的内容模型将会有很多共有的字段，但是它们将会有不同之处在它们存储的真实内容中。</p>
<h2 id="使用模型继承">使用模型继承</h2>
<p>Django支持模型继承。类似与Python中的标准类继承。Django提供以下三种方式来使用模型继承：</p>
<ul>
<li>Abstract models：非常有用当你想要安置一些公用信息到多个模型中。没有数据库表会被创建给抽象模型。</li>
<li>Multi-table model inheritance：可适当的利用当每个模型经过慎重考虑都是一个它自身的完整的模型。每个模型都会创建一个数据库表。</li>
<li>Proxy models：非常有用当你需要改变一个模型行为，比如说，包含额外的方法，修改默认管理器，或者使用不同的元选项。没有数据表会被创建给代理模型。</li>
</ul>
<p>让我们对以上三者都来一次近距离的实践。</p>
<h2 id="抽象模型">抽象模型</h2>
<p>一个抽象模型就是一个基础类，你定义在其中的字段就是你想要包含到所有子模型中的字段。Djnago不会创建任何数据库表给抽象模型。每个子模型都会创建一张数据库表，包含有继承自抽象类的字段以及在子模型中自己定义的字段。</p>
<p>为了抽象一个模型，你需要在<code>Meta</code>类中包含<code>abstract=True</code>。Django将会认出这个模型是一个抽象模型并且不会给它创建数据库表。为了创建子模型，你只需要基于这个抽象模型。下面就是一个例子关于一个抽象的<em>Content</em>模型和一个子的<code>Text</code>模型：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">BaseContent</span><span class="hljs-params">(models.Model)</span>:</span>
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">100</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        abstract <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Text</span><span class="hljs-params">(BaseContent)</span>:</span>
    body <span class="op">=</span> models.TextField()</code></pre></div>
<p>在这个例子中，Django将只会给<code>Text</code>模型创建表，包含<code>title</code>,<code>created</code>以及<code>body</code>字段。</p>
<h2 id="多表模型继承">多表模型继承</h2>
<p>在多表模型继承中，每个模型对应一个张数据库表。Django创建一个<em>OneToOneField</em>字段给子模型创建关系指向它的父模型。</p>
<p>为了使用多表继承，你必须基于一个存在的模型。djnago将会创建一张数据表给每个源头模型以及子模型。以下例子展示多表继承：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">BaseContent</span><span class="hljs-params">(models.Model)</span>:</span>
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">100</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Text</span><span class="hljs-params">(BaseContent)</span>:</span>
    body <span class="op">=</span> models.TextField()</code></pre></div>
<p>Django将会包含一个自动生成的<em>OneToOneField</em>字段在<code>Text</code>模型中并且给每个模型创建一张数据库表。</p>
<h2 id="代理模型">代理模型</h2>
<p>代理模型被用于改变一个模型的行为，举个例子，包含额外的方法或者不同的元选项。每个模型对源头模型的数据库表起作用。为了创建一个代理模型，在这个模型的<code>Meta</code>类中添加<code>proxy=True</code>。</p>
<p>以下例子说明如何创建一个代理模型：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> django.utils <span class="im"><span class="hljs-keyword">import</span></span> timezone
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">BaseContent</span><span class="hljs-params">(models.Model)</span>:</span>
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">100</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderedContent</span><span class="hljs-params">(BaseContent)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        proxy <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
        ordering <span class="op">=</span> [<span class="st"><span class="hljs-string">'created'</span></span>]
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">created_delta</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> timezone.now() <span class="op">-</span> <span class="va">self</span>.created</code></pre></div>
<p>这里，我们定义了一个<em>OrderedContent</em>模型这是一个代理模型给<em>Content</em>模型使用。这个模型提供了一个默认的排序给查询集并且一个额外的<code>create_delta()</code>方法。这两个模型，<code>Content</code>和<code>OrderedContent</code>，对同一个数据库表起作用，并且通过任一一个模型都能通过ORM渠道连接到对象。</p>
<h2 id="创建内容模型">创建内容模型</h2>
<p>我们的<em>courses</em>应用的<em>Content</em>模型包含一个通用关系来连接不同类型的内容给该应用。我们将要创建一个不同的模型给每种类型的内容。所有内容模型将会有一些公用的字段，以及额外的字段去存储定制数据。我们将会创建一个抽象模型来提供公用字段给所有内容模型。</p>
<p>编辑<em>courses</em>应用的<em>models.py</em>文件，并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ItemBase</span><span class="hljs-params">(models.Model)</span>:</span>
    owner <span class="op">=</span> models.ForeignKey(User,
                                related_name<span class="op">=</span><span class="st"><span class="hljs-string">'</span></span><span class="sc"><span class="hljs-string">%(class)s</span></span><span class="st"><span class="hljs-string">_related'</span></span>)
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">250</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        abstract <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.title
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Text</span><span class="hljs-params">(ItemBase)</span>:</span>
    content <span class="op">=</span> models.TextField()
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">File</span><span class="hljs-params">(ItemBase)</span>:</span>
    <span class="bu">file</span> <span class="op">=</span> models.FileField(upload_to<span class="op">=</span><span class="st"><span class="hljs-string">'files'</span></span>)
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Image</span><span class="hljs-params">(ItemBase)</span>:</span>
    <span class="bu">file</span> <span class="op">=</span> models.FileField(upload_to<span class="op">=</span><span class="st"><span class="hljs-string">'images'</span></span>)
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Video</span><span class="hljs-params">(ItemBase)</span>:</span>
    url <span class="op">=</span> models.URLField() </code></pre></div>
<p>在这串代码中，我们定义了一个抽象模型命名为<code>ItemBase</code>。除此以外，我们在<code>Meta</code>类中设置<code>abstract=True</code>。在这个模型中，我们定义<code>owner,title,created</code>，以及<code>updated</code>字段。这些公用字段将会被所有的内容类型使用到。<code>owner</code>字段允许我们去存储哪个用户创建了这个内容。因为和这个字段是被定义在一个抽象类中，我们需要不同的<code>related_name</code>给每个子模型。Django允许我们去指定一个占位符给<code>model</code>类名在<code>related_name</code>属性类似<code>%(class)s</code>。为了做到这些，<code>related_name</code>对每个子模型都会自动生成。因为我们使用<code>%(class)s_related</code>作为<code>related_name</code>，给子模型的相对关系将各自是<code>text_related,file_related,image_related,</code>以及<code>vide0_related</code>。</p>
<p>我们已经定义了四种不同的内容模型，它们都继承自<code>ItemBase</code>抽象模型，它们是：</p>
<ul>
<li>Text：用来存储文本内容。</li>
<li>File：用来存储文件，例如PDF。</li>
<li>Image：用来存储图片文件。</li>
<li>Video：用来存储视频。我们使用一个<code>URLField</code>字段来提供一个视频URL为了嵌入该视频。</li>
</ul>
<p>每个子模型包含定义在<code>ItemBase</code>类中的字段以及它自己的字段。<code>text_related,file_related,image_related,</code>以及<code>vide0_related</code>都会各自创建一张数据库表。不会有数据库表连接到<code>ItemBase</code>模型，因为它是一个抽象模型。</p>
<p>编辑你之前创建的<em>Content</em>模型，修改它的<code>content_type</code>字段如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">content_type <span class="op">=</span> models.ForeignKey(ContentType,
                      limit_choices_to<span class="op">=</span>{<span class="st"><span class="hljs-string">'model__in'</span></span>:(<span class="st"><span class="hljs-string">'text'</span></span>,
                                           <span class="st"><span class="hljs-string">'video'</span></span>,
                                           <span class="co"><span class="hljs-string">'image'</span></span>,
                                           <span class="co"><span class="hljs-string">'file'</span></span>)})</code></pre></div>
<p>我们添加一个<code>limit_choices_to</code>参数来限制<code>ContentType</code>对象可以被通用关系使用。我们使用<code>model__in</code>字段查找过滤这个查询给<code>ContentType</code>对象通过一个<code>model</code>属性就像'text','video','image',或者'file'。</p>
<p>让我们创建一个迁移来包含这些新的模型我们之前添加的。运行以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你会看到以下输出：</p>
<pre class="shell"><code class="hljs sql">Migrations for 'courses':
     0002_content_file_image_text_video.py:
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-keyword">Content</span>
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-keyword">File</span>
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> Image
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-built_in">Text</span>
       - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> Video</code></pre>
<p>之后，运行一下命令来应用新的迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会在输出结果看到以下内容：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">Running</span> <span class="hljs-selector-tag">migrations</span>:
     <span class="hljs-selector-tag">Rendering</span> <span class="hljs-selector-tag">model</span> <span class="hljs-selector-tag">states</span>... <span class="hljs-selector-tag">DONE</span>
     <span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">courses</span><span class="hljs-selector-class">.0002_content_file_image_text_video</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>我们之前创建的模型对于添加不同的内容给课程模块是很合适的。但是，仍然有一些东西是被遗漏的在我们的模型中。课程模块和内容应当跟随一个特定的顺序。我们需要一个字段，这个字段允许我们简单的排序它们。</p>
<h2 id="创建定制模型字段">创建定制模型字段</h2>
<p>Django自带一个完整的模型字段采集能让你用来构建你的模型。当然，你也可以创建你自己的模型字段来存储定制数据或者改变现有字段的行为。</p>
<p>我们需要一个字段允许我们给对象们定义次序。如果你想通过Djanog提供的一个字段来方便的处理这点，你大概会想到添加一个<code>PositiveIntegerField</code>给你的模型。这是一个好的起点。我们可以创建一个定制字段，该字段继承自<code>PositiveIntegerField</code>并且提供额外的行为。</p>
<p>有两种相关的功能我们将构建到我们的次序字段中：</p>
<ul>
<li>自动分配一个次序值当没有指定的次序被提供的时候。当没有次数被提供的时候存储一个对象，我们的字段将自动分配下一个次序，该次序基于最后存在次序的对象。如果有两个对象，分别是次序1和次序2，当保存第三个对象的时候，我们会自动分配次序3给第三个对象如果没有给予指定的次序。</li>
<li>次序对象关于其他的字段。课程模块将按照它们所属的课程和相关模块的内容进行排序。</li>
</ul>
<p>创建一个新的<em>fields.py</em>文件到<em>courses</em>应用目录下，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> django.core.exceptions <span class="im"><span class="hljs-keyword">import</span></span> ObjectDoesNotExist

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderField</span><span class="hljs-params">(models.PositiveIntegerField)</span>:</span>

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__init__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, for_fields</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        <span class="va">self</span>.for_fields <span class="op">=</span> for_fields
        <span class="bu">super</span>(OrderField, <span class="va">self</span>).<span class="fu">__init__</span>(<span class="op">*</span>args, <span class="op">**</span>kwargs)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">pre_save</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, model_instance, add)</span>:</span>
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="bu">getattr</span>(model_instance, <span class="va">self</span>.attname) <span class="op"><span class="hljs-keyword">is</span></span> <span class="va"><span class="hljs-keyword">None</span></span>:
            <span class="co"><span class="hljs-comment"># no current value</span></span>
            <span class="cf"><span class="hljs-keyword">try</span></span>:
                qs <span class="op">=</span> <span class="va">self</span>.model.objects.<span class="bu">all</span>()
                <span class="cf"><span class="hljs-keyword">if</span></span> <span class="va">self</span>.for_fields:
                    <span class="co"><span class="hljs-comment"># filter by objects with the same field values</span></span>
                    <span class="co"><span class="hljs-comment"># for the fields in "for_fields"</span></span>
                    query <span class="op">=</span> {field: <span class="bu">getattr</span>(model_instance, field) <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.for_fields}
                    qs <span class="op">=</span> qs.<span class="bu">filter</span>(<span class="op">**</span>query)
                <span class="co"><span class="hljs-comment"># get the order of the last item</span></span>
                last_item <span class="op">=</span> qs.latest(<span class="va">self</span>.attname)
                value <span class="op">=</span> last_item.order <span class="op">+</span> <span class="dv"><span class="hljs-number">1</span></span>
            <span class="cf"><span class="hljs-keyword">except</span></span> ObjectDoesNotExist:
                value <span class="op">=</span> <span class="dv"><span class="hljs-number">0</span></span>
            <span class="bu">setattr</span>(model_instance, <span class="va">self</span>.attname, value)
            <span class="cf"><span class="hljs-keyword">return</span></span> value
        <span class="cf"><span class="hljs-keyword">else</span></span>:
            <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(OrderField,
                        <span class="va">self</span>).pre_save(model_instance, add)                    </code></pre></div>
<p>这就是我们的定制<code>OrderField</code>.它继承自Django提供的<code>PositiveIntegerField</code>字段。我们的<code>OrderField</code>字段需要一个可选的<code>for_fields</code>参数，这个参数允许我们表明次序根据这些字段进行计算。</p>
<p>我们的字段覆盖<code>PositiveIntegerField</code>字段的<code>pre_save()</code>方法，这字段会在保存这个字段到数据库之前进行执行。在这个方法中，我们做了以下操作：</p>
<ul>
<li><p>1 我们检查在模型实例中的字段是否已有一个值。我们是<code>self.attname</code>，它是在这个模型中给予这个字段的属性名。如果在这个属性的值不同于<code>None</code>，我们就会进行如下操作来计算出一个次序给它：</p>
<ul>
<li>1 我们构建一个查询集去检索所有对象给这个字段的模型。我们通过访问<code>self.model</code>来检索该字段所属的模型类。</li>
<li>2 我们通过模型字段中的那些被定义在<code>for_fields</code>参数中的字段的当前值来过滤这个查询集（如果有的话）。为了做到这点，我们通过给予的字段来计算次序。</li>
<li>3 我们从数据库中使用最高的次序来检索对象通过是用<code>last_item = qs.latest(self.attname)</code>。如果没有找到对象，我们假定这个对象是第一个并且分配次序0给它。</li>
<li>4 如果找到一个对象，我们给找到的最高次序增加1。</li>
<li>5 我们分配计算过的次序给在模型实例中的字段的值通过使用<code>setattr()</code>并且返回它。</li>
</ul></li>
<li><p>2 如果这个模型实例有一个值给当前的字段，我们不需要做任何事情。</p></li>
</ul>
<blockquote>
<p>当你创建定制模型字段，使它们通过。避免硬编码数据被依赖一个指定模型或者字段。你的字段才能在任意模型中起效。</p>
</blockquote>
<p>你可以找到更多的信息关于编写定制模型字段，通过访问 <a href="https://docs.djangoproject.com/en/1.8/howto/custom-model-fields/" class="uri">https://docs.djangoproject.com/en/1.8/howto/custom-model-fields/</a> 。</p>
<p>让我们添加新的字段给我们的模型。编辑<em>courses</em>应用的<em>models.py</em>文件，导入新的字段如下所示：</p>
<pre><code class="hljs swift">from .fields <span class="hljs-keyword">import</span> OrderField</code></pre>
<p>之后，添加以下<code>OrderField</code>字段给<code>Module</code>模型：</p>
<pre><code class="hljs ini"><span class="hljs-attr">order</span> = OrderField(blank=<span class="hljs-literal">True</span>, for_fields=[<span class="hljs-string">'course'</span>])</code></pre>
<p>我们命名新的字段为<code>order</code>，并且我们指定该字段的次序根据课程计算通过设置<code>for_fields=['course']</code>。这意味着新的模块的次序将会是最后的同样的<em>Course</em>对象模块的次序增加1。现在你可以编辑<code>Module</code>模型的<code>__str__()</code>方法来包含它的次序如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'{}. {}'</span></span>.<span class="bu">format</span>(<span class="va">self</span>.order, <span class="va">self</span>.title)</code></pre></div>
<p>模块内容也需要跟随一个特定的次序。添加一个<code>OrderField</code>字段给<code>Content</code>模型如下所示：</p>
<pre><code class="hljs ini"><span class="hljs-attr">order</span> = OrderField(blank=<span class="hljs-literal">True</span>, for_fields=[<span class="hljs-string">'module'</span>])</code></pre>
<p>这一次，我们指定这个次序根据<code>moduel</code>字段进行计算。最后，让我们给这两个模型都添加一个默认的序列。添加如下<code>Meta</code>类给<code>Module</code>和<code>Content</code>模型：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
    ordering <span class="op">=</span> [<span class="st"><span class="hljs-string">'order'</span></span>]</code></pre></div>
<p><code>Module</code>和<code>Content</code>模型现在看上去如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Module</span><span class="hljs-params">(models.Model)</span>:</span>
    course <span class="op">=</span> models.ForeignKey(Course,related_name<span class="op">=</span><span class="st"><span class="hljs-string">'modules'</span></span>)
    title <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>)
    description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    order <span class="op">=</span> OrderField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>, for_fields<span class="op">=</span>[<span class="st"><span class="hljs-string">'course'</span></span>])
    
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> [<span class="st"><span class="hljs-string">'order'</span></span>]
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'{}. {}'</span></span>.<span class="bu">format</span>(<span class="va">self</span>.order, <span class="va">self</span>.title)
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Content</span><span class="hljs-params">(models.Model)</span>:</span>
    module <span class="op">=</span> models.ForeignKey(Module, related_name<span class="op">=</span><span class="st"><span class="hljs-string">'contents'</span></span>)
    content_type <span class="op">=</span> models.ForeignKey(ContentType,
                    limit_choices_to<span class="op">=</span>{<span class="st"><span class="hljs-string">'model__in'</span></span>:(<span class="st"><span class="hljs-string">'text'</span></span>,
                                                      <span class="st"><span class="hljs-string">'video'</span></span>,
                                                      <span class="co"><span class="hljs-string">'file'</span></span>)})
    item <span class="op">=</span> GenericForeignKey(<span class="st"><span class="hljs-string">'content_type'</span></span>, <span class="st"><span class="hljs-string">'object_id'</span></span>)
    order <span class="op">=</span> OrderField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>, for_fields<span class="op">=</span>[<span class="st"><span class="hljs-string">'module'</span></span>])
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> [<span class="st"><span class="hljs-string">'order'</span></span>]        </code></pre></div>
<p>让我们创建一个新模型迁移来体现新的次序字段。打开shell并且运行如下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span> <span class="hljs-selector-tag">courses</span></code></pre>
<p>你会看到如下输出：</p>
<pre class="shell"><code class="hljs sql">You are trying to add a non-nullable field 'order' to content without a default; we can't <span class="hljs-keyword">do</span> that (the <span class="hljs-keyword">database</span> needs something <span class="hljs-keyword">to</span> populate existing <span class="hljs-keyword">rows</span>).
Please <span class="hljs-keyword">select</span> a fix:
 <span class="hljs-number">1</span>) Provide a one-<span class="hljs-keyword">off</span> <span class="hljs-keyword">default</span> <span class="hljs-keyword">now</span> (will be <span class="hljs-keyword">set</span> <span class="hljs-keyword">on</span> all existing <span class="hljs-keyword">rows</span>)
 <span class="hljs-number">2</span>) Quit, <span class="hljs-keyword">and</span> let me <span class="hljs-keyword">add</span> a <span class="hljs-keyword">default</span> <span class="hljs-keyword">in</span> models.py
<span class="hljs-keyword">Select</span> an <span class="hljs-keyword">option</span>:</code></pre>
<p>Django正在告诉我们由于我们添加了一个新的字段给已经存在的模型，我们必须提供一个默认值给数据库中已经存在的各行记录。如果这个字段有<code>null=True</code>，它将会采用空值并且Django将会创建这个迁移而不会找我们要一个默认值。我们可以指定一个默认值或者取消这次迁移然后在创建这个迁移之前去<code>models.py</code>文件中给<code>order</code>字段添加一个<code>default</code>属性。</p>
<p>输入 1 然后按下回车来提供一个默认值给已经存在的记录。你将会看到如下输出：</p>
<pre class="shell"><code class="hljs cs">Please enter the <span class="hljs-keyword">default</span> <span class="hljs-keyword">value</span> now, <span class="hljs-keyword">as</span> valid Python
The datetime and django.utils.timezone modules are available, so you can <span class="hljs-keyword">do</span> e.g. timezone.now()
&gt;&gt;&gt;</code></pre>
<p>输入 0 作为给已经存在的记录的默认值然后按下回车。Djanog将会询问你还需要一个默认值给<code>Module</code>模型。选择第一个选项然后再次输入 0 作为默认值。最后，你将会看到如下类似的输入：</p>
<pre class="shell"><code class="hljs sql">Migrations for 'courses':
 0003_auto_20150701_1851.py:
    - <span class="hljs-keyword">Change</span> Meta options <span class="hljs-keyword">on</span> <span class="hljs-keyword">content</span>
    - <span class="hljs-keyword">Change</span> Meta options <span class="hljs-keyword">on</span> <span class="hljs-keyword">module</span>
    - <span class="hljs-keyword">Add</span> <span class="hljs-keyword">field</span> <span class="hljs-keyword">order</span> <span class="hljs-keyword">to</span> <span class="hljs-keyword">content</span>
    - <span class="hljs-keyword">Add</span> <span class="hljs-keyword">field</span> <span class="hljs-keyword">order</span> <span class="hljs-keyword">to</span> <span class="hljs-keyword">module</span></code></pre>
<p>之后，应用新的迁移通过以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>这个命令的输出将会通知你这次迁移成功的应用，如下所示：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">courses</span><span class="hljs-selector-class">.0003_auto_20150701_1851</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>让我们测试我们新的字段。打开shell使用<code>python manage.py shell</code>然后创建一个新的课程如下所示：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; from django.contrib.auth.models import User
<span class="hljs-meta">&gt;&gt;</span>&gt; from courses.models import Subject, Course, Module
<span class="hljs-meta">&gt;&gt;</span>&gt; user = User.objects.latest(<span class="hljs-string">'id'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; subject = Subject.objects.latest(<span class="hljs-string">'id'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; c1 = Course.objects.create(subject=subject, owner=user,
title=<span class="hljs-string">'Course 1'</span>, slug=<span class="hljs-string">'course1'</span>)</code></pre>
<p>我们已经在数据库中创建了一个课程。现在让我们给课程添加模块然后看下模块的次序是如何自动计算的。我们创建一个初始模板然后检查它的次序：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; m1 = Module.objects.create(course=c1, title=<span class="hljs-string">'Module 1'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; m1.order
<span class="hljs-number">0</span></code></pre>
<p><code>OrderField</code>设置这个模块的值为 0，因为这个模块是这个课程的第一个<code>Module</code>对象。现在我们创建第二个对象给这个课程：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; m2 = Module.objects.create(course=c1, title=<span class="hljs-string">'Module 2'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; m2.order
<span class="hljs-number">1</span></code></pre>
<p><code>OrderField</code>计算出下一个次序值是已经存在的对象中最高的次序值加上 1。让我们创建第三个模块强制指定一个次序：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; m3 = Module.objects.create(course=c1, title=<span class="hljs-string">'Module 3'</span>, order=<span class="hljs-number">5</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; m3.order
<span class="hljs-number">5</span></code></pre>
<p>如果我们指定了一个定制次序，<code>OrderField</code>字段将不会进行干涉，然后<code>order</code>的值将会使用指定的次序。</p>
<p>让我们添加第四个模块：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; m4 = Module.objects.create(course=c1, title=<span class="hljs-string">'Module 4'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; m4.order
<span class="hljs-number">6</span></code></pre>
<p>这第四个模块的次序会被自动设置。我们的<code>OrderField</code>字段不会保证所有的次序值是连续的。无论如何，它会根据已经存在的次序值并且分配下一个次序基于已经存在的最高次序。</p>
<p>让我们创建第二个课程并且添加一个模块给它：</p>
<pre class="shell"><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; c2 = Course.objects.create(subject=subject, title=<span class="hljs-string">'Course 2'</span>, slug=<span class="hljs-string">'course2'</span>, owner=user)
<span class="hljs-meta">&gt;&gt;</span>&gt; m5 = Module.objects.create(course=c2, title=<span class="hljs-string">'Module 1'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; m5.order
<span class="hljs-number">0</span></code></pre>
<p>为了计算这个新模块的次序，该字段只需要考虑基于同一课程的已经存在的模块。由于这是第二个课程的第一个模块，次序的结果值就是 0 。这是因为我们指定<code>for_fields=['course']</code>在<code>Module</code>模型的<code>order</code>字段中。</p>
<p>恭喜你！你已经成功的创建了你的第一个定制模型字段。</p>
<h2 id="创建一个内容管理系统">创建一个内容管理系统</h2>
<p>到现在我们已经创建了一个多功能数据模型，我们将要构建一个内容管理系统（CMS）。这个CMS将允许教师去创建课程以及管理课程的内容。我们需要提供以下功能：</p>
<ul>
<li>登录CMS。</li>
<li>排列教师创建的课程。</li>
<li>创建，编辑以及删除课程。</li>
<li>添加模块到一个课程中并且重新排序它们。</li>
<li>添加不同类型的内容给每个模块并且重新排序内容。</li>
</ul>
<h2 id="添加认证系统">添加认证系统</h2>
<p>我们将要使用Django的认证框架到我们的平台中。教师和学生都将会是Django <em>User</em>模型的一个实例。从而，他们将能够登录这个站点通过使用<code>django.contrib.auth</code>的认证视图。</p>
<p>编辑<em>educa</em>项目的主<em>urls.py</em>文件然后包含Django认证框架的<code>login</code>和<code>logout</code>视图：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> include, url
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.auth <span class="im"><span class="hljs-keyword">import</span></span> views <span class="im"><span class="hljs-keyword">as</span></span> auth_views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^accounts/login/$'</span></span>, auth_views.login, name<span class="op">=</span><span class="st"><span class="hljs-string">'login'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^accounts/logout/$'</span></span>, auth_views.logout, name<span class="op">=</span><span class="st"><span class="hljs-string">'logout'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
]</code></pre></div>
<h2 id="创建认证模板">创建认证模板</h2>
<p>在<em>courses</em>应用目录下创建如下文件结构：</p>
<pre class="shell"><code class="hljs cs">templates/
    <span class="hljs-keyword">base</span>.html
    registration/
        login.html
        logged_out.html</code></pre>
<p>在构建认证模板之前，我们需要给我们的项目准备好基础模板。编辑<em>base.html</em>模板文件然后添加以下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load staticfiles <span>%</span><span>}</span>
<span class="dt"><span class="hljs-meta">&lt;!DOCTYPE </span></span><span class="hljs-meta">html</span><span class="dt"><span class="hljs-meta">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">charset</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"utf-8"</span></span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span><span>{</span><span>%</span> block title <span>%</span><span>}</span>Educa<span>{</span><span>%</span> endblock <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">link</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">css</span>/<span class="hljs-attr">base.css</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">rel</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"stylesheet"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"header"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"logo"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Educa<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"menu"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>%</span> if request.user.is_authenticated <span>%</span><span>}</span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">logout"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Sign out<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
         <span>{</span><span>%</span> else <span>%</span><span>}</span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">login"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Sign in<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
         <span>{</span><span>%</span> endif <span>%</span><span>}</span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"content"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span>{</span><span>%</span> block content <span>%</span><span>}</span>
       <span>{</span><span>%</span> endblock <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">script</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span></span><span class="javascript">
       </span><span class="at"><span class="javascript">$</span></span><span class="javascript">(<span class="hljs-built_in">document</span>).</span><span class="at"><span class="javascript">ready</span></span><span class="javascript">(</span><span class="kw"><span class="javascript"><span class="hljs-function"><span class="hljs-keyword">function</span></span></span></span><span class="javascript"><span class="hljs-function">(<span class="hljs-params"></span>) </span></span><span class="op"><span class="javascript">{</span></span><span class="javascript">
         </span><span class="op"><span class="javascript"><span>{</span><span>%</span></span></span><span class="javascript"> block domready </span><span class="op"><span class="javascript"><span>%</span><span>}</span></span></span><span class="javascript">
         </span><span class="op"><span class="javascript"><span>{</span><span>%</span></span></span><span class="javascript"> endblock </span><span class="op"><span class="javascript"><span>%</span><span>}</span></span></span><span class="javascript">
       </span><span class="op"><span class="javascript">}</span></span><span class="javascript">)</span><span class="op"><span class="javascript">;</span></span><span class="javascript">
     </span><span class="op"><span class="hljs-tag">&lt;</span></span><span class="ss"><span class="hljs-tag">/<span class="hljs-name">script</span>&gt;</span></span>
<span class="ss">   <span class="hljs-tag">&lt;/<span class="hljs-name">body</span></span></span><span class="op"><span class="hljs-tag">&gt;</span></span>
<span class="op"><span class="hljs-tag">&lt;</span></span><span class="ss"><span class="hljs-tag">/<span class="hljs-name">html</span>&gt;</span></span></code></pre></div>
<p>这个基础模板将会被其他的模板扩展。在这个模板中，我们定义了以下区块：</p>
<ul>
<li>title：这个区块是给别的模板用来给每个页面添加定制的标题。</li>
<li>content：这个是内容的主区块。所有扩展基础模板的模板都可以添加各自的内容到这个区块。</li>
<li>domready：位于jQuery的<code>$document.ready()</code>方法里面。它允许我们执行代码当DOM完成加载的时候。</li>
</ul>
<p>这个模板使用的CSS样式位于本章实例代码的<em>courses</em>应用下的<em>static/</em>目录下。你可以拷贝<em>static/</em>目录到你的项目的相同目录下来使用它们。</p>
<p>编辑<em>registration/login.html</em>模板并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>Log-in<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Log-in<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span>{</span><span>%</span> if form.errors <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Your username and password didn't match. Please try again.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
       <span>{</span><span>%</span> else <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Please, use the following form to log-in:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
       <span>{</span><span>%</span> endif <span>%</span><span>}</span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"login-form"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url 'login' <span>%</span><span>}</span>"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
           <span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
           <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"hidden"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">name</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"next"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> next <span>}</span><span>}</span>"</span></span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Log-in"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这是一个给Django的<code>login</code>视图用的标准登录模板。编辑<em>registration/logged_out.html</em>模板然后添加以下代码：</p>
<pre class="shell"><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
   
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Logged out</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
     <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Logged out<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"module"</span>&gt;</span>
       <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>You have been successfully logged out. You can <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span>"login" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>log-in again<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
     <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板将会在用户登出后展示。通过命令<code>python manage.py runserver</code>命令运行开发服务器然后在你的浏览器中打开 <a href="http://127.0.0.1:8000/accounts/login/" class="uri">http://127.0.0.1:8000/accounts/login/</a> 。你会看到如下登录页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-2.png" alt="django-10-2"></p>
<h2 id="创建基于类的视图">创建基于类的视图</h2>
<p>我们将要构建一些视图用来创建，编辑，以及删除课程。为了这个目的我们将会使用基于类的视图。编辑<em>courses</em>应用的<em>views.py</em>文件并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.<span class="bu">list</span> <span class="im"><span class="hljs-keyword">import</span></span> ListView
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ManageCourseListView</span><span class="hljs-params">(ListView)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/list.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_queryset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        qs <span class="op">=</span> <span class="bu">super</span>(ManageCourseListView, <span class="va">self</span>).get_queryset()
        <span class="cf"><span class="hljs-keyword">return</span></span> qs.<span class="bu">filter</span>(owner<span class="op">=</span><span class="va">self</span>.request.user)</code></pre></div>
<p>以上就是<code>ManageCourseListView</code>视图。它从Django的通用<code>ListView</code>继承而来。我们重写了这个视图的<code>get_queryset()</code>方法来只对当前用户创建的课程进行检索。为了阻止用户对不是由他们创建的课程进行编辑，更新或者删除操作，我们还需要重写在创建，更新以及删除视图中的<code>get_queryse()</code>方法。当你需要去提供一个指定行为给多个基于类的视图，推荐你使用<code>mixins</code>。</p>
<h2 id="对基于类的视图使用mixins">对基于类的视图使用mixins</h2>
<p>mixins是一种特殊的用于一个类的多重继承。你可以使用它们来提供常见的离散功能，添加到其他的mixins，允许你去定义一个类的行为。有两种场景要使用mixins：</p>
<ul>
<li>你想要提供多个可选的特性给一个类</li>
<li>你想要使用一个特定的特性在多个类上</li>
</ul>
<p>你可以找到关于如何在基于类的视图上使用mixins的文档，通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/class-based-views/mixins/" class="uri">https://docs.djangoproject.com/en/1.8/topics/class-based-views/mixins/</a> 。</p>
<p>Django自带多个mixins用来提供额外的功能给你的基于类的视图。你可以找到所有的mixins在 <a href="https://docs.djangoproject.com/en/1.8/ref/class-based-views/mixins/" class="uri">https://docs.djangoproject.com/en/1.8/ref/class-based-views/mixins/</a> 。</p>
<p>我们将要创建一个mixin类来包含一个公用的行为并且将它给课程的视图使用。编辑<em>courses</em>应用的<em>views.py</em>文件，把它修改成如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse_lazy
<span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.<span class="bu">list</span> <span class="im"><span class="hljs-keyword">import</span></span> ListView
<span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.edit <span class="im"><span class="hljs-keyword">import</span></span> CreateView, UpdateView, <span class="op">\</span>
                                         DeleteView
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OwnerMixin</span><span class="hljs-params">(</span></span><span class="bu"><span class="hljs-class"><span class="hljs-params">object</span></span></span><span class="hljs-class"><span class="hljs-params">)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_queryset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        qs <span class="op">=</span> <span class="bu">super</span>(OwnerMixin, <span class="va">self</span>).get_queryset()
        <span class="cf"><span class="hljs-keyword">return</span></span> qs.<span class="bu">filter</span>(owner<span class="op">=</span><span class="va">self</span>.request.user)

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OwnerEditMixin</span><span class="hljs-params">(</span></span><span class="bu"><span class="hljs-class"><span class="hljs-params">object</span></span></span><span class="hljs-class"><span class="hljs-params">)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">form_valid</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, form)</span>:</span>
        form.instance.owner <span class="op">=</span> <span class="va">self</span>.request.user
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(OwnerEditMixin, <span class="va">self</span>).form_valid(form)
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OwnerCourseMixin</span><span class="hljs-params">(OwnerMixin)</span>:</span>
    model <span class="op">=</span> Course
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OwnerCourseEditMixin</span><span class="hljs-params">(OwnerCourseMixin, OwnerEditMixin)</span>:</span>
    fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'overview'</span></span>]
    success_url <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'manage_course_list'</span></span>)
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/form.html'</span></span>
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ManageCourseListView</span><span class="hljs-params">(OwnerCourseMixin, ListView)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/list.html'</span></span>
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseCreateView</span><span class="hljs-params">(OwnerCourseEditMixin, CreateView)</span>:</span>
    <span class="cf"><span class="hljs-keyword">pass</span></span>
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseUpdateView</span><span class="hljs-params">(OwnerCourseEditMixin, UpdateView)</span>:</span>
    <span class="cf"><span class="hljs-keyword">pass</span></span>
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseDeleteView</span><span class="hljs-params">(OwnerCourseMixin, DeleteView)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/delete.html'</span></span>
    success_url <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'manage_course_list'</span></span>)</code></pre></div>
<p>在上述代码中，我们创建了<code>OwnerMixin</code>和<code>OwnerEditMixin</code>这两个mixin。我们将要使用这些mixins与Django提供的<code>ListView</code>，<code>CreateView</code>，<code>UpdateView</code>以及<code>DeleteView</code>视图结合。<code>Ownermixin</code>导入了以下方法。</p>
<ul>
<li><code>get_queryset()</code>：这个方法被视图用来获取基础查询集。我们的mixin将会重写这个方法使用<code>owner</code>属性对对象进行过滤来检索属于当前用户的对象（request.user)。</li>
</ul>
<p><code>OwnerEditMixin</code>导入了以下方法：</p>
<ul>
<li><code>form_valid()</code>：这个方法被视图用来使用Django的<code>ModelFormMixin</code> mixin，也就是说，带有表单的视图或模型表单的视图比如<code>Createview</code>和<code>UpdateView.form_valid()</code>当提交的表单是有效的时候就会被执行。这个方法默认的行为是保存实例（对于模型表单）以及重定向用户到<code>success_url</code>。我们重写了这个方法来自动设置当前的用户到本次会被保存的对象的<code>owner</code>属性中。为了做到前面所说的，我们设置自动分配一个拥有者给该对象，当该对象被保存的时候。</li>
</ul>
<p>我们的<code>OwnerMixin</code>类能够被视图用来和任意模型进行交互使模型包含一个<code>owner</code>属性。</p>
<p>我们还定义了一个<code>OwnercourseMixin</code>类，该类继承<code>OwnerMixin</code>并且提供以下属性给子视图：</p>
<ul>
<li><code>model</code>：这个模型给查询集使用。被所有视图使用。</li>
</ul>
<p>我们定义一个<code>OwnerCourseEditMixin</code> mixin通过以下属性：</p>
<ul>
<li><code>fields</code>：这些模型字段用来从<code>CreateView</code>和<code>UpdateView</code>视图中构建模型。</li>
<li><code>success_url</code>：被<code>CreateView</code>和<code>UpdateView</code>使用来在表单成功提交之后重定向用户。我们之后将会创建一个名为<code>manage_course_list</code>的URL来使用。</li>
</ul>
<p>最后，我们创建以下视图，这些视图都是基于<code>OwnerCourseMixin</code>的子类：</p>
<ul>
<li><code>MangeCourselISTvIEW</code>：排序用户创建的课程。它从<code>OwnerCourseMixin</code>和<code>ListView</code>继承而来。</li>
<li><code>CoursecreateView</code>：使用模型表单来创建一个新的<code>Course</code>对象。它使用定义在<code>OwnerCourseEditMixin</code>中的字段来构建一个表单模型并且也是<code>CreateView</code>的子类。</li>
<li><code>CourseUpdateView</code>：允许编辑一个现有的<code>Course</code>对象。它从<code>OwnerCourseMixin</code>和<code>UpdateView</code>继承而来。</li>
<li><code>CourseDeleteView</code>：从<code>OwnerCourseMixin</code>和通用的<code>DeleteView</code>继承而来。定义<code>success_url</code>在对象被删除的时候重定向用户。</li>
</ul>
<h2 id="使用组和权限">使用组和权限</h2>
<p>我们已经创建了基础的视图来管理课程。但是目前所有的用户都可以使用这些视图。我们想要限制这些视图从而只有教师有权限去创建和管理课程。Django认证框架包含一个权限系统允许你去分配权限给用户和组。我们将要创建一个组给教师用户并且分配权限给他们可以创建，更新以及删除课程。</p>
<p>使用命令<code>python manage.py runserver</code>命令运行开发服务器并且在你的浏览器中打开 <a href="http://127.0.0.1:8000/admin/auth/group/add/" class="uri">http://127.0.0.1:8000/admin/auth/group/add/</a> 来创建一个新的<code>Group</code>对象。添加的组名为<em>Instructors</em>,然后选择<em>courses</em>应用中的除了<em>Subject</em>模型的所有权限，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-3.png" alt="django-10-3"></p>
<p>如你所见，有三种不同的权限给每个模型：<strong>Can add</strong>，<strong>can change</strong>以及<strong>Can delete</strong>。选择好给这个组的权限之后，点击<strong>Save</strong>按钮。</p>
<p>Django会自动给模型创建权限，但是你也可以创建定制的权限。你可以找到更多关于添加定制权限的文档，通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#custom-permissions" class="uri">https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#custom-permissions</a> 。</p>
<p>打开 <a href="http://127.0.0.1:8000/admin/auth/user/add/" class="uri">http://127.0.0.1:8000/admin/auth/user/add/</a> 然后创建一个新用户。编辑这个用户然后添加<strong>Instructors</strong>组给这个用户如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-4.png" alt="django-10-4"></p>
<p>用户会继承他们所在组的权限，但是你也可以在管理平台上添加单独的权限给一个指定的用户。当用户的<em>is_superuser</em>设置为<em>True</em>的时候会自动拥有所有的权限。</p>
<h2 id="限制使用基于类的视图">限制使用基于类的视图</h2>
<p>我们将要限制使用基于类的视图从而只有那些拥有合适权限的用户才能添加，修改，或者删除<code>Course</code>对象认证框架包含一个<code>permission_required</code>装饰器来限制对视图的使用。Django 1.9将会包含权限mixins给基于类的视图<strong>（译者注：到目前为止，Django版本已经是1.10.6）</strong>。然而，Django1.8还没有包含它们。因此，我们将要第三方模块提供的权限mixins，该第三方模块名为 django-braces<strong>（译者注：。。。。。。下面我是不是可以不用翻译了。。。。。。）</strong>。</p>
<h2 id="使用django-braces的mixins">使用django-braces的mixins</h2>
<p>Django-braces是一个第三方的模块，它包含一个通用mixins的采集给Django使用。这些mixins提供额外的特性给基于类的视图。你可以看到django-braces提供的所有mixins列表，通过访问 <a href="http://django-braces.readthedocs.org/en/latest/" class="uri">http://django-braces.readthedocs.org/en/latest/</a>。</p>
<p>使用pip命令安装django-braces：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-braces==<span class="hljs-number">1.8</span>.<span class="hljs-number">1</span></code></pre>
<p>我们将要使用以下两个django-braces提供的mixinx来限制视图的使用：</p>
<ul>
<li><em>LoginRequiredMixin</em>：复制<code>login_required</code>装饰器的功能。</li>
<li><em>PermissionRequiredMixin</em>：准许拥有指定权限的用户使用该视图。请记住，超级用户自动拥有所有权限。</li>
</ul>
<p>编辑<em>courses</em>应用的<em>views.py</em>文件，添加如下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> braces.views <span class="im"><span class="hljs-keyword">import</span></span> LoginRequiredMixin,
                            PermissionRequiredMixin</code></pre></div>
<p>像下面一样让<code>OwnerCourseMixin</code>继承<code>LoginRequiredMixin</code>：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OwnerCourseMixin</span><span class="hljs-params">(OwnerMixin, LoginRequiredMixin)</span>:</span> 
    model <span class="op">=</span> Course
    fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'overview'</span></span>]
    success_url <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'manage_course_list'</span></span>)</code></pre></div>
<p>之后，添加一个<code>permission_required</code>属性给创建，跟新，以及删除视图，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseCreateView</span><span class="hljs-params">(PermissionRequiredMixin,
                       OwnerCourseEditMixin,
                       CreateView)</span>:</span>
    permission_required <span class="op">=</span> <span class="st"><span class="hljs-string">'courses.add_course'</span></span>
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseUpdateView</span><span class="hljs-params">(PermissionRequiredMixin,
                       OwnerCourseEditMixin,
                       UpdateView)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/form.html'</span></span>
    permission_required <span class="op">=</span> <span class="st"><span class="hljs-string">'courses.change_course'</span></span>
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseDeleteView</span><span class="hljs-params">(PermissionRequiredMixin, 
                       OwnerCourseMixin,
                       DeleteView)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/course/delete.html'</span></span>
    success_url <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'manage_course_list'</span></span>)
    permission_required <span class="op">=</span> <span class="st"><span class="hljs-string">'courses.delete_course'</span></span></code></pre></div>
<p><code>PermissionRequiredMixin</code>会在用户使用视图的时候检查该用户是否有指定在<code>permission_required</code>属性中的权限。我们的视图现在只准许有适当权限的用户使用。</p>
<p>让我们给以上视图创建URLs。在<em>courses</em>应用目录中创建新的文件命名为<em>urls.py</em>。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^mine/$'</span></span>,
        views.ManageCourseListView.as_view(),
        name<span class="op">=</span><span class="st"><span class="hljs-string">'manage_course_list'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^create/$'</span></span>,
        views.CourseCreateView.as_view(),
        name<span class="op">=</span><span class="st"><span class="hljs-string">'course_create'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;pk&gt;\d+)/edit/$'</span></span>,
        views.CourseUpdateView.as_view(),
        name<span class="op">=</span><span class="st"><span class="hljs-string">'course_edit'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;pk&gt;\d+)/delete/$'</span></span>,
        views.CourseDeleteView.as_view(),
        name<span class="op">=</span><span class="st"><span class="hljs-string">'course_delete'</span></span>),
]</code></pre></div>
<p>以上的URL模式是给列表，创建，编辑以及删除课程试图使用的。编辑<em>educa</em>项目的主<em>urls.py</em>文件然后包含<em>courses</em>应用的URL模式，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^accounts/login/$'</span></span>, auth_views.login, name<span class="op">=</span><span class="st"><span class="hljs-string">'login'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^accounts/logout/$'</span></span>, auth_views.logout, name<span class="op">=</span><span class="st"><span class="hljs-string">'logout'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
    url(<span class="vs"><span class="hljs-string">r'^course/'</span></span>, include(<span class="st"><span class="hljs-string">'courses.urls'</span></span>)),
]</code></pre></div>
<p>我们需要给这些视图创建模块。在<em>courses</em>应用中创建以下目录以及文件：</p>
<pre class="shell"><code class="hljs cpp">courses/
       manage/
           course/
               <span class="hljs-built_in">list</span>.html
               form.html
               <span class="hljs-keyword">delete</span>.html</code></pre>
<p>编辑 <em>courses/manage/course/list.html</em>模板并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>My courses<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>My courses<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span>{</span><span>%</span> for course in object_list <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"course-info"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span><span>{</span><span>{</span> course.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_edit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_delete"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Delete<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
       <span>{</span><span>%</span> empty <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>You haven't created any courses yet.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
       <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_create"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Create new course<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这是<code>ManageCourseListView</code>视图的模板。在这个模板中，我们通过当前用户来排列课程。我们给每个课程都包含了编辑或者删除链接，以及一个创建新课程的链接。</p>
<p>使用命令<code>python manage.py runserver</code>命令运行开发服务器。在你的浏览器中打开 <a href="http://127.0.0.1:8000/accounts/login/?next=/course/mine/" class="uri">http://127.0.0.1:8000/accounts/login/?next=/course/mine/</a> 然后使用<em>Instrctors</em>组中的一个用户进行登录。登录完成后，你会被重定向到 <a href="http://127.0.0.1:8000/course/mine/" class="uri">http://127.0.0.1:8000/course/mine/</a> 并且你会看到如下页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-5.png" alt="django-10-5"></p>
<p>这个页面将会展示所有当前用户创建的课程。</p>
<p>让我们创建一个给创建和更新课程视图使用的模板，该模板用来展示表单。编辑<em>courses/manage/course/form.html</em>模板并且输入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">   <span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>
   <span>{</span><span>%</span> block title <span>%</span><span>}</span>
     <span>{</span><span>%</span> if object <span>%</span><span>}</span>
       Edit course "<span>{</span><span>{</span> object.title <span>}</span><span>}</span>"
     <span>{</span><span>%</span> else <span>%</span><span>}</span>
       Create a new course
     <span>{</span><span>%</span> endif <span>%</span><span>}</span>
   <span>{</span><span>%</span> endblock <span>%</span><span>}</span>
   <span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
       <span>{</span><span>%</span> if object <span>%</span><span>}</span>
         Edit course "<span>{</span><span>{</span> object.title <span>}</span><span>}</span>"
       <span>{</span><span>%</span> else <span>%</span><span>}</span>
         Create a new course
       <span>{</span><span>%</span> endif <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Course info<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"."</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
         <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Save course"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
   <span>{</span><span>%</span> endblock <span>%</span><span>}</span>  </code></pre></div>
<p>这个<em>form.html</em>模板被<code>CoursecREATEvIEW</code>和<code>courseUpdateView</code>视图使用。在这个模板中，我们检查是否有个<em>object</em>变量在上下文环境中。如果<em>object</em>存在上下文环境中，我们就知道我们正在更新一个存在的课程，并且我们在页面标题中使用它。如果不存在，我们就要创建一个新的<em>Course</em>对象。</p>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/course/mine/" class="uri">http://127.0.0.1:8000/course/mine/</a> 然后点击<strong>Create new course</strong>按钮。你会看到如下页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-6.png" alt="django-10-6"></p>
<p>填写好表单内容然后点击<strong>Save course</strong>按钮。这个课程将会被保存并且你将会被重定向到课程列表页面。它看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-7.png" alt="django-10-7"></p>
<p>之后，点击你刚才创建的课程的<strong>Edit</strong>链接。你将会再次看到表单，但是这一次你将编辑一个已经存在的<em>Course</em>对象而不是创建新课程。</p>
<p>最后，编辑<em>courses/manage/course/delete.html</em>模板然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">   <span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>
   <span>{</span><span>%</span> block title <span>%</span><span>}</span>Delete course<span>{</span><span>%</span> endblock <span>%</span><span>}</span>
   <span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Delete course "<span>{</span><span>{</span> object.title <span>}</span><span>}</span>"<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">""</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Are you sure you want to delete "<span>{</span><span>{</span> object <span>}</span><span>}</span>"?<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span></span></span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">button</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Confirm"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
   <span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板是给<code>CourseDeleteView</code>视图使用的。这个视图从Django提供的<code>DeleteView</code>视图继承而来，<code>DeleteView</code>视图期望用户确认删除一个对象。</p>
<p>打开你的浏览器，点击你的课程的<strong>Delete</strong>链接。你会看到如下确认页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-8.png" alt="django-10-8"></p>
<p>点击<strong>CONFIRM</strong>按钮。这个课程将会被删除并且你再次会被重定向到课程列表页面。</p>
<p>教师们现在可以创建，编辑，以及删除课程。下一步，我们需要提供他们一个内容管理系统来给课程添加模块以及内容。我们将从管理课程模块开始。</p>
<h2 id="使用formsets">使用formsets</h2>
<p>Django自带一个抽象层用于在同一个页面中使用多个表单。这些表单的组合成为formsets。formsets能管理多个<em>Form</em>或者<em>ModelForm</em>表单实例。所有的表单都可以一次性提交并且formset会照顾到一些事情，例如，表单的初始化数据展示，限制表单能够提交的最大数字，以及所有表单的验证。</p>
<p>formsets包含一个<code>is_valid()</code>方法来一次性验证所有表单。你还可以提供初始数据给表单以及指定展示任意多的额外的空的表单。</p>
<p>你可以学习到更多关于formsets，通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/forms/modelforms/#model-formsets" class="uri">https://docs.djangoproject.com/en/1.8/topics/forms/modelforms/#model-formsets</a> 。</p>
<h2 id="管理课程模块">管理课程模块</h2>
<p>由于课程会被分为可变数量的模块，因此在这里使用formets是有意义的。在<em>courses</em>应用目录下创建一个<em>forms.py</em>文件，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> django.forms.models <span class="im"><span class="hljs-keyword">import</span></span> inlineformset_factory
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Course, Module

ModuleFormSet <span class="op">=</span> inlineformset_factory(Course,
                                         Module,
                                         fields<span class="op">=</span>[<span class="st"><span class="hljs-string">'title'</span></span>,
                                                 <span class="st"><span class="hljs-string">'description'</span></span>],
                                         extra<span class="op">=</span><span class="dv"><span class="hljs-number">2</span></span>,
                                         can_delete<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>以上就是<code>ModuleFormSet</code> formset。我们使用Django提供的<code>inlineformset_factory()</code>函数来构建它。内联formsets是在formsets之上的一个小抽象，用于方便被关联对象的操作。这个函数允许我们去给关联到一个<em>Course</em>对象的<em>Module</em>对象动态的构建一个模型formset。</p>
<p>我们使用以下参数去构建formset：</p>
<ul>
<li>fields：这个字段将会被formset中的每个表单包含。</li>
<li>extra：允许我们设置在formset中显示的空的额外的表单数。</li>
<li>can_delete：如果你将这个参数设置为<em>True</em>，Django将会包含一个布尔字段给所有的表单，该布尔字段将会渲染成一个复选框。允许你确定这个对象你真的要进行删除。</li>
</ul>
<p>编辑<em>courses</em>应用的<em>views.py</em>文件并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> redirect, get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.base <span class="im"><span class="hljs-keyword">import</span></span> TemplateResponseMixin, View
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> ModuleFormSet

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseModuleUpdateView</span><span class="hljs-params">(TemplateResponseMixin, View)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/module/formset.html'</span></span>
    course <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_formset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, data</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> ModuleFormSet(instance<span class="op">=</span><span class="va">self</span>.course,data<span class="op">=</span>data)

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">dispatch</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, pk)</span>:</span>
        <span class="va">self</span>.course <span class="op">=</span> get_object_or_404(Course,
                                        <span class="bu">id</span><span class="op">=</span>pk,
                                        owner<span class="op">=</span>request.user)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(CourseModuleUpdateView,
                     <span class="va">self</span>).dispatch(request, pk)
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        formset <span class="op">=</span> <span class="va">self</span>.get_formset()
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'course'</span></span>: <span class="va">self</span>.course,
                                        <span class="st"><span class="hljs-string">'formset'</span></span>: formset})
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        formset <span class="op">=</span> <span class="va">self</span>.get_formset(data<span class="op">=</span>request.POST)
        <span class="cf"><span class="hljs-keyword">if</span></span> formset.is_valid():
            formset.save()
            <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'manage_course_list'</span></span>)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'course'</span></span>: <span class="va">self</span>.course,
                                        <span class="st"><span class="hljs-string">'formset'</span></span>: formset})        </code></pre></div>
<p><code>CourseModuleUpdateView</code>视图控制formset给一个指定的课程添加，更新，以及删除模块。这个视图继承自以下的mixins和视图：</p>
<ul>
<li><p>TemplateResponseMixin：这个mixins负责渲染模板以及返回一个HTTP响应。它需要一个template_name属性，该属性指明要被渲染的模板，并提供<code>render_to_ response()</code>方法来传递上下文并渲染模板。</p></li>
<li><p>View：Django提供的基础的基于类的视图。</p></li>
</ul>
<p>在这个视图中，我们导入以下方法：</p>
<ul>
<li><p>get_formset()：我们定义这个方法去避免重复构建formset的代码。我们使用可选数据为给予的<code>Course</code>对象创建一个<code>ModuleFormSet</code>对象。</p></li>
<li><p>dispatch()：这个方法由<code>View</code>类提供。它需要一个HTTP请求及其参数并尝试委托一个与使用的HTTP方法匹配的小写方法：GET请求被委派给<code>get()</code>方法和一个POST请求到<code>post()</code>。在这种方法中，我们使用<code>get_object_or_404()</code>快捷方式函数获取属于当前用户的给予id参数的Course对象。我们将这串代码包含在<code>dispatch()</code>方法中是因为我们需要检索所有GET和POST请求的课程。我们保存该对象到这个视图的<code>course</code>属性给使它能被别的方法使用。</p></li>
<li>get()：给GET请求执行。我们构建一个空的<code>ModuleFormSet</code> formset并且使用<code>TemplateResponseMixin</code>提供的<code>render_to_response()</code>方法将它与当前的<code>Course</code>对象一起渲染到模板中。</li>
<li><p>post()：给POST请求执行。在这个方法中，我们执行以下操作：</p>
<ul>
<li>1 我们使用提交的数据构建一个<code>ModuleFormSet</code>实例。</li>
<li>2 我们执行formset的<code>is_valid()</code>方法来验证其中的所有表单。</li>
<li>3 如果这个formset验证通过，我们通过调用<code>save()</code>方法来保存它。在这点上，任何的修改操作，例如增加，更新或者标记模块用来删除，都会应用到数据库中。之后，我们重定向用户到<code>manage_course_list</code> URL。如果这个formset没有通过验证，我们渲染模板展示所有内置的错误信息。</li>
</ul></li>
</ul>
<p>编辑<em>courses</em>应用的<em>urls.py</em>文件，添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^(?P&lt;pk&gt;\d+)/module/$'</span></span>,
    views.CourseModuleUpdateView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'course_module_update'</span></span>),</code></pre></div>
<p>在<em>courses/manage/</em>模板目录中创建一个新的目录命名为<em>module</em>。创建一个<em>courses/manage/module/formset.html</em>模板并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
     Edit "<span>{</span><span>{</span> course.title <span>}</span><span>}</span>"
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Edit "<span>{</span><span>{</span> course.title <span>}</span><span>}</span>"<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Course modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">""</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>{</span> formset <span>}</span><span>}</span>
         <span>{</span><span>{</span> formset.management_form <span>}</span><span>}</span>
         <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Save modules"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>在这个模板中，我们创建一个<code>&lt;form&gt;</code>HTML元素，在其中我们包含我们的<code>formset</code>。我们还通过变量<code><span>{</span><span>{</span> formset.management_form <span>}</span><span>}</span></code>包含给formset使用的管理表单。这个管理表单包含隐藏的字段去控制保单的初始化，总数，最小值和最大值。如你所见，创建一个formset非常容易。</p>
<p>编辑<em>courses/manage/course/list.html</em>模板并且在课程编辑和删除链接下方添加以下链接给<code>course_module_update</code>使用：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_edit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_delete"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Delete<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_module_update"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span></code></pre></div>
<p>我们已经包含了用来编辑课程模板的链接。在你浏览器中打开 <a href="http://127.0.0.1:8000/course/mine/" class="uri">http://127.0.0.1:8000/course/mine/</a> 然后选择一个课程点击对应的<strong>Edit modules</strong>链接。你会看到一个如下的formset：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-9.png" alt="django-10-9"></p>
<p>这个formset包含所有在这个课程中存在的<em>Module</em>对象的表单。在这些表单之后，有两个空的额外的表单会被展示因为我们给<code>ModuleFormSet</code>设置<code>extra=2</code>。当你保存这个formset的时候，Django将会包含另外两个额外的字段来添加新的模块。</p>
<h2 id="添加内容给课程模块">添加内容给课程模块</h2>
<p>现在，我们需要一个方法来添加内容给课程模块。我们有四种不同的内容类型：文本，视频，图片以及文件。我们可以考虑创建四个不同的视图去保存内容，给每个模型都对应上一个视图。然而，我们将采取更通用的方法，并创建一个处理创建或更新任何内容模型的对象的视图。</p>
<p>编辑<em>courses</em>应用的<em>views.py</em>文件并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.forms.models <span class="im"><span class="hljs-keyword">import</span></span> modelform_factory
<span class="im"><span class="hljs-keyword">from</span></span> django.apps <span class="im"><span class="hljs-keyword">import</span></span> apps
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Module, Content

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ContentCreateUpdateView</span><span class="hljs-params">(TemplateResponseMixin, View)</span>:</span>
    module <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    model <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    obj <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/content/form.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_model</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, model_name)</span>:</span>
        <span class="cf"><span class="hljs-keyword">if</span></span> model_name <span class="op"><span class="hljs-keyword">in</span></span> [<span class="st"><span class="hljs-string">'text'</span></span>, <span class="st"><span class="hljs-string">'video'</span></span>, <span class="st"><span class="hljs-string">'image'</span></span>, <span class="st"><span class="hljs-string">'file'</span></span>]:
            <span class="cf"><span class="hljs-keyword">return</span></span> apps.get_model(app_label<span class="op">=</span><span class="st"><span class="hljs-string">'courses'</span></span>,
                                     model_name<span class="op">=</span>model_name)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va"><span class="hljs-keyword">None</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_form</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, model, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        Form <span class="op">=</span> modelform_factory(model, exclude<span class="op">=</span>[<span class="st"><span class="hljs-string">'owner'</span></span>,
                                                    <span class="st"><span class="hljs-string">'order'</span></span>,
                                                    <span class="co"><span class="hljs-string">'created'</span></span>,
                                                    <span class="co"><span class="hljs-string">'updated'</span></span>])
        <span class="cf"><span class="hljs-keyword">return</span></span> Form(<span class="op">*</span>args, <span class="op">**</span>kwargs)
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">dispatch</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, module_id, model_name, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="va">self</span>.module <span class="op">=</span> get_object_or_404(Module,
                                        <span class="bu">id</span><span class="op">=</span>module_id,
                                    course__owner<span class="op">=</span>request.user)
        <span class="va">self</span>.model <span class="op">=</span> <span class="va">self</span>.get_model(model_name)
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="bu">id</span>:
            <span class="va">self</span>.obj <span class="op">=</span> get_object_or_404(<span class="va">self</span>.model,
                                        <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                                        owner<span class="op">=</span>request.user)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(ContentCreateUpdateView,
              <span class="va">self</span>).dispatch(request, module_id, model_name, <span class="bu">id</span>)</code></pre></div>
<p>以上是<code>ContentCreateUpdateView</code>视图的第一部分。这个视图允许我们去创建和更新不同模块的内容。这个视图定义了以下方法：</p>
<ul>
<li>get_model()：在这儿，我们会对被给予的模型名是否四种内容模型中的一种：text,video,image以及file.之后我们使用Django的<code>apps</code>模块去通过给予的模型名来获取实际的类。如果给予的模型名不是其中的一种，我们返回<code>None</code>。</li>
<li>get_form()：我们使用表单框架的<code>modelform_factory()</code>函数来构建一个动态的表单。由于我们将要给<em>Text</em>，<em>Video</em>，<em>Image</em>以及<em>File</em>模型构建一个表单，我们使用<code>exclude</code>参数去指定要从表单中排除的公共字段，并允许自动包含所有其他属性。通过做到这点，我们不必去知道依赖的模型中锁包含的字段。</li>
<li><p>dispatch()：它检索以下URL参数并且存储相符的模块，模型以及内容对象作为类的属性：</p>
<ul>
<li>module_id：The id for the module that the content is/will be associated with<strong>(译者注：求比较好的翻译)</strong>。</li>
<li>model_name：要创建或更新的内容的模型名。</li>
<li>id：这是将要更新的对象的id。在创建新对象的时候它会是<code>None</code>。</li>
</ul></li>
</ul>
<p>添加以下<code>get()</code>和<code>post()</code>方法给<code>ContentCreateUpdateView</code>：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, module_id, model_name, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    form <span class="op">=</span> <span class="va">self</span>.get_form(<span class="va">self</span>.model, instance<span class="op">=</span><span class="va">self</span>.obj)
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'form'</span></span>: form,
                                       <span class="st"><span class="hljs-string">'object'</span></span>: <span class="va">self</span>.obj})
                                       
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, module_id, model_name, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    form <span class="op">=</span> <span class="va">self</span>.get_form(<span class="va">self</span>.model,
                            instance<span class="op">=</span><span class="va">self</span>.obj,
                            data<span class="op">=</span>request.POST,
                            files<span class="op">=</span>request.FILES)
    <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
        obj <span class="op">=</span> form.save(commit<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>)
        obj.owner <span class="op">=</span> request.user
        obj.save()
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> <span class="bu">id</span>:
            <span class="co"><span class="hljs-comment"># new content</span></span>
            Content.objects.create(module<span class="op">=</span><span class="va">self</span>.module,
        <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'module_content_list'</span></span>, <span class="va">self</span>.module.<span class="bu">id</span>)
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'form'</span></span>: form,
                                       <span class="st"><span class="hljs-string">'object'</span></span>: <span class="va">self</span>.obj})</code></pre></div>
<p>以上方法如下所示：</p>
<ul>
<li>get()：当收到一个GET请求的时候会被执行。我们构建模型表单给<code>Text</code>，<code>Video</code>，<code>Image</code>,以及<code>File</code>实例使用当它们被保存的时候。除此以外，我们不会传递实例给创建新的对象，因为<code>self.obj</code>在没有id提供的时候是<code>None</code>。</li>
<li>post()：当收到一个POST请求的时候会被执行。我们构建模型表单会传递所有提交的数据和文件给该表单。之后我们验证该表单。如果这个表单验证通过，我们创建一个新的对象并且在保存该对象到数据库之前分配<code>request.user</code>作为该对象的拥有者。我们会检查<code>id</code>参数，如果没有提供<code>id</code>，我们就知道当前用户正在创建一个新的对象而不是更新一个已经存在的对象。如果这是一个新的对象，我们创建一个<code>Content</code>对象给给予的模块并且关联新的内容给该模块。</li>
</ul>
<p>编辑<em>courses</em>应用的<em>urls.py</em>文件禀帖添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^module/(?P&lt;module_id&gt;\d+)/content/(?P&lt;model_name&gt;\w+)/create/$'</span></span>,
    views.ContentCreateUpdateView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'module_content_create'</span></span>),
url(<span class="vs"><span class="hljs-string">r'^module/(?P&lt;module_id&gt;\d+)/content/(?P&lt;model_name&gt;\w+)/(?P&lt;id&gt;\d+)/$'</span></span>,
    views.ContentCreateUpdateView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'module_content_update'</span></span>),</code></pre></div>
<p>以上新的URL模式如下：</p>
<ul>
<li>module_content_create：用来创建新的文本，视频，图片或者文件对象并且给一个模块添加这些对象。它包含<code>module_id</code>和<code>model_name</code>参数。前者允许连接新的内容对象给给予的模块。后者指定构建表单使用的内容模型。</li>
<li>module_content_update：用来更新一个已有的文本，视频，图片或者文件对象。它包含<code>module_id</code>和<code>model_name</code>参数，以及一个<code>id</code>参数来辨明那个需要被更新的内容。</li>
</ul>
<p>在<em>courses/manage/</em>模板目录下创建新的目录命名为<em>content</em>。创建模板<em>courses/manage/content/form.html</em>并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">   <span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>
   
   <span>{</span><span>%</span> block title <span>%</span><span>}</span>
     <span>{</span><span>%</span> if object <span>%</span><span>}</span>
       Edit content "<span>{</span><span>{</span> object.title <span>}</span><span>}</span>"
     <span>{</span><span>%</span> else <span>%</span><span>}</span>
       Add a new content
     <span>{</span><span>%</span> endif <span>%</span><span>}</span>
   <span>{</span><span>%</span> endblock <span>%</span><span>}</span>
   
   <span>{</span><span>%</span> block content <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
       <span>{</span><span>%</span> if object <span>%</span><span>}</span>
         Edit content "<span>{</span><span>{</span> object.title <span>}</span><span>}</span>"
       <span>{</span><span>%</span> else <span>%</span><span>}</span>
         Add a new content
       <span>{</span><span>%</span> endif <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Course info<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">""</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">enctype</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"multipart/form-data"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
         <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Save content"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
   <span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板是给<code>ContentCreateUpdateView</code>视图使用的。在这个模板中，我们会检查是否有一个<code>object</code>变量在上下文环境中。如果<code>object</code>存在上下文环境中，我们知道我们正在更新一个已经存在的对象。如果没有，我们在创建一个新的对象。</p>
<p>我们在<code>&lt;form&gt;</code>HTML元素中包含<code>enctype="multipart/form-data"</code>，因为这个表单包含一个文件上传用来给<code>Field</code>和<code>Image</code>内容模型使用。</p>
<p>运行开发服务器。给存在的课程创建一个模块并且在你的浏览器中打开 <a href="http://127.0.0.1:8000/course/module/6/content/image/create/" class="uri">http://127.0.0.1:8000/course/module/6/content/image/create/</a> 。如果有必要，在ULR中修改模块id。你将会看到以下表单用来创建新的<code>Image</code>对象：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-10.png" alt="django-10-10"></p>
<p>先不要提交表单。如果你想要尝试，它将会是失败的，因为我们还没有定义<code>module_content_list</code>的URL。我们一会儿就要去创建它。</p>
<p>我们还需要一个视图去删除内容。编辑<em>courses</em>应用的<em>views.py</em>文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ContentDeleteView</span><span class="hljs-params">(View)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        content <span class="op">=</span> get_object_or_404(Content,
                            <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                            module__course__owner<span class="op">=</span>request.user)
        module <span class="op">=</span> content.module
        content.item.delete()
        content.delete()
        <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'module_content_list'</span></span>, module.<span class="bu">id</span>)</code></pre></div>
<p><code>ContentDeleteView</code>通过给予的id检索<code>content</code>对象，它删除关联的<em>Text</em>，<em>Video</em>，<em>Image</em>以及<em>File</em>对象，并且在最后，它会删除<code>Content</code>对象并且重定向用户到<code>module_content_list</code> URL去排列其他模块的内容。</p>
<p>编辑<em>courses</em>应用的<em>urls.py</em>文件并且添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^content/(?P&lt;id&gt;\d+)/delete/$'</span></span>,
    views.ContentDeleteView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'module_content_delete'</span></span>),</code></pre></div>
<p>现在，教师们可以方便的创建，更新以及删除内容。</p>
<h2 id="管理模块和内容">管理模块和内容</h2>
<p>我们已经构建了用来创建，编辑以及删除课程模块和内容的视图。现在，我们需要一个视图去给一个课程显示所有的模块并且给一个指定的模块排列所有的内容。</p>
<p>编辑<em>courses</em>应用的<em>views.py</em>文件并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ModuleContentListView</span><span class="hljs-params">(TemplateResponseMixin, View)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/manage/module/content_list.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, module_id)</span>:</span>
        module <span class="op">=</span> get_object_or_404(Module,
                                      <span class="bu">id</span><span class="op">=</span>module_id,
                                      course__owner<span class="op">=</span>request.user)
                                      
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'module'</span></span>: module})</code></pre></div>
<p>以上就是<code>ModuleContentListView</code>视图。这个视图通过给予的id拿到<code>Module</code>对象该对象是属于当前的用户并且通过给予的模块渲染一个模板。</p>
<p>编辑<em>courses</em>应用的<em>urls.py</em>文件，添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^module/(?P&lt;module_id&gt;\d+)/$'</span></span>,
    views.ModuleContentListView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'module_content_list'</span></span>),</code></pre></div>
<p>在<em>templates/courses/manage/module/</em>目录下创建新的模板命名为<em>content_list.html</em>，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> block title <span>%</span><span>}</span>
     Module <span>{</span><span>{</span> module.order|add:1 <span>}</span><span>}</span>: <span>{</span><span>{</span> module.title <span>}</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>
<span>{</span><span>%</span> block content <span>%</span><span>}</span>
<span>{</span><span>%</span> with course=module.course <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Course "<span>{</span><span>{</span> course.title <span>}</span><span>}</span>"<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"contents"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"modules"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      <span>{</span><span>%</span> for m in course.modules.all <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">data-id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> m.id <span>}</span><span>}</span>"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">m</span></span></span><span class="hljs-tag"> </span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">=</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="hljs-tag">
</span><span class="ot"><span class="hljs-tag">        <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
          <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_list"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">m.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>
              Module <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"order"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> m.order|add:1 <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span></span>
            <span>{</span><span>{</span> m.title <span>}</span><span>}</span>
          <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span> 
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
      <span>{</span><span>%</span> empty <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>No modules yet.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
      <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_module_update"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Module <span>{</span><span>{</span> module.order|add:1 <span>}</span><span>}</span>: <span>{</span><span>{</span> module.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Module contents:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module-contents"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      <span>{</span><span>%</span> for content in module.contents.all <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">data-id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> content.id <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
          <span>{</span><span>%</span> with item=content.item <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span>{</span><span>{</span> item <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"#"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_delete"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">content.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
              <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Delete"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
              <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
          <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
      <span>{</span><span>%</span> empty <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>This module has no contents yet.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
      <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
      <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">hr</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Add new content:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"content-types"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_create"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">text</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Text<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_create"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">image</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Image<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_create"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">video</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Video<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_create"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">file</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>File<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span> 
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endwith <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>   </code></pre></div>
<p>这个模板展示一个课程所有的模块以及被选中的模块的内容。我们迭代课程模块并将它们展示在侧边栏。我们还迭代模块的内容并且通过<code>content.item</code>去获取关联的<em>Text</em>，<em>Video</em>，<em>Image</em>以及<em>File</em>对象。我们还包含可以创建新的文本，视频，图片以及文件内容的链接。</p>
<p>我们想要知道每个<em>item</em>对象是哪种类型（文本，视频，图片或者文件）的对象。我们需要模型名用来构建URL去编辑对象。除此以外，我们还在模板中展示各式各样的不同的<em>item</em>，基于<em>item</em>的内容类型。我们可以通过访问对象的<code>_meta</code>属性来从模型的<code>Meta</code>类中获取一个对象的模型。尽管如此，Django不允许访问开头是下划线的变量或者属性在模板中为了编辑检索私有属性或者调用到私有方法。我们可以通过编写一个定制模板过滤器来解决这个问题。</p>
<p>在<em>courses</em>应用目录下创建以下文件结构：</p>
<pre class="shell"><code class="hljs">templatetags/
    __init__.py
    course.py</code></pre>
<p>编辑<em>course.py</em>模块，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> template

register <span class="op">=</span> template.Library()

<span class="at"><span class="hljs-meta">@register.filter</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">model_name</span><span class="hljs-params">(obj)</span>:</span>
    <span class="cf"><span class="hljs-keyword">try</span></span>:
        <span class="cf"><span class="hljs-keyword">return</span></span> obj._meta.model_name
    <span class="cf"><span class="hljs-keyword">except</span></span> <span class="pp">AttributeError</span>:
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va"><span class="hljs-keyword">None</span></span></code></pre></div>
<p>以上就是<code>model_name</code>模板过滤器。我们可以在模板中通过<code>object|model_name</code>应用它来给一个对象获取模型的名字。</p>
<p>编辑<em>templates/courses/manage/module/content_list.html</em>模板，在<code><span>{</span><span>%</span> extends <span>%</span><span>}</span></code>模板标签下添加以下内容：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> course <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这样将会加载<em>course</em>模板标签。之后，将以下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span>{</span><span>{</span> item <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"#"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span></code></pre></div>
<p>替换成：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span>{</span><span>{</span> item <span>}</span><span>}</span> (<span>{</span><span>{</span> item|model_name <span>}</span><span>}</span>)<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_update"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module.id</span> <span class="hljs-attr">item</span></span></span><span class="er"><span class="hljs-tag">|<span class="hljs-attr">model_name</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">item.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span></code></pre></div>
<p>现在，我们在模板中展示item模型并且使用模型名曲构建编辑对象的链接。编辑<em>courses/manage/course/list.html</em>模板，添加一个链接给<code>module_content_list</code> URL，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_module_update"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Edit modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span>{</span><span>%</span> if course.modules.count &gt; 0 <span>%</span><span>}</span>
 <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">module_content_list"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.modules.first.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Manage contents<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span></code></pre></div>
<p>这个新链接允许用户去访问课程的第一个模块的内容，如果有好多内容的话。</p>
<p>打开 <a href="http://127.0.0.1:8000/course/mine/" class="uri">http://127.0.0.1:8000/course/mine/</a> 然后点击一个包含最新模块的课程的<strong>Manage contents</strong>链接。你会看到如下页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-11.png" alt="django-10-11"></p>
<p>当你点击左方侧边栏的一个模块上，它的内容会在主区域显示。这个模板还包含用来给展示的模块添加一个新的文本，视频，图片或者文件内容。给这个模块添加一堆不同的内容然后看下结果。这个内容将会出现在<strong>Module contents</strong>之后，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-12.png" alt="django-10-12"></p>
<h2 id="重新整理模块和内容">重新整理模块和内容</h2>
<p>我们需要提供一个简单的放来去重新排序课程模板和它们的内容。我们将要使用一个JavaScript drag-n-drop 控件去让我们的用户通过拖拽课程的模块来对课程模块进行重新排序。当用户完成拖拽一个模块，我们将会执行一个异步请求（AJAX）去存储新的模块顺序。</p>
<p>我们需要一个视图，该视图通过编译在JSON中的模块的id来检索新的对象。编辑<em>courses</em>应用的<em>views.py</em>文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> braces.views <span class="im"><span class="hljs-keyword">import</span></span> CsrfExemptMixin, JsonRequestResponseMixin

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ModuleOrderView</span><span class="hljs-params">(CsrfExemptMixin,
                         JsonRequestResponseMixin,
                         View)</span>:</span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request)</span>:</span>
        <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span>, order <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.request_json.items():
            Module.objects.<span class="bu">filter</span>(<span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                course__owner<span class="op">=</span>request.user).update(order<span class="op">=</span>order)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_json_response({<span class="st"><span class="hljs-string">'saved'</span></span>: <span class="st"><span class="hljs-string">'OK'</span></span>})</code></pre></div>
<p>以上是<code>ModuleOrderView</code>。我们使用以下django-braces的mixins：</p>
<ul>
<li>csrfExemptMixin：用来避免在POST请求中检查一个CSRF token。</li>
<li>JsonRequestResponseMixin：将请求的数据分析为JSON并且将相应也序列化成JSON并且返回一个<code>application/json</code>内容类型的HTTP响应。</li>
</ul>
<p>我们可以构建一个类似的视图去排序一个模块的内容。添加以下代码到<em>views.py</em>中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ContentOrderView</span><span class="hljs-params">(CsrfExemptMixin,
                          JsonRequestResponseMixin,
                          View)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request)</span>:</span>
        <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span>, order <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.request_json.items():
            Content.objects.<span class="bu">filter</span>(<span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                          module__course__owner<span class="op">=</span>request.user) <span class="op">\</span>
                          .update(order<span class="op">=</span>order)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_json_response({<span class="st"><span class="hljs-string">'saved'</span></span>: <span class="st"><span class="hljs-string">'OK'</span></span>})</code></pre></div>
<p>现在，编辑<em>courses</em>应用的<em>urls.py</em>文件，添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^module/order/$'</span></span>,
       views.ModuleOrderView.as_view(),
       name<span class="op">=</span><span class="st"><span class="hljs-string">'module_order'</span></span>),
url(<span class="vs"><span class="hljs-string">r'^content/order/$'</span></span>,
       views.ContentOrderView.as_view(),
       name<span class="op">=</span><span class="st"><span class="hljs-string">'content_order'</span></span>),</code></pre></div>
<p>最后，我们在模板中导入drag-n-drop功能。我们将要使用jQuery UI库来使用这个功能。jQuery UI基于jQuery构建并且它提供了一组界面交互，效果和小部件。我们将要使用它的<em>sortable</em>元素。首先，我们需要在基础模板中加载jQuery UI。打开<em>courses</em>应用下的<em>templates/</em>目录下的<em>base.html</em>文件，在加载jQuery的下方添加jQuery UI脚本，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">script</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/jquery.min.js"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">script</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"https://ajax.googleapis.com/ajax/libs/jqueryui/1.11.4/jquery-ui.min.js"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span></code></pre></div>
<p><strong>（译者注：要用以上地址，记得FQ。。。。。或者自己直接下载）</strong></p>
<p>我们在jQuery框架下加载jQuery UI。现在，编辑<em>courses/manage/module/content_list.html</em>模板添加以下代码在模板的底部：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> block domready <span>%</span><span>}</span>
   $('#modules').sortable({
       stop: function(event, ui) {
           modules_order = {};
           $('#modules').children().each(function(){
               // update the order field
               $(this).find('.order').text($(this).index() + 1);
               // associate the module's id with its order
               modules_order[$(this).data('id')] = $(this).index();
               });
               $.ajax({
                type: 'POST',
                url: '<span>{</span><span>%</span> url "module_order" <span>%</span><span>}</span>',
                contentType: 'application/json; charset=utf-8',
                dataType: 'json',
                data: JSON.stringify(modules_order)
                });
    }
});

$('#module-contents').sortable({
    stop: function(event, ui) {
        contents_order = {};
        $('#module-contents').children().each(function(){
            // associate the module's id with its order
            contents_order[$(this).data('id')] = $(this).index();
           });
        $.ajax({
               type: 'POST',
               url: '<span>{</span><span>%</span> url "content_order" <span>%</span><span>}</span>',
               contentType: 'application/json; charset=utf-8',
               dataType: 'json',
               data: JSON.stringify(contents_order),
        }); 
      }
   });
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>  </code></pre></div>
<p>这个JavaScripy代码在<code><span>{</span><span>%</span> block domready <span>%</span><span>}</span></code>区块中，因此它会被包含在我们之前定义在<em>base.html</em>模板中的jQuery的<code>$(document).ready()</code>事件中。这将保证我们的JavaScripy代码会在页面每次加载的时候都会被执行一次。我们给列在侧边栏的模块定义了一个<em>sortable</em>元素并且给模块内容列也定义了一个不同的。这两者有着相似的方式。在以上代码中，我们执行以下任务：</p>
<ul>
<li>1 首先，我们给<em>modules</em> HTML元素定义了一个<em>sortable</em>元素。请记住，我们使用<code>#moudles</code>，因为jQuery给选择器使用CSS符号。</li>
<li>2 我们给<em>stop</em>事件指定一个函数。这个时间会在用户每次储存一个元素的时候被触发。</li>
<li>3 我们创建一个空的<em>modules_orders</em>目录。给这个目录的键将会是模块的id，并且给每个模块的值都会被分配次序。</li>
<li>4 我们迭代<code>#module</code>子元素。我们给每个模块重新计算展示次序并且拿到每个模块的<em>data-id</em>属性，该属性包含了模块的id。我们给<em>modules_order</em>目录添加id作为一个键并且模型的新的索引作为值。</li>
<li>5 我们运行一个AJAX POST请求给<em>content_order</em> URL，在请求中包含<em>modules_orders</em>的序列化的JSON数据。相应的<code>ModuleOrderView</code>会负责更新模块的顺序。</li>
</ul>
<p><em>sortable</em>元素排列内容非常类似与上者的方法。回到你的浏览器然后重载页面。现在你将可以点击并且拖动模块和内容，去重新排序它们如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-10-13.png" alt="django-10-13"></p>
<p>很好！现在你可以重新排序课程模块和模块内容了。</p>
<h2 id="总结">总结</h2>
<p>在这章中，你学习了如何创建一个多功能的内容管理系统。你使用了模型继承以及创建了一个定制模型字段。你还通过基于类的视图和mixins工作。你创建了formsets以及一个系统去管理不同类型的内容。</p>
<p>在下一章，你将会创建一个学生注册系统。你还将熏染不同类型的内容，并且你还会学习如何使用Django的缓存框架。</p>
</div>