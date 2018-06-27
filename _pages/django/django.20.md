---
layout: single
permalink: /django/example11/
title: "Django By Example 第十一章"
sidebar:
  nav: "django"
# toc: true
---

<div><h1 id="第十一章"><strong>第十一章</strong></h1>
<h1 id="缓存内容"><strong>缓存内容</strong></h1>
<p>在上一章中，你使用模型继承和一般关系创建了一个灵活的课程内容模型。你也使用基于类的视图，表单集，以及 内容的 AJAX 排序，创建了一个课程管理系统。在这一章中，你将会：</p>
<ul>
<li>创建展示课程信息的公共视图</li>
<li>创建一个学生注册系统</li>
<li>在<em>courses</em>中管理学生注册</li>
<li>创建多样化的课程内容</li>
<li>使用缓存框架缓存内容</li>
</ul>
<p>我们将会从创建一个课程目录开始，好让学生能够浏览当前的课程以及注册这些课程。</p>
<h2 id="展示课程"><strong>展示课程</strong></h2>
<p>对于课程目录，我们需要创建以下的功能：</p>
<ul>
<li>列出所有的可用课程，可以通过可选科目过滤</li>
<li>展示一个单独的课程概览</li>
</ul>
<p>编辑 courses 应用的 <code>views.py</code> ，添加以下代码：</p>
<pre class="python`"><code class="hljs python"><span class="hljs-keyword">from</span> django.db.models <span class="hljs-keyword">import</span> Count
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Subject

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">CourseListView</span><span class="hljs-params">(TemplateResponseMixin, View)</span>:</span>
    model = Course
    template_name = <span class="hljs-string">'courses/course/list.html'</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get</span><span class="hljs-params">(self, request, subject=None)</span>:</span>
        subjects = Subject.objects.annotate(
                      total_courses=Count(<span class="hljs-string">'courses'</span>))
        courses = Course.objects.annotate(
                      total_modules=Count(<span class="hljs-string">'modules'</span>))
        <span class="hljs-keyword">if</span> subject:
            subject = get_object_or_404(Subject, slug=subject)
            courses = courses.filter(subject=subject)
       <span class="hljs-keyword">return</span> self.render_to_response({<span class="hljs-string">'subjects'</span>:subjects,
                                               <span class="hljs-string">'subject'</span>: subject,
                                               <span class="hljs-string">'courses'</span>: courses})</code></pre>
<p>这是 <code>CourseListView</code> 。它继承了 <code>TemplateResponseMixin</code> 和 <code>View</code> 。在这个视图中，我们实现了下面的功能：</p>
<ul>
<li>1 我们检索所有的课程，包括它们当中的每个课程总数。我们使用 ORM 的 <code>annotate()</code> 方法和 <code>Count()</code> 聚合方法来实现这一功能。</li>
<li>2 我们检索所有的可用课程，包括在每个课程中包含的模块总数。</li>
<li>3 如果给了科目的 slug URL 参数，我们就检索对应的课程对象，然后我们将会把查询限制在所给的科目之内。</li>
<li>4 我们使用 <code>TemplateResponseMixin</code> 提供的 <code>render_to_response()</code> 方法来把对象渲染到模板中，然后返回一个 HTTP 响应。</li>
</ul>
<p>让我们创建一个详情视图来展示单一课程的概览。在 <code>views.py</code> 中添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.detail <span class="im"><span class="hljs-keyword">import</span></span> DetailView

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseDetailView</span><span class="hljs-params">(DetailView)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/course/detail.html'</span></span></code></pre></div>
<p>这个视图继承了 Django 提供的通用视图 <code>DetailView</code> 。我们定义了 <code>model</code> 和 <code>template_name</code> 属性。Django 的 <code>DetailView</code> 期望一个 <strong>主键(pk)</strong> 或者 <strong>slug</strong> URL 参数来检索对应模型的一个单一对象。然后它就会渲染 <code>template_name</code> 中的模板，同样也会把上下文中的对象渲染进去。</p>
<p>编辑 <code>educa</code> 项目中的主 <code>urls.py</code> 文件，添加以下 URL 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> courses.views <span class="im"><span class="hljs-keyword">import</span></span> CourseListView

urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># ...</span></span>
    url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, CourseListView.as_view(), name<span class="op">=</span><span class="st"><span class="hljs-string">'course_list'</span></span>),
]</code></pre></div>
<p>我们把 <code>course_list</code> 的 URL 模式添加进了项目的主 <code>urls.py</code> 中，因为我们想要把课程列表展示在 <code>http://127.0.0.1:8000/</code> 中，然后 <code>courses</code> 应用的所有的其他 URL 都有 <code>/course/</code> 前缀。</p>
<p>编辑 <code>courses</code> 应用的 <code>urls.py</code> ，添加以下 URL 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^subject/(?P&lt;subject&gt;[\w-]+)/$'</span></span>,
    views.CourseListView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'course_list_subject'</span></span>),

url(<span class="vs"><span class="hljs-string">r'^(?P&lt;slug&gt;[\w-]+)/$'</span></span>,
    views.CourseDetailView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'course_detail'</span></span>),</code></pre></div>
<p>我们定义了以下 URL 模式：</p>
<ul>
<li><code>course_list_subject</code>：用于展示一个科目的所有课程</li>
<li><code>course_detail</code>：用于展示一个课程的概览</li>
</ul>
<p>让我们为 <code>CourseListView</code> 和 <code>CourseDetailView</code> 创建模板。在 <code>courses</code> 应用的 <code>templates/courses/</code> 路径下创建以下路径：</p>
<ul>
<li>course/</li>
<li>list.html</li>
<li>detail.html</li>
</ul>
<p>编辑 <code>courses/course/list.html</code> 模板，写入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    <span>{</span><span>%</span> if subject <span>%</span><span>}</span>
        <span>{</span><span>{</span> subject.title <span>}</span><span>}</span> courses
    <span>{</span><span>%</span> else <span>%</span><span>}</span>
        All courses
    <span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
    <span>{</span><span>%</span> if subject <span>%</span><span>}</span>
        <span>{</span><span>{</span> subject.title <span>}</span><span>}</span> courses
    <span>{</span><span>%</span> else <span>%</span><span>}</span>
        All courses
    <span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"contents"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Subjects<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"modules"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">not</span> <span class="hljs-attr">subject</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span class="hljs-attr">class</span></span></span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_list"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>All<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
        <span>{</span><span>%</span> for s in subjects <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">subject</span></span></span><span class="hljs-tag"> </span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">=</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">s</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span class="hljs-attr">class</span></span></span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_list_subject"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">s.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                    <span>{</span><span>{</span> s.title <span>}</span><span>}</span>
                    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span><span>{</span><span>{</span> s.total_courses <span>}</span><span>}</span> courses<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>%</span> for course in courses <span>%</span><span>}</span>
        <span>{</span><span>%</span> with subject=course.subject <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_detail"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> course.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_list_subject"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">subject.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> subject <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>.
                <span>{</span><span>{</span> course.total_modules <span>}</span><span>}</span> modules.
                Instructor: <span>{</span><span>{</span> course.owner.get_full_name <span>}</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
        <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
    <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板用于展示可用课程列表。我们创建了一个 HTML 列表来展示所有的 <code>Subject</code> 对象，然后为它们每一个都创建了一个链接，这个链接链接到 <code>course_list_subject</code> 的 URL 。 如果存在当前科目，我们就把 <code>selected</code> HTML 类添加到当前科目中高亮显示该科目。我们迭代每个 <code>Course</code> 对象，展示模块的总数和教师的名字。</p>
<p>使用 <code>python manage.py runserver</code> 打开代发服务器，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> 。你看到的应该是像下面这个样子：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-1.png" alt="django-11-1"></p>
<p>左边的侧边栏包含了所有的科目，包括每个科目的课程总数。你可以点击任意一个科目来筛选展示的课程。</p>
<p>编辑 <code>courses/course/detail.html</code> 模板，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    <span>{</span><span>{</span> object.title <span>}</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span>{</span><span>%</span> with subject=course.subject <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
            <span>{</span><span>{</span> object.title <span>}</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Overview<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_list_subject"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">subject.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> subject.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>.
                <span>{</span><span>{</span> course.modules.count <span>}</span><span>}</span> modules.
                Instructor: <span>{</span><span>{</span> course.owner.get_full_name <span>}</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
            <span>{</span><span>{</span> object.overview|linebreaks <span>}</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>在这个模板中，我们展示了每个单一课程的概览和详情。访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，点击任意一个课程。你就应该看到有下面结构的页面了：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-2.png" alt="django-11-2"></p>
<p>我们已经创建了一个展示课程的公共区域了。下面，我们需要让用户可以注册为学生以及注册他们的课程。</p>
<h2 id="添加学生注册"><strong>添加学生注册</strong></h2>
<p>使用下面的命令创键一个新的应用：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">students</span></code></pre>
<p>编辑 <code>educa</code> 项目的 <code>settings.py</code> ，把 <code>students</code> 添加进 <code>INSTALLED_APPS</code> 设置中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'students'</span></span>,
)</code></pre></div>
<h2 id="创建一个学生注册视图"><strong>创建一个学生注册视图</strong></h2>
<p>编辑 <code>students</code> 应用的 <code>views.py</code> ，写入下下面的代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse_lazy
<span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.edit <span class="im"><span class="hljs-keyword">import</span></span> CreateView
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.auth.forms <span class="im"><span class="hljs-keyword">import</span></span> UserCreationForm
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib.auth <span class="im"><span class="hljs-keyword">import</span></span> authenticate, login

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">StudentRegistrationView</span><span class="hljs-params">(CreateView)</span>:</span>
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'students/student/registration.html'</span></span>
    form_class <span class="op">=</span> UserCreationForm
    success_url <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'student_course_list'</span></span>)
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">form_valid</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, form)</span>:</span>
        result <span class="op">=</span> <span class="bu">super</span>(StudentRegistrationView,
                        <span class="va">self</span>).form_valid(form)
        cd <span class="op">=</span> form.cleaned_data
        user <span class="op">=</span> authenticate(username<span class="op">=</span>cd[<span class="st"><span class="hljs-string">'username'</span></span>],
                            password<span class="op">=</span>cd[<span class="st"><span class="hljs-string">'password1'</span></span>])
        login(<span class="va">self</span>.request, user)
        <span class="cf"><span class="hljs-keyword">return</span></span> result</code></pre></div>
<p>这个视图让学生可以注册进我们的网站里。我们使用了可以提供创建模型对象功能的通用视图 <code>CreateView</code> 。这个视图要求以下属性：</p>
<ul>
<li><code>template_name</code>：渲染这个视图的模板路径。</li>
<li><code>form_class</code>：用于创建对象的表单，我们使用 Django 的 <code>UserCreationForm</code> 作为注册表单来创建 <code>User</code> 对象。</li>
<li><code>success_url</code>：当表单成功提交时要将用户重定向到的 URL 。我们逆序了 <code>student_course_list</code> URL，我们稍候将会将建它来展示学生已报名的课程。</li>
</ul>
<p>当合法的表单数据被提交时 <code>form_valid()</code> 方法就会执行。它必须返回一个 HTTP 响应。我们覆写了这个方法来让用户在成功注册之后登录。</p>
<p>在 students 应用路径下创建一个新的文件，命名为 <code>urls.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^register/$'</span></span>,
           views.StudentRegistrationView.as_view(),
           name<span class="op">=</span><span class="st"><span class="hljs-string">'student_registration'</span></span>),
]</code></pre></div>
<p>编辑 <code>educa</code> 的主 <code>urls.py</code> ，然后把 <code>students</code> 应用的 URLs 引入进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^students/'</span></span>, include(<span class="st"><span class="hljs-string">'students.urls'</span></span>)),</code></pre></div>
<p>在 <code>students</code> 应用内创建如下的文件结构:</p>
<pre><code class="hljs">templates/
    students/
        student/
            registration.html</code></pre>
<p>编辑 <code>students/student/registration.html</code> 模板，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    Sign up
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
        Sign up
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Enter your details to create an account:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">""</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
            <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Create my account"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>最后编辑 <code>educa</code> 的设置文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse_lazy
LOGIN_REDIRECT_URL <span class="op">=</span> reverse_lazy(<span class="st"><span class="hljs-string">'student_course_list'</span></span>)</code></pre></div>
<p>这个是由 <code>auth</code> 模型用来给用户在成功的登录之后重定向的设置，如果请求中没有 next 参数的话。</p>
<p>打开开发服务器，访问 <a href="http://127.0.0.1:8000/students/register/" class="uri">http://127.0.0.1:8000/students/register/</a> ，你可以看到像这样的注册表单：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-3.png" alt="django-11-3"></p>
<h2 id="报名"><strong>报名</strong></h2>
<p>在用户创建一个账号之后，他们应该就可以在 <code>courses</code> 中报名了。为了保存报名表，我们需要在 <code>Course</code> 和 <code>User</code> 模型之间创建一个多对多关系。编辑 <code>courses</code> 应用的 <code>models.py</code> 然后把下面的字段添加进 <code>Course</code> 模型中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">students <span class="op">=</span> models.ManyToManyField(User,
                        related_name<span class="op">=</span><span class="st"><span class="hljs-string">'courses_joined'</span></span>,
                        blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>在 shell 中执行下面的命令来创建迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你可以看到类似下面的输出：</p>
<pre><code class="hljs delphi">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'courses'</span>:
    <span class="hljs-number">0004</span>_course_students.py:
       - Add field students <span class="hljs-keyword">to</span> course</code></pre>
<p>接下来执行下面的命令来应用迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你可以看到以下输出：</p>
<pre><code class="hljs groovy">Operations to <span class="hljs-string">perform:</span>
    Apply all <span class="hljs-string">migrations:</span> courses
Running <span class="hljs-string">migrations:</span>
    Rendering model states... DONE
    Applying courses<span class="hljs-number">.0004</span>_course_students... OK</code></pre>
<p>我们现在就可以把学生和他们报名的课程相关联起来了。<br>
让我们创建学生报名课程的功能吧。</p>
<p>在 <code>students</code> 应用内创建一个新的文件，命名为 <code>forms.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> courses.models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseEnrollForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    course <span class="op">=</span> forms.ModelChoiceField(queryset<span class="op">=</span>Course.objects.<span class="bu">all</span>(),
                                    widget<span class="op">=</span>forms.HiddenInput)</code></pre></div>
<p>我们将会把这张表用于学生报名。<code>course</code> 字段是学生报名的课程。所以，它是一个 <code>ModelChoiceField</code> 。我们使用 <code>HiddenInput</code> 控件，因为我们不打算把这个字段展示给用户。我们将会在 <code>CourseDetailView</code> 视图中使用这个表单来展示一个报名按钮。</p>
<p>编辑 <code>students</code> 应用的 <code>views.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.edit <span class="im"><span class="hljs-keyword">import</span></span> FormView
<span class="im"><span class="hljs-keyword">from</span></span> braces.views <span class="im"><span class="hljs-keyword">import</span></span> LoginRequiredMixin
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> CourseEnrollForm

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">StudentEnrollCourseView</span><span class="hljs-params">(LoginRequiredMixin, FormView)</span>:</span>
    course <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    form_class <span class="op">=</span> CourseEnrollForm
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">form_valid</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, form)</span>:</span>
        <span class="va">self</span>.course <span class="op">=</span> form.cleaned_data[<span class="st"><span class="hljs-string">'course'</span></span>]
        <span class="va">self</span>.course.students.add(<span class="va">self</span>.request.user)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">super</span>(StudentEnrollCourseView,
                        <span class="va">self</span>).form_valid(form)

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_success_url</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> reverse_lazy(<span class="st"><span class="hljs-string">'student_course_detail'</span></span>,
                                args<span class="op">=</span>[<span class="va">self</span>.course.<span class="bu">id</span>])</code></pre></div>
<p>这就是 <code>StudentEnrollCourseView</code> 。它负责学生在 <code>courses</code> 中报名。新的视图继承了 <code>LoginRequiredMixin</code> ，所以只有登录了的用户才可以访问到这个视图。我们把 <code>CourseEnrollForm</code>表单用在了 <code>form_class</code> 属性上，同时我们也定义了一个 <code>course</code> 属性来储存所给的 <code>Course</code> 对象。当表单合法时，我们把当前用户添加到课程中已报名学生中去。</p>
<p>如果表单提交成功，<code>get_success_url</code> 方法就会返回用户将会被重定向到的 URL 。这个方法相当于 <code>success_url</code> 属性。我们反序 <code>student_course_detail</code> URL ,我们稍候将会创建它来展示课程中的学生。</p>
<p>编辑 <code>students</code> 应用的 <code>urls.py</code> ，添加以下 URL 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^enroll-course/$'</span></span>,
    views.StudentEnrollCourseView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_enroll_course'</span></span>),</code></pre></div>
<p>让我们把报名按钮表添加进课程概览页。编辑 <code>course</code> 应用的 <code>views.py</code> ，然后修改 <code>CourseDetailView</code> 让它看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> students.forms <span class="im"><span class="hljs-keyword">import</span></span> CourseEnrollForm

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseDetailView</span><span class="hljs-params">(DetailView)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/course/detail.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_context_data</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        context <span class="op">=</span> <span class="bu">super</span>(CourseDetailView,
                        <span class="va">self</span>).get_context_data(<span class="op">**</span>kwargs)
        context[<span class="st"><span class="hljs-string">'enroll_form'</span></span>] <span class="op">=</span> CourseEnrollForm(
                        initial<span class="op">=</span>{<span class="st"><span class="hljs-string">'course'</span></span>:<span class="va">self</span>.<span class="bu">object</span>})
        <span class="cf"><span class="hljs-keyword">return</span></span> context</code></pre></div>
<p>我们使用 <code>get_context_data()</code> 方法来在渲染进模板中的上下文里引入报名表。我们初始化带有当前 <code>Course</code> 对象的表单的隐藏 course 字段，这样它就可以被直接提交了。</p>
<p>编辑 <code>courses/course/detail.html</code> 模板，然后找到下面的这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>{</span> object.overview|linebreaks <span>}</span><span>}</span></code></pre></div>
<p>起始行应该被替换为下面的这几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>{</span> object.overview|linebreaks <span>}</span><span>}</span>
<span>{</span><span>%</span> if request.user.is_authenticated <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">student_enroll_course"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span>{</span><span>{</span> enroll_form <span>}</span><span>}</span>
        <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Enroll now"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
<span>{</span><span>%</span> else <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">student_registration"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        Register to enroll
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span></code></pre></div>
<p>这个按钮就是用于报名的。如果用户是被认证过的，我们就展示包含了隐藏表单字段的报名按钮，这个表单指向了 <code>student_enroll_course</code> URL。如果用户没有被认证，我们将会展示一个注册链接。</p>
<p>确保已经打开了开发服务器，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，然后点击一个课程。如果你登录了，你就可以在底部看到 <strong>ENROLL NOW</strong> 按钮，就像这样：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-4.png" alt="django-11-4"></p>
<p>如果你没有登录，你就会看到一个<strong>Register to enroll</strong> 的按钮。</p>
<h2 id="获取课程内容"><strong>获取课程内容</strong></h2>
<p>我们需要一个视图来展示学生已经报名的课程，和一个获取当前课程内容的视图。编辑 <code>students</code> 应用的 <code>views.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.<span class="bu">list</span> <span class="im"><span class="hljs-keyword">import</span></span> ListView
<span class="im"><span class="hljs-keyword">from</span></span> courses.models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">StudentCourseListView</span><span class="hljs-params">(LoginRequiredMixin, ListView)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'students/course/list.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_queryset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        qs <span class="op">=</span> <span class="bu">super</span>(StudentCourseListView, <span class="va">self</span>).get_queryset()
        <span class="cf"><span class="hljs-keyword">return</span></span> qs.<span class="bu">filter</span>(students__in<span class="op">=</span>[<span class="va">self</span>.request.user])</code></pre></div>
<p>这个是用于列出学生已经报名课程的视图。它继承 <code>LoginRequiredMixin</code> 来确保只有登录的用户才可以连接到这个视图。同时它也继承了通用视图 <code>ListView</code> 来展示 <code>Course</code> 对象列表。我们覆写了 <code>get_queryset()</code> 方法来检索用户已经报名的课程。我们通过学生的 <code>ManyToManyField</code> 字段来筛选查询集以达到这个目的。</p>
<p>把下面的代码添加进 <code>views.py</code> 文件中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.generic.detail <span class="im"><span class="hljs-keyword">import</span></span> DetailView

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">StudentCourseDetailView</span><span class="hljs-params">(DetailView)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'students/course/detail.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_queryset</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        qs <span class="op">=</span> <span class="bu">super</span>(StudentCourseDetailView, <span class="va">self</span>).get_queryset()
        <span class="cf"><span class="hljs-keyword">return</span></span> qs.<span class="bu">filter</span>(students__in<span class="op">=</span>[<span class="va">self</span>.request.user])
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_context_data</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        context <span class="op">=</span> <span class="bu">super</span>(StudentCourseDetailView,
                        <span class="va">self</span>).get_context_data(<span class="op">**</span>kwargs)
        <span class="co"><span class="hljs-comment"># get course object</span></span>
        course <span class="op">=</span> <span class="va">self</span>.get_object()
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="st"><span class="hljs-string">'module_id'</span></span> <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.kwargs:
            <span class="co"><span class="hljs-comment"># get current module</span></span>
            context[<span class="st"><span class="hljs-string">'module'</span></span>] <span class="op">=</span> course.modules.get(
                            <span class="bu">id</span><span class="op">=</span><span class="va">self</span>.kwargs[<span class="st"><span class="hljs-string">'module_id'</span></span>])
        <span class="cf"><span class="hljs-keyword">else</span></span>:
            <span class="co"><span class="hljs-comment"># get first module</span></span>
            context[<span class="st"><span class="hljs-string">'module'</span></span>] <span class="op">=</span> course.modules.<span class="bu">all</span>()[<span class="dv"><span class="hljs-number">0</span></span>]
        <span class="cf"><span class="hljs-keyword">return</span></span> context</code></pre></div>
<p>这是 <code>StudentCourseDetailView</code> 。我们覆写了 <code>get_queryset</code> 方法把查询集限制在用户报名的课程之内。我们同样也覆写了 <code>get_context_data()</code> 方法来把课程的一个模块赋值在上下文内，如果给了 <code>model_id</code> URL 参数的话。否则，我们就赋值课程的第一个模块。这样，学生就可以在课程之内浏览各个模块了。</p>
<p>编辑 <code>students</code> 应用的 <code>urls.py</code> ，添加以下 URL 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^courses/$'</span></span>,
    views.StudentCourseListView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_course_list'</span></span>),

url(<span class="vs"><span class="hljs-string">r'^course/(?P&lt;pk&gt;\d+)/$'</span></span>,
    views.StudentCourseDetailView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_course_detail'</span></span>),

url(<span class="vs"><span class="hljs-string">r'^course/(?P&lt;pk&gt;\d+)/(?P&lt;module_id&gt;\d+)/$'</span></span>,
    views.StudentCourseDetailView.as_view(),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_course_detail_module'</span></span>),</code></pre></div>
<p>在 <code>students</code> 应用的 <code>templates/students/</code> 路径下创建以下文件结构：</p>
<pre><code class="hljs cpp">course/
    detail.html
    <span class="hljs-built_in">list</span>.html</code></pre>
<p>编辑 <code>students/course/list.html</code> 模板，然后添加下列代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>My courses<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>My courses<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>

    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span>{</span><span>%</span> for course in object_list <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"course-info"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span><span>{</span><span>{</span> course.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">student_course_detail"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">course.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Access contents<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
        <span>{</span><span>%</span> empty <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
                You are not enrolled in any courses yet.
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">course_list"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Browse courses<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>to enroll in a course.
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板展示了用户报名的课程。编辑 <code>students/course/detail.html</code> 模板，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    <span>{</span><span>{</span> object.title <span>}</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>
        <span>{</span><span>{</span> module.title <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"contents"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Modules<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"modules"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span>{</span><span>%</span> for m in object.modules.all <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">data-id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> m.id <span>}</span><span>}</span>"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">m</span></span></span><span class="hljs-tag"> </span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">=</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">module</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="hljs-tag">
</span><span class="ot"><span class="hljs-tag"><span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">student_course_detail_module"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">object.id</span> <span class="hljs-attr">m.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
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
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span>{</span><span>%</span> for content in module.contents.all <span>%</span><span>}</span>
            <span>{</span><span>%</span> with item=content.item <span>%</span><span>}</span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span><span>{</span><span>{</span> item.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
                <span>{</span><span>{</span> item.render <span>}</span><span>}</span>
            <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板用于报名了的学生连接到课程内容。首先我们创建了一个包含所有课程模块的 HTML 列表且高亮当前模块。然后我们迭代当前的模块内容，之后使用 <code><span>{</span><span>{</span> item.render <span>}</span><span>}</span></code> 来连接展示内容。接下来我们将会在内容模型中添加 <code>render()</code> 方法。这个方法将会负责精准的展示内容。</p>
<h2 id="渲染不同类型的内容"><strong>渲染不同类型的内容</strong></h2>
<p>我们需要提供一个方法来渲染不同类型的内容。编辑 <code>course</code> 应用的 <code>models.py</code> ，把 <code>render()</code> 方法添加进 <code>ItemBase</code> 模型中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.template.loader <span class="im"><span class="hljs-keyword">import</span></span> render_to_string
<span class="im"><span class="hljs-keyword">from</span></span> django.utils.safestring <span class="im"><span class="hljs-keyword">import</span></span> mark_safe

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ItemBase</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">render</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> render_to_string(<span class="st"><span class="hljs-string">'courses/content/{}.html'</span></span>.<span class="bu">format</span>(
                            <span class="va">self</span>._meta.model_name), {<span class="st"><span class="hljs-string">'item'</span></span>: <span class="va">self</span>})</code></pre></div>
<p>这个方法使用了 <code>render_to_string()</code> 方法来渲染模板以及返回一个作为字符串的渲染内容。每种内容都使用以内容模型命名的模板渲染。我们使用 <code>self._meta.model_name</code> 来为 <code>la</code> 创建合适的模板名。 <code>render()</code> 方法提供了一个渲染不同页面的通用接口。</p>
<p>在 <code>courses</code> 应用的 <code>templates/courses/</code> 路径下创建如下文件结构：</p>
<pre><code class="hljs delphi">content/
    text.html
    <span class="hljs-keyword">file</span>.html
    image.html
    video.html</code></pre>
<p>编辑 <code>courses/content/text.html</code> 模板，写入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>{</span> item.content|linebreaks|safe <span>}</span><span>}</span></code></pre></div>
<p>编辑 <code>courses/content/file.html</code> 模板，写入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> item.file.url <span>}</span><span>}</span>"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Download file<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>编辑 <code>courses/content/image.html</code> 模板，写入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">img</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> item.file.url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>为了使上传带有 <code>ImageField</code> 和 <code>FielField</code> 的文件工作，我们需要配置我们的项目以使用开发服务器提供媒体文件服务。编辑你的项目中的 <code>settings.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">MEDIA_URL <span class="op">=</span> <span class="st"><span class="hljs-string">'/media/'</span></span>
MEDIA_ROOT <span class="op">=</span> os.path.join(BASE_DIR, <span class="st"><span class="hljs-string">'media/'</span></span>)</code></pre></div>
<p>记住 <code>MEDIA_URL</code> 是服务上传文件的基本 URL 路径， <code>MEDIA_ROOT</code> 是放置文件的本地路径。</p>
<p>编辑你的项目的主 <code>urls.py</code> ，添加以下 imports：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls.static <span class="im"><span class="hljs-keyword">import</span></span> static</code></pre></div>
<p>然后，把下面这几行写入文件的结尾：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">+=</span> static(settings.MEDIA_URL,
                document_root<span class="op">=</span>settings.MEDIA_ROOT)</code></pre></div>
<p>你的项目现在已经准备好使用开发服务器上传和服务文件了。记住开发服务器不能用于生产环境中。我们将会在下一章中学习如何配置生产环境。</p>
<p>我们也需要创建一个模板来渲染 <code>Video</code> 对象。我们将会使用 django-embed-video 来嵌入视频内容。 Django-embed-video 是一个第三方 Django 应用，它使你可以通过提供一个视频的公共 URL 来在模板中嵌入视频，类似来自 YouTube 或者 Vimeo 的资源。</p>
<p>使用下面的命令来安装这个包：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">pip</span> isntall django-embed-video==<span class="hljs-number">1</span>.<span class="hljs-number">0</span>.<span class="hljs-number">0</span></code></pre>
<p>然后编辑项目的 <code>settings.py</code> 然后添加 <code>embed_video</code> 到 <code>INSTALLED_APPS</code>设置 中。你可以在这里找到 django-embed-video 的文档：<br>
<a href="http://django-embed-video.readthedocs.org/en/v1.0.0/" class="uri">http://django-embed-video.readthedocs.org/en/v1.0.0/</a> 。</p>
<p>编辑 <code>courses/content/video.html</code> 模板，写入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load embed_video_tags <span>%</span><span>}</span>
<span>{</span><span>%</span> video item.url 'small' <span>%</span><span>}</span></code></pre></div>
<p>现在运行开发服务器，访问 <a href="http://127.0.0.1:8000/course/mine/" class="uri">http://127.0.0.1:8000/course/mine/</a> 。用属于教师组或者超级管理员的用户访问站点，然后添加一些内容到一个课程中。为了引入视频内容，你也可以复制任何一个 YouTube 视频 URL ，比如 ：<a href="https://www.youtube.com/watch?n=bgV39DlmZ2U" class="uri">https://www.youtube.com/watch?n=bgV39DlmZ2U</a> ,然后把它引入到表单的 <code>url</code> 字段中。在添加内容到课程中之后，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，点击课程然后点击<strong>ENROLL NOW</strong>按钮。你就可以在课程中报名了，然后被重定向到 <code>student_course_detail</code> URL 。下面这张图片展示了一个课程内容样本：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-5.png" alt="django-11-5"></p>
<p>真棒！你已经创建了一个渲染课程的通用接口了，它们中的每一个都会被用特定的方式渲染。</p>
<h2 id="使用缓存框架"><strong>使用缓存框架</strong></h2>
<p>你的应用的 HTTP 请求通常是数据库链接，数据处理，和模板渲染的。就处理数据而言，它的开销可比服务一个静态网站大多了。</p>
<p>请求开销在你的网站有越来越多的流量时是有意义的。这也使得缓存变得很有必要。通过缓存 HTTP 请求中 的查询结果，计算结果，或者是渲染上下文，你将会避免在接下来的请求中巨大的开销。这使得服务端的响应时间和处理时间变短。</p>
<p>Django 配备有一个健硕的缓存系统，这使得你可以使用不同级别的颗粒度来缓存数据。你可以缓存单一的查询，一个特定的输出视图，部分渲染的模板上下文，或者整个网站。缓存系统中的内容会在默认时间内被储存。你可以指定缓存数据过期的时间。</p>
<p>这是当你的站点收到一个 HTTP 请求时将会通常使用的缓存框架的方法：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>试着在缓存中寻找缓存数据
<span class="hljs-bullet">2. </span>如果找到了，就返回缓存数据
<span class="hljs-bullet">3. </span>如果没有找到，就执行下面的步骤：
<span class="hljs-code">    1. 执行查询或者处理请求来获得数据</span>
<span class="hljs-code">    2. 在缓存中保存生成的数据</span>
<span class="hljs-code">    3. 返回数据</span></code></pre>
<p>你可以在这里找到更多关于 Django 缓存系统的细节信息：<a href="https://docs.djangoproject.com/en/1.8/topics/cache/" class="uri">https://docs.djangoproject.com/en/1.8/topics/cache/</a> 。</p>
<h2 id="激活缓存后端"><strong>激活缓存后端</strong></h2>
<p>Django 配备有几个缓存后端，他们是：</p>
<ul>
<li><code>backends.memcached.MemcachedCache</code> 或 <code>backends.memcached.PyLibMCCache</code>：一个内存缓存后端。内存缓存是一个快速、高效的基于内存的缓存服务器。后端的使用取决于你选择的 Python 绑定（bindings）。</li>
<li><code>backends.db.DatabaseCache</code>： 使用数据库作为缓存系统。</li>
<li><code>backends.filebased.FileBasedCache</code>：使用文件储存系统。把每个缓存值序列化和储存为单一的文件。</li>
<li><code>backends.locmem.LocMemCache</code>：本地内存缓存后端。这是默认的缓存后端</li>
<li><code>backends.dummy.DummyCache</code>：一个用于开发的虚拟缓存后端。它实现了缓存交互界面而不用真正的缓存任何东西。缓存是独立进程且是线程安全的</li>
</ul>
<blockquote>
<p>对于可选的实现，使用内存的缓存后端吧，比如 <code>Memcached</code> 后端。</p>
</blockquote>
<h2 id="安装-memcached"><strong>安装 Memcached</strong></h2>
<p>我们将会使用 Memcached 缓存后端。内存缓存运行在内存中，它在 RAM 中分配了指定的数量。当分配的 RAM 满了时，Memcahed 就会移除最老的数据来保存新的数据。</p>
<p>在这个网址下载 Memcached： <a href="http://memcached.org/downloads" class="uri">http://memcached.org/downloads</a> 。如果你使用 Linux, 你可以使用下面的命令安装：</p>
<pre><code class="hljs go">./configure &amp;&amp; <span class="hljs-built_in">make</span> &amp;&amp; <span class="hljs-built_in">make</span> test &amp;&amp; sudo <span class="hljs-built_in">make</span> install</code></pre>
<p>如果你在使用 Mac OS X, 你可以使用命令 <code>brew install Memcached</code> 通过 Homebrew 包管理器来安装 Memcached 。你可以在这里下载 Homebrew <code>http://brew.sh</code></p>
<p>如果你正在使用 Windwos ，你可以在这里找到一个 Windows 的 Memcached 二进制版本：<a href="http://code.jellycan.com/memcached/" class="uri">http://code.jellycan.com/memcached/</a> 。</p>
<p>在安装 Memcached 之后，打开 shell ，使用下面的命令运行它：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">memcached</span> <span class="hljs-selector-tag">-l</span> 127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:11211</span></code></pre>
<p>Memcached 将会默认地在 11211 运行。当然，你也可以通过 <code>-l</code> 选项指定一个特定的主机和端口。你可以在这里找到更多关于 Memcached 的信息：<a href="http://memcached.org/" class="uri">http://memcached.org</a> 。</p>
<p>在安装 Memcached 之后，你需要安装它的 Python 绑定（bindings）。使用下面的命令安装：】</p>
<pre><code class="hljs cmake">python <span class="hljs-keyword">install</span> python3-memcached==<span class="hljs-number">1.51</span></code></pre>
<h2 id="缓存设置"><strong>缓存设置</strong></h2>
<p>Django 提供了如下的缓存设置：</p>
<ul>
<li><code>CACHES</code>：一个包含所有可用的项目缓存。</li>
<li><code>CACHE_MIDDLEWARE_ALIAS</code>：用于储存的缓存别名。</li>
<li><code>CACHE_MIDDLEWARE_KEY_PREFIX</code>：缓存键的前缀。设置一个缓存前缀来避免键的冲突，如果你在几个站点中分享相同的缓存的话。</li>
<li><code>CACHE_MIDDLEWARE_SECONDS</code> ：默认的缓存页面秒数</li>
</ul>
<p>项目的缓存系统可以使用 <code>CACHES</code> 设置来配置。这个设置是一个字典，让你可以指定多个缓存的配置。每个 <code>CACHES</code> 字典中的缓存可以指定下列数据：</p>
<ul>
<li><code>BACKEND</code>：使用的缓存后端。</li>
<li><code>KEY_FUNCTION</code>：包含一个指向回调函数的点路径的字符，这个函数以prefix(前缀)、verision(版本)、和 key (键) 作为参数并返回最终缓存键（cache key）。</li>
<li><code>KEY_PREFIX</code>：一个用于所有缓存键的字符串，避免冲突。</li>
<li><code>LOCATION</code>：缓存的位置。基于你的缓存后端，这可能是一个路径、一个主机和端口，或者是内存中后端的名字。</li>
<li><code>OPTIONS</code>：任何额外的传递向缓存后端的参数。</li>
<li><code>TIMEOUT</code>：默认的超时时间，以秒为单位，用于储存缓存键。默认设置是 300 秒，也就是五分钟。如果把它设置为 <code>None</code> ，缓存键将不会过期。</li>
<li><code>VERSION</code>：默认的缓存键的版本。对于缓存版本是很有用的。</li>
</ul>
<h2 id="把-memcached-添加进你的项目"><strong>把 memcached 添加进你的项目</strong></h2>
<p>让我们为我们的项目配置缓存。编辑 <code>educa</code> 项目的 <code>settings.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">CACHES <span class="op">=</span> {
    <span class="st"><span class="hljs-string">'default'</span></span>: {
        <span class="st"><span class="hljs-string">'BACKEND'</span></span>: <span class="st"><span class="hljs-string">'django.core.cache.backends.memcached.</span></span><span class="hljs-string">
</span><span class="st"><span class="hljs-string">MemcachedCache'</span></span>,
    <span class="co"><span class="hljs-string">'LOCATION'</span></span>: <span class="st"><span class="hljs-string">'127.0.0.1:11211'</span></span>,
    }
}</code></pre></div>
<p>我们正在使用 <code>MemcachedCache</code> 后端。我们使用 <code>address:port</code> 标记指定了它的位置。如果你有多个 <code>memcached</code> 实例，你可以在 <code>LOCATION</code> 中使用列表。</p>
<h2 id="监控缓存"><strong>监控缓存</strong></h2>
<p>这里有一个第三方包叫做 django-memcached-status ，它可以在管理站点展示你的项目的 <code>memcached</code> 实例的统计数据。为了兼容 Python3<strong>（译者夜夜月注：python3大法好。）</strong> ，从下面的分支中安装它：</p>
<pre><code class="hljs groovy">pip install git+<span class="hljs-string">git:</span><span class="hljs-comment">//github.com/zenx/django-memcached-status.git</span></code></pre>
<p>编辑 <code>settings.py</code> ，然后把 <code>memcached_status</code> 添加进 <code>INSTALLED_APPS</code> 设置中。确保 memcached 正在运行，在另外一个 shell 中打开开发服务器，然后访问 <a href="http://127.0.0.1:8000/adim/" class="uri">http://127.0.0.1:8000/adim/</a> ，使用超级用户登录进管理站点，你就可以看到如下的区域：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-6.png" alt="django-11-6"></p>
<p>这张图片展示了缓存使用。绿色代表了空闲的缓存，红色的表示使用了的空间。如果你点击方框的标题，它展示了你的 memcached 实例的统计详情。</p>
<p>我们已经为项目安装好了 memcached 并且可以监控它。让我们开始缓存数据吧！</p>
<h2 id="缓存级别"><strong>缓存级别</strong></h2>
<p>Django 提供了以下几个级别按照颗粒度上升的缓存排列：</p>
<ul>
<li><code>Low-level cache API</code>：提供了最高颗粒度。允许你缓存具体的查询或计算结果。</li>
<li><code>Per-view cache</code>：提供单一视图的缓存。</li>
<li><code>Template cache</code>：允许你缓存模板片段。</li>
<li><code>Per-site cache</code>：最高级的缓存。它缓存你的整个网站。</li>
</ul>
<blockquote>
<p>在你执行缓存之请仔细考虑下缓存策略。首先要考虑那些不是单一用户为基础的查询和计算开销</p>
</blockquote>
<h2 id="使用-low-level-cache-api-低级缓存api"><strong>使用 low-level cache API （低级缓存API）</strong></h2>
<p>低级缓存 API 让你可以缓存任意颗粒度的对象。它位于 <code>django.core.cache</code> 。你可以像这样导入它：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.cache <span class="im"><span class="hljs-keyword">import</span></span> cache</code></pre></div>
<p>这使用的是默认的缓存。它相当于 <code>caches['default']</code> 。通过它的别名来连接一个特定的缓存也是可能的：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.cache <span class="im"><span class="hljs-keyword">import</span></span> caches
my_cache <span class="op">=</span> caches[<span class="st"><span class="hljs-string">'alias'</span></span>]</code></pre></div>
<p>让我们看看缓存 API 是如何工作的。使用命令 <code>python manage.py shell</code> 打开 shell 然后执行下面的代码：</p>
<pre><code class="hljs python"><span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> django.core.cache <span class="hljs-keyword">import</span> cache
<span class="hljs-meta">&gt;&gt;&gt; </span>cache.set(<span class="hljs-string">'musician'</span>, <span class="hljs-string">'Django Reinhardt'</span>, <span class="hljs-number">20</span>)</code></pre>
<p>我们连接的是默认的缓存后端，使用 <code>set{key,value, timeout}</code> 来保存一个名为 <code>musician</code> 的键和它的为字符串 <code>Django Reinhardt</code> 的值 20 秒钟。如果我们不指定过期时间，Django 会使在 <code>CACHES</code> 设置中缓存后端的默认过期时间。现在执行下面的代码：</p>
<pre><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; cache.get(<span class="hljs-string">'musician'</span>)
<span class="hljs-string">'Django Reinhardt'</span></code></pre>
<p>我们在缓存中检索键。等待 20 秒然后指定相同的代码：</p>
<pre><code class="hljs python"><span class="hljs-meta">&gt;&gt;&gt; </span>cache.get(<span class="hljs-string">'musician'</span>)
<span class="hljs-keyword">None</span></code></pre>
<p><code>musician</code> 缓存键已经过期了，<code>get()</code> 方法返回了 None 因为键已经不在缓存中了。</p>
<blockquote>
<p>在缓存键中要避免储存 None 值，因为这样你就无法区分缓存值和缓存过期了</p>
</blockquote>
<p>让我们缓存一个查询集：</p>
<pre><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; from courses.models import Subject
<span class="hljs-meta">&gt;&gt;</span>&gt; subjects = Subject.objects.all()
<span class="hljs-meta">&gt;&gt;</span>&gt; cache.set(<span class="hljs-string">'all_subjects'</span>, subjects)</code></pre>
<p>我们执行了一个在 <code>Subject</code> 模型上的查询集，然后把返回的对象储存在 <code>all_subjects</code> 键中。让我们检索一下缓存数据:</p>
<pre><code class="hljs fsharp">&gt;&gt;&gt; cache.get('all_subjects')
<span class="hljs-meta">[&lt;Subject: Mathematics&gt;, &lt;Subject: Music&gt;, &lt;Subject: Physics&gt;,
&lt;Subject: Programming&gt;]</span></code></pre>
<p>我们将会在视图中缓存一些查询集。编辑 <code>courses</code> 应用的 <code>views.py</code> ，添加以下 导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.cache <span class="im"><span class="hljs-keyword">import</span></span> cache</code></pre></div>
<p>在 <code>CourseListView</code> 的 <code>get()</code> 方法，把下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">subjects <span class="op">=</span> Subject.objects.annotate(
        total_courses<span class="op">=</span>Count(<span class="st"><span class="hljs-string">'courses'</span></span>))</code></pre></div>
<p>替换为：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">subjects <span class="op">=</span> cache.get(<span class="st"><span class="hljs-string">'all_subjects'</span></span>)
<span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> subjects:
    subjects <span class="op">=</span> Subject.objects.annotate(
                total_courses<span class="op">=</span>Count(<span class="st"><span class="hljs-string">'courses'</span></span>))
    cache.<span class="bu">set</span>(<span class="st"><span class="hljs-string">'all_subjects'</span></span>, subjects)</code></pre></div>
<p>在这段代码中，我们首先尝试使用 <code>cache.get()</code> 来从缓存中得到 <code>all_students</code> 键。如果所给的键没有找到，返回的是 None 。如果键没有被找到（没有被缓存，或者缓存了但是过期了），我们就执行查询来检索所有的 <code>Subject</code> 对象和它们课程的数量，我们使用 <code>cache.set()</code> 来缓存结果。</p>
<p>打开代发服务器，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000</a> 。当视图被执行的时候，缓存键没有被找到的话查询集就会被执行。访问 <a href="http://127.0.0.1:8000/adim/" class="uri">http://127.0.0.1:8000/adim/</a> 然后打开 memcached 统计。你可以看到类似于下面的缓存的使用数据：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-11-7.png" alt="django-11-7"></p>
<p>看一眼 <strong>Curr Items</strong> 应该是 1 。这表示当前有一个内容被缓存。<strong>Get Hits</strong> 表示有多少的 get 操作成功了，<strong>Get Miss</strong>表示有多少的请求丢失了。<strong>Miss Ratio</strong> 是使用它们俩来计算的。</p>
<p>现在导航回 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，重载页面几次。如果你现在看缓存统计的话，你就会看到更多的（<strong>Get Hits</strong>和<strong>Cmd Get</strong>被执行了）</p>
<h2 id="基于动态数据的缓存"><strong>基于动态数据的缓存</strong></h2>
<p>有很多时候你都会想使用基于动态数据的缓存的。基于这样的情况，你必须要创建包含所有要求信息的动态键来特别定以缓存数据。编辑 <code>courses</code> 应用的 <code>views.py</code> ，修改 <code>CourseListView</code> ，让它看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseListView</span><span class="hljs-params">(TemplateResponseMixin, View)</span>:</span>
    model <span class="op">=</span> Course
    template_name <span class="op">=</span> <span class="st"><span class="hljs-string">'courses/course/list.html'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, subject</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        subjects <span class="op">=</span> cache.get(<span class="st"><span class="hljs-string">'all_subjects'</span></span>)
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> subjects:
            subjects <span class="op">=</span> Subject.objects.annotate(
                            total_courses<span class="op">=</span>Count(<span class="st"><span class="hljs-string">'courses'</span></span>))
            cache.<span class="bu">set</span>(<span class="st"><span class="hljs-string">'all_subjects'</span></span>, subjects)
        all_courses <span class="op">=</span> Course.objects.annotate(
            total_modules<span class="op">=</span>Count(<span class="st"><span class="hljs-string">'modules'</span></span>))
        <span class="cf"><span class="hljs-keyword">if</span></span> subject:
            subject <span class="op">=</span> get_object_or_404(Subject, slug<span class="op">=</span>subject)
            key <span class="op">=</span> <span class="st"><span class="hljs-string">'subject_{}_courses'</span></span>.<span class="bu">format</span>(subject.<span class="bu">id</span>)
            courses <span class="op">=</span> cache.get(key)
            <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> courses:
                courses <span class="op">=</span> all_courses.<span class="bu">filter</span>(subject<span class="op">=</span>subject)
                cache.<span class="bu">set</span>(key, courses)
        <span class="cf"><span class="hljs-keyword">else</span></span>:
            courses <span class="op">=</span> cache.get(<span class="st"><span class="hljs-string">'all_courses'</span></span>)
            <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> courses:
                courses <span class="op">=</span> all_courses
                cache.<span class="bu">set</span>(<span class="st"><span class="hljs-string">'all_courses'</span></span>, courses)
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.render_to_response({<span class="st"><span class="hljs-string">'subjects'</span></span>: subjects,
                                        <span class="st"><span class="hljs-string">'subject'</span></span>: subject,
                                        <span class="co"><span class="hljs-string">'courses'</span></span>: courses})</code></pre></div>
<p>在这个场景中，我们把课程和根据科目筛选的课程都缓存了。我们使用 <code>all_courses</code> 缓存键来储存所有的课程，如果没有给科目的话。如果给了一个科目的话我们就用 <code>'subject_()_course'.format(subject.id)</code>动态的创建缓存键。</p>
<p>注意到我们不能用一个缓存查询集来创建另外的查询集是很重要的，因为我们已经缓存了当前的查询结果，所以我们不能这样做：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">courses <span class="op">=</span> cache.get(<span class="st"><span class="hljs-string">'all_courses'</span></span>)
courses.<span class="bu">filter</span>(subject<span class="op">=</span>subject)</code></pre></div>
<p>相反，我们需要创建基本的查询集 <code>Course.objects.annotate(total_modules=Count('modules'))</code> ，它不会被执行除非你强制执行它，然后用它来更进一步的用 <code>all_courses.filter(subject=subject)</code> 限制查询集万一数据没有在缓存中找到的话。</p>
<h2 id="缓存模板片段"><strong>缓存模板片段</strong></h2>
<p>缓存模板片段是一个高级别的方法。你需要使用 <code><span>{</span><span>%</span> load cache <span>%</span><span>}</span></code> 在模板中载入缓存模板标签。然后你就可以使用 <code><span>{</span><span>%</span>  cache <span>%</span><span>}</span></code> 模板标签来缓存特定的模板片段了。你通常可以像下面这样使用缓存标签：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> cache 300 fragment_name <span>%</span><span>}</span>
...
<span>{</span><span>%</span> endcache <span>%</span><span>}</span></code></pre></div>
<p><code><span>{</span><span>%</span>  cache <span>%</span><span>}</span></code> 标签要求两个参数：过期时间，以秒为单位，和一个片段名称。如果你需要缓存基于动态数据的内容，你可以通过传递额外的参数给 <code><span>{</span><span>%</span> cache <span>%</span><span>}</span></code> 模板标签来特别的指定片段。</p>
<p>编辑 <code>students</code> 应用的 <code>/students/course/detail.html</code> 。在顶部添加以下代码，就在 <code><span>{</span><span>%</span> extends <span>%</span><span>}</span></code> 标签的后面：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load cache <span>%</span><span>}</span></code></pre></div>
<p>然后把下面几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> for content in module.contents.all <span>%</span><span>}</span>
    <span>{</span><span>%</span> with item=content.item <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span><span>{</span><span>{</span> item.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
        <span>{</span><span>{</span> item.render <span>}</span><span>}</span>
    <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span></code></pre></div>
<p>替换为：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> cache 600 module_contents module <span>%</span><span>}</span>
    <span>{</span><span>%</span> for content in module.contents.all <span>%</span><span>}</span>
        <span>{</span><span>%</span> with item=content.item <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span><span>{</span><span>{</span> item.title <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
            <span>{</span><span>{</span> item.render <span>}</span><span>}</span>
        <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
    <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span>{</span><span>%</span> endcache <span>%</span><span>}</span></code></pre></div>
<p>我们使用名字 <code>module_contents</code> 和传递当前的 <code>Module</code> 对象来缓存模板片段。这对于当请求不同的模型是避免缓存一个模型的内容和服务错误的内容来说是很重要的。</p>
<blockquote>
<p>如果 <code>USE_I18N</code> 设置是为 True，单一站点的中间件缓存将会遵照当前激活的语言。如果你使用了 <code><span>{</span><span>%</span>  cache <span>%</span><span>}</span></code> 模板标签以及可用翻译特定的变量中的一个，那么他们的效果将会是一样的，比如：<code><span>{</span><span>%</span>  cache 600 name request.LANGUAGE_CODE <span>%</span><span>}</span></code></p>
</blockquote>
<h2 id="缓存视图"><strong>缓存视图</strong></h2>
<p>你可以使用位于 <code>django.views.decrators.cache</code> 的 <code>cache_page</code> 装饰器来焕春输出的单个视图。装饰器要求一个过期时间的参数（以秒为单位）。</p>
<p>让我们在我们的视图中使用它。编辑 <code>students</code> 应用的 <code>urls.py</code> ，添加以下 导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.decorators.cache <span class="im"><span class="hljs-keyword">import</span></span> cache_page</code></pre></div>
<p>然后按照如下在 <code>student_course_detail_module</code> URL 模式上应用 <code>cache_page</code> 装饰器：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^course/(?P&lt;pk&gt;\d+)/$'</span></span>,
    cache_page(<span class="dv"><span class="hljs-number">60</span></span> <span class="op">*</span> <span class="dv"><span class="hljs-number">15</span></span>)(views.StudentCourseDetailView.as_view()),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_course_detail'</span></span>),

url(<span class="vs"><span class="hljs-string">r'^course/(?P&lt;pk&gt;\d+)/(?P&lt;module_id&gt;\d+)/$'</span></span>,
    cache_page(<span class="dv"><span class="hljs-number">60</span></span> <span class="op">*</span> <span class="dv"><span class="hljs-number">15</span></span>)(views.StudentCourseDetailView.as_view()),
    name<span class="op">=</span><span class="st"><span class="hljs-string">'student_course_detail_module'</span></span>),</code></pre></div>
<p>现在 <code>StudentCourseDetailView</code> 的结果就会被缓存 15 分钟了。</p>
<blockquote>
<p>单一的视图缓存使用 URL 来创建缓存键。多个指向同一个视图的 URLs 将会被分开储存</p>
</blockquote>
<h2 id="使用单一站点缓存"><strong>使用单一站点缓存</strong></h2>
<p>这是最高级的缓存。他让你可以缓存你的整个站点。</p>
<p>为了使用单一站点缓存，你需要编辑项目中的 <code>settings.py</code> ，把 <code>UpdateCacheMiddleware</code> 和 <code>FetchFromCacheMiddleware</code> 添加进 <code>MIDDLEWARE_CLASSES</code> 设置中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">MIDDLEWARE_CLASSES <span class="op">=</span> (
    <span class="st"><span class="hljs-string">'django.contrib.sessions.middleware.SessionMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.cache.UpdateCacheMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.common.CommonMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.cache.FetchFromCacheMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.csrf.CsrfViewMiddleware'</span></span>,
    <span class="co"><span class="hljs-comment"># ...</span></span>
)</code></pre></div>
<p>记住在请求的过程中，中间件是按照所给的顺序来执行的，在相应过程中是逆序执行的。<code>UpdateCacheMiddleware</code> 被放在 <code>CommonMiddleware</code> 之前，因为它在相应时才执行，此时中间件是逆序执行的。<code>FetchFromCacheMiddleware</code> 被放在 <code>CommonMiddleware</code> 之后，是因为它需要连接后者的的请求数据集。</p>
<p>然后，把下列设置添加进 <code>settings.py</code> 文件：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">CACHE_MIDDLEWARE_ALIAS <span class="op">=</span> <span class="st"><span class="hljs-string">'default'</span></span>
CACHE_MIDDLEWARE_SECONDS <span class="op">=</span> <span class="dv"><span class="hljs-number">60</span></span> <span class="op">*</span> <span class="dv"><span class="hljs-number">15</span></span> <span class="co"><span class="hljs-comment"># 15 minutes</span></span>
CACHE_MIDDLEWARE_KEY_PREFIX <span class="op">=</span> <span class="st"><span class="hljs-string">'educa'</span></span></code></pre></div>
<p>在这些设置中我们为中间件使用了默认的缓存，然后我们把全局缓存过期时间设置为 15 分钟。我们也指定了所有的缓存键前缀来避免冲突万一我们为多个项目使用了相同的 memcached 后端。我们的站点现在将会为所有的 GET 请求缓存以及返回缓存内容。</p>
<p>我们已经完成了这个来测试单一站点缓存功能。尽管，以站点的缓存对于我们来说是不怎么合适的，因为我们的课程管理视图需要展示更新数据来实时的给出任何的修改。我们项目中的最好的方法是缓存用于展示给学生的课程内容的模板或者视图数据。</p>
<p>我们已经大致体验过了 Django 提供的方法来缓存数据。你应合适的定义你自己的缓存策略，优先考虑开销最大的查询集或者计算。</p>
<h2 id="总结"><strong>总结</strong></h2>
<p>在这一章中，我们创建了一个用于课程的公共视图，创建了一个用于学生注册和报名课程的系统。我们安装了 memcached 以及实现了不同级别的缓存。</p>
<p>在下一章中，我们将会为你的项目创建 RESTful API。</p>
</div>
