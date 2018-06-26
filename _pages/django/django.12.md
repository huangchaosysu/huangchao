---
layout: single
permalink: /django/example3/
title: "Django By Example 第三章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第三章">第三章</h1>
<h2 id="扩展你的blog应用">扩展你的blog应用</h2>
<p>在上一章中我们学习了表单的基础以及如何在项目中集成第三方的应用。本章将会包含以下内容：</p>
<ul>
<li>创建自定义的模板标签（template tags)和过滤器（filters）</li>
<li>添加一个站点地图和帖子反馈（post feed）</li>
<li>使用Solr和Haystack构建一个搜索引擎</li>
</ul>
<h3 id="创建自定义的模板标签template-tags和过滤器filters">创建自定义的模板标签（template tags)和过滤器（filters）</h3>
<p>Django提供了很多内置的模板标签（tags），例如<code><span>{</span><span>%</span> if <span>%</span><span>}</span></code>或者<code><span>{</span><span>%</span> block <span>%</span><span>}</span></code>。你已经在你的模板（template）中使用过一些了。你可以在https://docs.djangoproject.com/en/1.8/ ref/templates/builtins/ 中找到关于内置模板标签（template tags）以及过滤器（filter）的完整参考。</p>
<p>当然，Django也允许你创建自己的模板标签（template tags）来执行自定义的动作。当你需要在你的模板中添加功能而Django模板标签（template tags)的核心设置无法提供此功能的时候，自定义模板标签会非常方便。</p>
<h3 id="创建自定义的模板标签template-tags">创建自定义的模板标签（template tags）</h3>
<p>Django提供了以下帮助函数（functions）来允许你以一种简单的方式创建自己的模板标签（template tags）：</p>
<ul>
<li>simple_tag：处理数据并返回一个字符串（string）</li>
<li>inclusion_tag：处理数据并返回一个渲染过的模板（template）</li>
<li>assignment_tag：处理数据并在上下文（context）中设置一个变量（variable）</li>
</ul>
<p>模板标签（template tags）必须存在Django的应用中。</p>
<p>进入你的blog应用目录，创建一个新的目录命名为<em>templatetags</em>然后在该目录下创建一个空的<em><strong>init</strong>.py</em>文件。接着在该目录下继续创建一个文件并命名为<em>blog_tags.py</em>。到此，我们的blog应用文件结构应该如下所示：</p>
<pre><code class="hljs r">blog/
    __init__.py
    models.py
    <span class="hljs-keyword">...</span>
    templatetags/
        __init__.py
        blog_tags.py</code></pre>
<p>文件的命名是非常重要的。你将在模板（template）中使用这些模块的名字加载你的标签（tags)。</p>
<p>我们将要开始创建一个简单标签（simple tag）来获取blog中所有已发布的帖子。编辑你刚才创建的<em>blog_tags.py</em>文件，加入以下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django <span class="hljs-keyword">import</span> template

register = template.Library()

<span class="hljs-keyword">from</span> ..models <span class="hljs-keyword">import</span> Post

<span class="hljs-meta">@register.simple_tag</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">total_posts</span><span class="hljs-params">()</span>:</span>
    <span class="hljs-keyword">return</span> Post.published.count()</code></pre>
<p>我们已经创建了一个简单的模板标签（template tag）用来取回目前为止所有已发布的帖子。每一个模板标签（template tags）都需要包含一个叫做<em>register</em>的变量来表明自己是一个有效的标签（tag）库。这个变量是<em>template.Library</em>的一个实例，它是用来注册你自己的模板标签（template tags）和过滤器（filter）的。我们用一个Python函数定义了一个名为<em>total_posts</em>的标签，并用<code>@register.simple-tag</code>装饰器定义此函数为一个简单标签（tag）并注册它。</p>
<p>Django将会使用这个函数名作为标签（tag）名。如果你想使用别的名字来注册这个标签（tag），你可以指定装饰器的<em>name</em>属性，比如<code>@register.simple_tag(name='my_tag')</code>。</p>
<blockquote>
<p>在添加了新的模板标签（template tags）模块后，你必须重启Django开发服务才能使用新的模板标签（template tags)和过滤器（filters)。</p>
</blockquote>
<p>在使用自定义的模板标签（template tags)之前，你必须使用<em><span>{</span><span>%</span> load <span>%</span><span>}</span></em>标签在模板（template）中来加载它们才能有效。就像之前提到的，你需要使用包含了你的模板标签（template tags)和过滤器（filter)的Python模块的名字。打开<em>blog/base.html</em>模板（template）在顶部添加<em><span>{</span><span>%</span> load blog_tags <span>%</span><span>}</span></em>加载你自己的模板标签（template tags)模块。之后使用你创建的标签（tag）来显示你的帖子总数。只需要在你的模板（template）中添加<em><span>{</span><span>%</span> total_posts <span>%</span><span>}</span></em>。最新的模板（template）看上去如下所示：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> blog_tags <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> staticfiles <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-meta">&lt;!DOCTYPE html&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">static</span></span> "css/blog.css" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">"stylesheet"</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"content"</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"sidebar"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>My blog<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>This is my blog. I've written </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">total_posts</span> <span>%</span><span>}</span></span><span class="xml"> posts so far.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre>
<p>我们需要重启服务来保证新的文件被加载到项目中。使用<em>Ctrl+C</em>停止服务然后通过以下命令再次启动：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">runserver</span></code></pre>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 。你会在站点的侧边栏（sidebar）看到帖子的总数，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-1.png" alt="django-3-1"></p>
<p>自定义模板标签（template tags）的作用是你可以处理任何的数据并且在任何模板（template）中添加它而不用去关心视图（view）执行。你可以在你的模板（template）中运行查询集（QuerySets）或者处理任何数据展示结果。</p>
<p>现在，我们要创建另外一个标签（tag），可以在我们blog的侧边栏（sidebar）展示最新的几个帖子。这一次我们要使用一个包含标签（inclusion tag）。使用一个包含标签（inclusion tag），你就可以利用模板标签（template tags）返回的上下文变量（context variables）来渲染模板（template）。编辑<em>blog_tags.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-meta">@register.inclusion_tag('blog/post/latest_posts.html')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">show_latest_posts</span><span class="hljs-params">(count=<span class="hljs-number">5</span>)</span>:</span>
    latest_posts = Post.published.order_by(<span class="hljs-string">'-publish'</span>)[:count]
    <span class="hljs-keyword">return</span> {<span class="hljs-string">'latest_posts'</span>: latest_posts}</code></pre>
<p>在以上代码中，<a>我们通过装饰器*@register.inclusion_tag*注册模板标签</a>（template tag），指定模板（template）必须被<em>blog/post/latest_posts.html</em>返回的值渲染。我们的模板标签（template tag)将会接受一个可选的<em>count</em>参数（默认是5）允许我们指定我们想要显示的帖子数量。我们使用这个变量来限制<code>Post.published.order_by('-publish')[:count]</code>查询的结果。请注意，这个函数返回了一个字典变量而不是一个简单的值。包含标签（inclusion tags）必须返回一个字典值，作为上下文（context）来渲染特定的模板（template）。包含标签（inclusion tags）返回一个字典。这个我们刚创建的模板标签（template tag）可以通过传入可选的评论数量值来使用显示，类似*<span>{</span><span>%</span> show_latest_posts 3 <span>%</span><span>}</span>。</p>
<p>现在，在<em>blog/post/</em>下创建一个新的模板（template）文件并且命名为<em>latest_posts.html</em>。在该文件中添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> post <span class="hljs-keyword">in</span> latest_posts <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.title <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span></code></pre>
<p>在这里，我们使用模板标签（template tag）返回的<em>latest_posts</em>变量展示了一个无序的帖子列表。现在，编辑<em>blog/base.hmtl</em>模板（template）添加这个新的模板标签（template tag）来展示最新的3条帖子。侧边栏（sidebar）区块（block）看上去应该如下所示：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"sidebar"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>My blog<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>This is my blog. I've written </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">total_posts</span> <span>%</span><span>}</span></span><span class="xml"> posts so far.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span>Latest posts<span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">show_latest_posts</span> 3 <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre>
<p>这个模板标签（template tag）被调用而且传入了需要展示的帖子数（原文此处 number of comments，应该是写错了)。当前模板（template）在给予上下文（context）的位置会被渲染。</p>
<p>现在，回到浏览器刷新这个页面，侧边栏应该如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-2.png" alt="django-3-2"></p>
<p>最后，我们来创建一个分配标签（assignment tag）。分配标签（assignment tag）类似简单标签（simple tags）但是他们将结果存储在给予的变量中。我们将会创建一个分配标签（assignment tag）来展示拥有最多评论的帖子。编辑<em>blog_tags.py</em>文件，在其中添加如下导入和模板标签：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.db.models <span class="hljs-keyword">import</span> Count

<span class="hljs-meta">@register.assignment_tag</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_most_commented_posts</span><span class="hljs-params">(count=<span class="hljs-number">5</span>)</span>:</span>
    <span class="hljs-keyword">return</span> Post.published.annotate(
                total_comments=Count(<span class="hljs-string">'comments'</span>)
            ).order_by(<span class="hljs-string">'-total_comments'</span>)[:count]
    </code></pre>
<p>这个查询集（QuerySet）使用<em>annotate()</em>函数，为了进行聚合查询，使用了<em>Count</em>聚合函数。我们构建了一个查询集（QuerySet），聚合了每一个帖子的评论总数并保存在<em>total_comments</em>字段中，接着我们通过这个字段对查询集（QuerySet）进行排序。我们还提供了一个可选的<em>count</em>变量，通过给定的值来限制返回的帖子数量。</p>
<p>除了<em>Count</em>以外，Django还提供了不少聚合函数，例如<em>Avg,Max,Min,Sum</em>.你可以在 <a href="https://docs.djangoproject.com/en/1.8/topics/db/aggregation/" class="uri">https://docs.djangoproject.com/en/1.8/topics/db/aggregation/</a> 页面读到更多关于聚合方法的信息。</p>
<p>编辑<em>blog/base.html</em>模板（template），在侧边栏（sidebar）<code>&lt;div&gt;</code>元素中添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span>Most commented posts<span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">get_most_commented_posts</span> <span class="hljs-keyword">as</span> most_commented_posts <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> post <span class="hljs-keyword">in</span> most_commented_posts <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.title <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span></code></pre>
<p>使用分配模板标签（assignment template tags）的方法是<em><span>{</span><span>%</span> template_tag as variable <span>%</span><span>}</span></em>。对于我们的模板标签（template tag）来说，我们使用<em><span>{</span><span>%</span> get_most_commented_posts as most_commented_posts <span>%</span><span>}</span></em>。 这样，我们可以存储这个模板标签（template tag）返回的结果到一个新的名为<em>most_commented_posts</em>变量中。之后，我们就可以用一个无序列表(unordered list)显示返回的帖子。</p>
<p>现在，打开浏览器刷新页面来看下最终的结果，应该如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-3.png" alt="django-3-3"></p>
<p>你可以在 <a href="https://docs.djangoproject.com/en/1.8/howto/custom-template-tags/" class="uri">https://docs.djangoproject.com/en/1.8/howto/custom-template-tags/</a> 页面得到更多的关于自定义模板标签（template tags）的信息。</p>
<h3 id="创建自定义的模板过滤器template-filters">创建自定义的模板过滤器（template filters）</h3>
<p>Django拥有多种内置的模板过滤器（template filters）允许你对模板（template）中的变量进行修改。这些过滤器其实就是Python函数并提供了一个或两个参数————一个是需要处理的变量值，一个是可选的参数。它们返回的值可以被展示或者被别的过滤器（filters）处理。一个过滤器（filter）类似<em><span>{</span><span>{</span> variable|my_filter <span>}</span><span>}</span></em>或者再带上一个参数，类似<em><span>{</span><span>{</span> variable|my_filter:"foo" <span>}</span><span>}</span></em>。你可以对一个变量调用你想要的多次过滤器（filter），类似<em><span>{</span><span>{</span> variable|filter1|filter2 <span>}</span><span>}</span></em>, 每一个过滤器（filter）都会对上一个过滤器（filter）输出的结果进行过滤。</p>
<p>我们这就创建一个自定义的过滤器（filter），可以在我们的blog帖子中使用markdown语法，然后在模板（template）中将帖子内容转变为HTML。Markdown是一种非常容易使用的文本格式化语法并且它可以转变为HTML。你可以在 <a href="http://daringfireball.net/projects/markdown/basics" class="uri">http://daringfireball.net/projects/markdown/basics</a> 页面学习这方面的知识。</p>
<p>首先，通过pip渠道安装Python的markdown模板：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Markdown==<span class="hljs-number">2.6</span>.<span class="hljs-number">2</span></code></pre>
<p>之后编辑<em>blog_tags.py</em>文件，包含如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.utils.safestring <span class="hljs-keyword">import</span> mark_safe
<span class="hljs-keyword">import</span> markdown

<span class="hljs-meta">@register.filter(name='markdown')</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">markdown_format</span><span class="hljs-params">(text)</span>:</span>
    <span class="hljs-keyword">return</span> mark_safe(markdown.markdown(text))</code></pre>
<p>我们使用和模板标签（template tags）一样的方法来注册我们自己的模板过滤器（template filter）。为了避免我们的函数名和<em>markdown</em>模板名起冲突，我们将我们的函数命名为<em>markdown_format</em>，然后将过滤器（filter）命名为<em>markdown</em>，在模板（template）中的使用方法类似<em><span>{</span><span>{</span> variable|markdown <span>}</span><span>}</span></em>。Django会转义过滤器（filter）生成的HTML代码。我们使用Django提供的<em>mark_safe</em>方法来标记结果，在模板（template）中作为安全的HTML被渲染。默认的，Django不会信赖任何HTML代码并且在输出之前会进行转义。唯一的例外就是被标记为安全转义的变量。这样的操作可以阻止Django从输出中执行潜在的危险的HTML，并且允许你创建一些例外情况只要你知道你正在运行的是安全的HTML。</p>
<p>现在，在帖子列表和详情模板（template）中加载你的模板标签（template tags）模块。在<em>post/list.html</em>和<em>post/detail.html</em>模板（template）的顶部<em><span>{</span><span>%</span> extends <span>%</span><span>}</span></em>的后方添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> blog_tags <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>在<em>post/detail.thml</em>模板中，替换以下内容：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.body|<span class="hljs-name">linebreaks</span> <span>}</span><span>}</span></span><span class="xml"></span></code></pre>
<p>替换成：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.body|markdown <span>}</span><span>}</span></span><span class="xml"></span></code></pre>
<p>之后，在<em>post/list.html</em>文件中，替换以下内容：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.body|<span class="hljs-name">truncatewords</span>:30|<span class="hljs-name">linebreaks</span> <span>}</span><span>}</span></span><span class="xml"></span></code></pre>
<p>替换成：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.body|markdown|<span class="hljs-name">truncatewords</span>_html:30 <span>}</span><span>}</span></span><span class="xml"></span></code></pre>
<p><em>truncateword_html</em>过滤器（filter）会在一定数量的单词后截断字符串，避免没有关闭的HTML标签（tags）。</p>
<p>现在，打开 <a href="http://127.0.0.1:8000/admin/blog/post/add/" class="uri">http://127.0.0.1:8000/admin/blog/post/add/</a> ,添加一个帖子，内容如下所示：</p>
<pre><code class="hljs markdown"><span class="hljs-section">This is a post formatted with markdown
--------------------------------------</span>
<span class="hljs-emphasis">*This is emphasized*</span> and <span class="hljs-strong">**this is more emphasized**</span>.
Here is a list:
<span class="hljs-bullet">* </span>One
<span class="hljs-bullet">* </span>Two
<span class="hljs-bullet">* </span>Three
And a [<span class="hljs-string">link to the Django website</span>](<span class="hljs-link">https://www.djangoproject.com/</span>)</code></pre>
<p>在浏览器中查看帖子的渲染情况，你会看到如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-4.png" alt="django-3-4"></p>
<p>就像你所看到的，自定义的模板过滤器（template filters）对于自定义的格式化是非常有用的。你可以在 <a href="https://docs.djangoproject.com/en/1.8/howto/custom-template-tags/#writing-custom-templatefilters" class="uri">https://docs.djangoproject.com/en/1.8/howto/custom-template-tags/#writing-custom-templatefilters</a> 页面获取更多关于自定义过滤器（filter）的信息。</p>
<h3 id="为你的站点添加一个站点地图sitemap">为你的站点添加一个站点地图（sitemap）</h3>
<p>Django自带一个站点地图（sitemap）框架，允许你为你的站点动态生成站点地图（sitemap）。一个站点地图（sitemap）是一个xml文件，它会告诉搜索引擎你的网站中存在的页面，它们的关联和它们更新的频率。使用站点地图（sitemap），你可以帮助网络爬虫（crawlers）来对你的网站内容进行索引和标记。</p>
<p>Django站点地图（sitemap）框架依赖<em>django.contrib.sites</em>模块，这个模块允许你将对象和正在你项目运行的特殊网址关联起来。当你想用一个单独Django项目运行多个网站时，这是非常方便的。为了安装站点地图（sitemap）框架，我们需要在项目中激活<em>sites</em>和<em>sitemap</em>这两个应用。编辑项目中的<em>settings.py</em>文件在<em>INSTALLED_APPS</em>中添加<em>django.contrib.sites</em>和<em>django.contrib.sitemaps</em>。之后为站点ID定义一个新的设置，如下所示：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">SITE_ID</span> = <span class="hljs-number">1</span>
<span class="hljs-comment"># Application definition</span>
INSTALLED_APPS = (
<span class="hljs-comment"># ...</span>
<span class="hljs-string">'django.contrib.sites'</span>,
<span class="hljs-string">'django.contrib.sitemaps'</span>,
)</code></pre>
<p>现在，运行以下命令在数据库中为Django的站点应用创建所需的表：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会看到如下的输出内容：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">sites</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p><em>sites</em>应用现在已经在数据库中进行了同步。现在，在你的<em>blog</em>应用目录下创建一个新的文件命名为<em>sitemaps.py</em>。打开这个文件，输入以下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib.sitemaps <span class="hljs-keyword">import</span> Sitemap
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Post

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PostSitemap</span><span class="hljs-params">(Sitemap)</span>:</span>
    changefreq = <span class="hljs-string">'weekly'</span>
    priority = <span class="hljs-number">0.9</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">items</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> Post.published.all()
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">lastmod</span><span class="hljs-params">(self, obj)</span>:</span>
        <span class="hljs-keyword">return</span> obj.publish
        </code></pre>
<p>通过继承<em>sitemaps</em>模块提供的<em>Sitemap</em>类我们创建一个自定义的站点地图（sitemap）。<em>changefreq</em>和<em>priority</em>属性表明了帖子页面修改的频率和它们在网站中的关联性（最大值是1）。<em>items()</em>方法返回了在这个站点地图（sitemap）中所包含对象的查询集（QuerySet）。默认的，Django在每个对象中调用<em>get_absolute_url()</em>方法来获取它的URL。请记住，这个方法是我们在第一章（创建一个blog应用）中创建的，用来获取每个帖子的标准URL。如果你想为每个对象指定URL，你可以在你的站点地图（sitemap）类中添加一个<em>location</em>方法。<em>lastmode</em>方法接收<em>items()</em>返回的每一个对象并且返回对象的最后修改时间。<em>changefreq</em>和<em>priority</em>两个方法既可以是方法也可以是属性。你可以在Django的官方文档 <a href="https://docs.djangoproject.com/en/1.8/ref/contrib/sitemaps/" class="uri">https://docs.djangoproject.com/en/1.8/ref/contrib/sitemaps/</a> 页面中获取更多的站点地图（sitemap）参考。</p>
<p>最后，我们只需要添加我们的站点地图（sitemap）URL。编辑项目中的主*urls.py文件，如下所示添加站点地图（sitemap）：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> include, url
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> django.contrib.sitemaps.views <span class="hljs-keyword">import</span> sitemap
<span class="hljs-keyword">from</span> blog.sitemaps <span class="hljs-keyword">import</span> PostSitemap

sitemaps = {
    <span class="hljs-string">'posts'</span>: PostSitemap,
}

urlpatterns = [
    url(<span class="hljs-string">r'^admin/'</span>, include(admin.site.urls)),
    url(<span class="hljs-string">r'^blog/'</span>,
        include(<span class="hljs-string">'blog.urls'</span>namespace=<span class="hljs-string">'blog'</span>, app_name=<span class="hljs-string">'blog'</span>)),
    url(<span class="hljs-string">r'^sitemap\.xml$'</span>, sitemap, {<span class="hljs-string">'sitemaps'</span>: sitemaps},
        name=<span class="hljs-string">'django.contrib.sitemaps.views.sitemap'</span>),
]</code></pre>
<p>在这里，我们加入了必要的导入并定义了一个<em>sitemaps</em>的字典。我们定义了一个URL模式来匹配<em>sitemap.xml</em>并使用<em>sitemap</em>视图（view）。<em>sitemaps</em>字典会被传入到<em>sitemap</em>视图（view）中。现在，在浏览器中打开 <a href="http://127.0.0.1:8000/sitemap.xml" class="uri">http://127.0.0.1:8000/sitemap.xml</a> 。你会看到类似下方的XML代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-meta">&lt;?xml version="1.0" encoding="UTF-8"?&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">urlset</span> <span class="hljs-attr">xmlns</span>=<span class="hljs-string">"http://www.sitemaps.org/schemas/sitemap/0.9"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">url</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">loc</span>&gt;</span>http://example.com/blog/2015/09/20/another-post/<span class="hljs-tag">&lt;/<span class="hljs-name">loc</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">lastmod</span>&gt;</span>2015-09-29<span class="hljs-tag">&lt;/<span class="hljs-name">lastmod</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">changefreq</span>&gt;</span>weekly<span class="hljs-tag">&lt;/<span class="hljs-name">changefreq</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">priority</span>&gt;</span>0.9<span class="hljs-tag">&lt;/<span class="hljs-name">priority</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">url</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">url</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">loc</span>&gt;</span>http://example.com/blog/2015/09/20/who-was-djangoreinhardt/<span class="hljs-tag">&lt;/<span class="hljs-name">loc</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">lastmod</span>&gt;</span>2015-09-20<span class="hljs-tag">&lt;/<span class="hljs-name">lastmod</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">changefreq</span>&gt;</span>weekly<span class="hljs-tag">&lt;/<span class="hljs-name">changefreq</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">priority</span>&gt;</span>0.9<span class="hljs-tag">&lt;/<span class="hljs-name">priority</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">url</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">urlset</span>&gt;</span></span></code></pre>
<p>调用<em>get_absolute_url()</em>方法，每个帖子的URL已经被构建。如同我们之前在站点地图(sitemap)中所指定的，<em>lastmode</em>属性对应该帖子的<em>publish</em>日期字段，<em>changefreq</em>和<em>priority</em>属性也从我们的<em>PostSitemap</em>类中带入。你能看到被用来构建URL的域（domain）是<em>example.com</em>。这个域（domain）来自存储在数据库中的一个<em>Site</em>对象。这个默认的对象是在我们之前同步<em>sites</em>框架数据库时创建的。在你的浏览器中打开http://127.0.0.1:8000/admin/sites/site/ ，你会看到如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-5.png" alt="django-3-5"></p>
<p>这是<em>sites</em>框架管理视图（admin view）的列表显示。在这里，你可以设置<em>sites</em>框架使用的域（domain）或者主机（host），而且应用也依赖它们。为了生成能在我们本地环境可用的URL，更改域（domain）名为<em>127.0.0.1:8000</em>，如下图所示并保存：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-6.png" alt="django-3-6"></p>
<p>为了开发需要我们指向了我们本地主机。在生产环境中，你必须使用你自己的<em>sites</em>框架域（domain)名。</p>
<h3 id="为你的blog帖子创建feeds">为你的blog帖子创建feeds</h3>
<blockquote>
<p>译者注：这节中有不少英文单词，例如feed，syndication，Atom等，没有比较好的翻译，很多地方也都是直接不翻译保留原文，所以我也保留原文）</p>
</blockquote>
<p>Django有一个内置的syndication feed框架，就像用<em>sites</em>框架创建站点地图（sitemap）一样，使用类似的方式（manner），你可以动态（dynamically）生成RSS或者Atom feeds。</p>
<p>在blog应用的目录下创建一个新文件命名为<em>feeds.py</em>。添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib.syndication.views <span class="hljs-keyword">import</span> Feed
<span class="hljs-keyword">from</span> django.template.defaultfilters <span class="hljs-keyword">import</span> truncatewords
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Post

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">LatestPostsFeed</span><span class="hljs-params">(Feed)</span>:</span>
    title = <span class="hljs-string">'My blog'</span>
    link = <span class="hljs-string">'/blog/'</span>
    description = <span class="hljs-string">'New posts of my blog.'</span>
    
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">items</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> Post.published.all()[:<span class="hljs-number">5</span>]
        
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">item_title</span><span class="hljs-params">(self, item)</span>:</span>
        <span class="hljs-keyword">return</span> item.title
        
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">item_description</span><span class="hljs-params">(self, item)</span>:</span>
        <span class="hljs-keyword">return</span> truncatewords(item.body, <span class="hljs-number">30</span>)
        </code></pre>
<p>首先，我们继承了syndication框架的<em>Feed</em>类创建了一个子类。其中的<em>title，link，description</em>属性各自对应RSS中的<code>&lt;title&gt;</code>,<code>&lt;link&gt;</code>,<code>&lt;description&gt;</code>元素。</p>
<p><em>items()</em>方法返回包含在feed中的对象。我们只给这个feed取回最新五个已发布的帖子。<em>item_title()</em>和<em>item_description()</em>方法接受<em>items()</em>返回的每个对象然后返回每个item各自的标题和描述信息。我们使用内置的<em>truncatewords</em>模板过滤器（template filter）构建帖子的描述信息并只保留最前面的30个单词。</p>
<p>现在，编辑blog应用下的<em>urls.py</em>文件，导入你刚创建的<em>LatestPostsFeed</em>，在新的URL模式（pattern）中实例化feed：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> .feeds <span class="hljs-keyword">import</span> LatestPostsFeed

urlpatterns = [
    <span class="hljs-comment"># ...</span>
    url(<span class="hljs-string">r'^feed/$'</span>, LatestPostsFeed(), name=<span class="hljs-string">'post_feed'</span>),
]</code></pre>
<p>在浏览器中转到 <a href="http://127.0.0.1:8000/blog/feed/" class="uri">http://127.0.0.1:8000/blog/feed/</a> 。你会看到最新的5个blog帖子的RSS feedincluding：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-meta">&lt;?xml version="1.0" encoding="utf-8"?&gt;</span>

<span class="hljs-tag">&lt;<span class="hljs-name">rss</span> <span class="hljs-attr">xmlns:atom</span>=<span class="hljs-string">"http://www.w3.org/2005/Atom"</span> <span class="hljs-attr">version</span>=<span class="hljs-string">"2.0"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">channel</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>My blog<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">link</span>&gt;</span>http://127.0.0.1:8000/blog/<span class="hljs-tag">&lt;/<span class="hljs-name">link</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">description</span>&gt;</span>New posts of my blog.<span class="hljs-tag">&lt;/<span class="hljs-name">description</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">atom:link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"http://127.0.0.1:8000/blog/feed/"</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">"self"</span>/&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">language</span>&gt;</span>en-us<span class="hljs-tag">&lt;/<span class="hljs-name">language</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">lastBuildDate</span>&gt;</span>Sun, 20 Sep 2015 20:40:55 -0000<span class="hljs-tag">&lt;/<span class="hljs-name">lastBuildDate</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">item</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span>Who was Django Reinhardt?<span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">link</span>&gt;</span>http://127.0.0.1:8000/blog/2015/09/20/who-was-django-reinhardt/<span class="hljs-tag">&lt;/<span class="hljs-name">link</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">description</span>&gt;</span>The Django web framework was named after the amazing jazz guitarist Django Reinhardt.<span class="hljs-tag">&lt;/<span class="hljs-name">description</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">guid</span>&gt;</span>http://127.0.0.1:8000/blog/2015/09/20/who-was-django-reinhardt/<span class="hljs-tag">&lt;/<span class="hljs-name">guid</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">item</span>&gt;</span> 
        ...
    <span class="hljs-tag">&lt;/<span class="hljs-name">channel</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">rss</span>&gt;</span></span></code></pre>
<p>如果你在一个RSS客户端中打开相同的URL，通过用户友好的接口你能看到你的feed。</p>
<p>最后一步是在blog的侧边栏（sitebar）添加一个feed订阅（subscription）链接。打开<em>blog/base.html</em>模板（template），在侧边栏（sitebar）的<em>div</em>中的帖子总数下添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "blog:post_feed" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Subscribe to my RSS feed<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 看下侧边栏（sitebar）。这个新链接将会带你去blog的feed：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-7.png" alt="django-3-7"></p>
<h3 id="使用solr和haystack添加一个搜索引擎">使用Solr和Haystack添加一个搜索引擎</h3>
<blockquote>
<p>译者注:终于到了这一节，之前自己按照本节一步步操作的走下来添加了一个搜索引擎但是并没有达到像本节中所说的效果，期间还出现了很多莫名其妙的错误，可以说添加的搜索引擎功能失败了，而且本节中的提到的工具版本过低，官网最新版本的操作步骤已经和本节中描述的不一样，本节的翻译就怕会带来更多的错误，大家有需要尽量去阅读下英文原版。另外，一些单词没有好的翻译我还是保留原文。</p>
</blockquote>
<p>现在，我们要为我们的blog添加搜索功能。Django ORM允许你使用<em>icontains</em>过滤器（filter）执行对大小写不敏感的查询操作。举个例子，你可以使用以下的查询方式来找到内容中包含单词<em>framework</em>的帖子：</p>
<pre><code class="hljs swift"><span class="hljs-type">Post</span>.objects.<span class="hljs-built_in">filter</span>(body__icontains='framework')</code></pre>
<p>但是，如果你想要更加强大的搜索功能，你需要使用合适的搜索引擎。我们准备使用<em>Solr</em>结合Django的方式为我们的blog构建一个搜索引擎。<em>Solr</em>是一个非常流行的开源搜索平台，它提供了全文检索（full text search），term boosting，hit highlighting，分类搜索（faceted search）以及动态聚集（clustering），还有其他更多的高级搜索特性。</p>
<p>为了在我们的项目中集成<em>Solr</em>，我们需要使用<em>Haystack</em>。<em>Haystack</em>是一个为多个搜索引擎提供抽象层工作的Django应用。它提供了一个非常类似于Django查询集（QuerySets）的简单的API。让我们开始安装和配置<em>Solr</em>和<em>Haystack</em>。</p>
<h3 id="安装solr">安装Solr</h3>
<p>你需要1.7或更高的Java运行环境来安装Solr。你可以在终端中输入<code>java -version</code>来检查你的java版本。下方的输出和你的输出可能有所出入，但是你必须保证安装的版本至少也要是1.7的：</p>
<pre><code class="hljs lisp">java version <span class="hljs-string">"1.7.0_25"</span>
Java(<span class="hljs-name">TM</span>) SE Runtime Environment (<span class="hljs-name">build</span> <span class="hljs-number">1.7</span>.<span class="hljs-number">0</span>_25-b15)
Java HotSpot(<span class="hljs-name">TM</span>) <span class="hljs-number">64</span>-Bit Server VM (<span class="hljs-name">build</span> <span class="hljs-number">23.25</span>-b01, mixed mode)</code></pre>
<p>如果你没有安装过Java或者版本低于要求版本，你可以在 <a href="http://www.oracle.com/technetwork/java/javase/downloads/index.html" class="uri">http://www.oracle.com/technetwork/java/javase/downloads/index.html</a> 下载Java。</p>
<p>检查Java版本后，从 <a href="http://archive.apache.org/dist/lucene/solr/" class="uri">http://archive.apache.org/dist/lucene/solr/</a> 下载<strong>4.10.4</strong>版本的Solr（译者注：请一定要下载这个版本，不然下面的操作无法进行！！）。解压下载的文件进入<em>Solr</em>安装路径下的<em>example</em>目录（也就是,<code>cd solr-4.10.4/example/</code>)。这个目录下包含了一个准备使用的<em>Solr</em>配置。在这个目录下，通过以下命令，用内置的Jetty web服务运行<em>Solr</em>：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">java</span> <span class="hljs-selector-tag">-jar</span> <span class="hljs-selector-tag">start</span><span class="hljs-selector-class">.jar</span></code></pre>
<p>打开你的浏览器，进入URL:<a href="http://127.0.0.1:8983/solr/" class="uri">http://127.0.0.1:8983/solr/</a> 。你会看到如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-8.png" alt="django-3-8"></p>
<p>以上是<em>Solr</em>的管理控制台。这个控制台向你展示了数据统计，允许你管理你的搜索后端，检测索引数据，并且执行查询操作。</p>
<h3 id="创建一个solr-core">创建一个Solr core</h3>
<p>Solr允许你隔离每一个core实例。每个Solr <strong>core</strong>是一个<strong>Lucene全文搜索引擎</strong>实例，连同一个Solr配置，一个数据架构（schema），以及其他要求的配置才能使用。Slor允许你动态地创建和管理cores。参考例子配置中包含了一个叫<em>collection1</em>的core。如果你点击<strong>Core Admin</strong>菜单栏， 你可以看到这个core的信息，如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-9.png" alt="django-3-9"></p>
<p>我们要为我们的blog应用创建一个core。首先，我们需要为我们的core创建文件结构。进入<em>solr-4.10.4/example/</em>目录下，创建一个新的目录命名为<em>blog</em>。然后在<em>blog</em>目录下创建空文件和目录，如下所示：</p>
<pre><code class="hljs haskell"><span class="hljs-title">blog</span>/ 
    <span class="hljs-class"><span class="hljs-keyword">data</span>/</span>
    conf/
    protwords.txt
    schema.xml
    solrconfig.xml
    stopwords.txt
    synonyms.txt
    lang/
    stopwords_en.txt
            </code></pre>
<p>在<em>solrconfig.xml</em>文件中添加如下XML代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-meta">&lt;?xml version="1.0" encoding="utf-8" ?&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">config</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">luceneMatchVersion</span>&gt;</span>LUCENE_36<span class="hljs-tag">&lt;/<span class="hljs-name">luceneMatchVersion</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">requestHandler</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"/select"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"solr.StandardRequestHandler"</span> <span class="hljs-attr">default</span>=<span class="hljs-string">"true"</span> /&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">requestHandler</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"/update"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"solr.UpdateRequestHandler"</span> /&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">requestHandler</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"/admin"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"solr.admin.AdminHandlers"</span> /&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">requestHandler</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"/admin/ping"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"solr.PingRequestHandler"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">lst</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"invariants"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">str</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"qt"</span>&gt;</span>search<span class="hljs-tag">&lt;/<span class="hljs-name">str</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">str</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"q"</span>&gt;</span>*:*<span class="hljs-tag">&lt;/<span class="hljs-name">str</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">lst</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">requestHandler</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">config</span>&gt;</span></span></code></pre>
<p>你还可以从本章的示例代码中拷贝该文件。这是一个最小的Solr配置。编辑<em>schema.xml</em>文件，加入如下XML代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-meta">&lt;?xml version="1.0" ?&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">schema</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"default"</span> <span class="hljs-attr">version</span>=<span class="hljs-string">"1.5"</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">schema</span>&gt;</span></span></code></pre>
<p>这是一个空的<strong>架构（schema）</strong>。这个架构（schema）定义了在搜索引擎中将被索引到的数据的字段和字段类型。之后我们要使用一个自定义的架构（schema）。</p>
<p>现在，点击<strong>Core Admin</strong>菜单栏再点击<strong>Add core</strong>按钮。你会看到如下图所示的一张表单，允许你为你的core指定信息：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-10.png" alt="django-3-10"></p>
<p>用以下数据填写表单：</p>
<ul>
<li>name: blog</li>
<li>instanceDir: blog</li>
<li>dataDir: data</li>
<li>config: solrconfig.xml</li>
<li>schema: schema.xml</li>
</ul>
<p><em>name</em>字段是你想给这个core起的名字。<em>instanceDir</em>字段是你的core的目录。<em>dataDir</em>是索引数据将要存放的目录，它位于<em>instanceDir</em>目录下面。<em>config</em>字段是你的<em>Solr</em> XML配置文件名。<em>schema</em>字段是你的<em>Solr</em> XML 数据架构（schema)文件名。</p>
<p>现在，点击<strong>Add Core</strong>按钮。如果你看到下图所示，说明你新的core已经成功的添加到Solr中：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-11.png" alt="django-3-11"></p>
<h3 id="安装haystack">安装Haystack</h3>
<p>为了在Django中使用Solr，我们还需要Haystack。使用下面的命令，通过pip渠道安装Haystack：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-haystack==<span class="hljs-number">2.4</span>.<span class="hljs-number">0</span></code></pre>
<p>Haystack能和一些搜索引擎后台交互。要使用Solr后端，你还需要安装<em>pysolr</em>模块。运行如下命令安装它：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> pysolr==<span class="hljs-number">3.3</span>.<span class="hljs-number">2</span></code></pre>
<p>在<em>django-haystack</em>和<em>pysolr</em>完成安装后，你还需要在你的项目中激活<em>Haystack</em>。打开<em>settings.py</em>文件，在<em>INSTALLED_APPS</em>设置中添加<em>haystack</em>，如下所示：</p>
<pre><code class="hljs makefile">INSTALLED_APPS = (
    <span class="hljs-comment"># ...</span>
    haystack', 
)</code></pre>
<p>你还需要为haystack定义搜索引擎后端。为此你要添加一个<em>HAYSTACK_CONNECTIONS</em>设置。在<em>settings.py</em>文件中添加如下内容：</p>
<pre><code class="hljs scala"><span class="hljs-type">HAYSTACK_CONNECTIONS</span> = {
    <span class="hljs-symbol">'defaul</span>t': {
        <span class="hljs-symbol">'ENGIN</span>E': <span class="hljs-symbol">'haystack</span>.backends.solr_backend.<span class="hljs-type">SolrEngine</span>',
        <span class="hljs-symbol">'UR</span>L': <span class="hljs-symbol">'http</span>:<span class="hljs-comment">//127.0.0.1:8983/solr/blog'</span>
    },
}</code></pre>
<p>要注意URL要指向我们的blog core。到此为止，Haystack已经安装好并且已经为使用Solr做好了准备。</p>
<h3 id="创建索引indexex">创建索引（indexex）</h3>
<p>现在，我们必须将我们想要存储在搜索引擎中的模型进行注册。Haystack的惯例是在你的应用中创建一个<em>search_indexes.py</em>文件，然后在该文件中注册你的模型（models）。在你的blog应用目录下创建一个新的文件命名为<em>search_indexes.py</em>，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> haystack <span class="hljs-keyword">import</span> indexes
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Post

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">PostIndex</span><span class="hljs-params">(indexes.SearchIndex, indexes.Indexable)</span>:</span>
    text = indexes.CharField(document=<span class="hljs-keyword">True</span>, use_template=<span class="hljs-keyword">True</span>)
    publish = indexes.DateTimeField(model_attr=<span class="hljs-string">'publish'</span>)
    
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_model</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> Post
        
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">index_queryset</span><span class="hljs-params">(self, using=None)</span>:</span>
        <span class="hljs-keyword">return</span> self.get_model().published.all()</code></pre>
<p>这是一个<em>Post</em>模型（model）的自定义<em>SearchIndex</em>。通过这个索引（index），我们告诉Haystack这个模型（model）中的哪些数据必须被搜索引擎编入索引。这个索引（index）是通过继承<em>indexes.SearchIndex</em>和<em>indexes.Indexable</em>构建的。每一个<em>SearchIndex</em>都需要它的其中一个字段拥有<code>document=True</code>。按照惯例，这个字段命名为<em>text</em>。这个字段是一个主要的搜索字段。通过使用<code>use_template=True</code>，我们告诉Haystack这个字段将会被渲染成一个数据模板（template）来构建document，它会被搜索引擎编入索引（index）。<em>publish</em>字段是一个日期字段也会被编入索引。我们通过<em>model_attr</em>参数来表明这个字段对应<em>Post</em>模型（model）的<em>publish</em>字段。这个字段将用 被索引的<em>Post</em>对象的<em>publish</em>字段的内容 索引。</p>
<p>额外的字段，像这个为搜索提供额外的过滤器（filters），是非常有用的。<em>get_model()</em>方法必须返回将储存在这个索引中的documents的模型（model）。<em>index_queryset()</em>方法返回将会被编入索引的对象的查询集（QuerySet）。请注意，我们只包含了发布状态的帖子。</p>
<p>现在，在blog应用的模板（templates）目录下创建目录和文件<em>search/indexes/blog/post_text.txt</em>，然后添加如下代码：</p>
<pre><code class="hljs clojure"><span>{</span><span>{</span> object.title <span>}</span><span>}</span>
<span>{</span><span>{</span> object.tags.all|join:<span class="hljs-string">", "</span> <span>}</span><span>}</span>
<span>{</span><span>{</span> object.body <span>}</span><span>}</span></code></pre>
<p>这是document模板（template）的默认路径，是给索引中的<em>text</em>字段使用的。Haystack使用应用名和模型（model）名来动态构建这个路径。每一次我们要索引一个对象，Haystack都会基于这个模板（template）构建一个document，之后在Solr的搜索引擎中索引这个document。</p>
<p>现在，我们已经有了一个自定义的搜索索引（index），我们需要创建合适的Solr架构（schema）。Solr的配置基于XML，所以我们必须为我们即将索引（index）的数据生成一个XML架构（schema）。非常幸运，haystack提供了一个基于我们的搜索索引（indexes），动态生成架构（schema）的方法。打开终端，运行以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">build_solr_schema</span></code></pre>
<p>你会看到一个XML输出。如果你看下生成的XML代码的底部，你会看到Haystack自动为你的<em>PostIndex</em>生成了字段：</p>
<pre><code class="hljs fsharp">&lt;field name=<span class="hljs-string">"text"</span> <span class="hljs-class"><span class="hljs-keyword">type</span></span>=<span class="hljs-string">"text_en"</span> indexed=<span class="hljs-string">"true"</span> stored=<span class="hljs-string">"true"</span> multiValued=<span class="hljs-string">"false"</span> /&gt;
&lt;field name=<span class="hljs-string">"publish"</span> <span class="hljs-class"><span class="hljs-keyword">type</span></span>=<span class="hljs-string">"date"</span> indexed=<span class="hljs-string">"true"</span> stored=<span class="hljs-string">"true"</span> multiValued=<span class="hljs-string">"false"</span> /&gt;
 </code></pre>
<p>从<code>&lt;?xml version="1.0"？&gt;</code>开始拷贝所有输出的XML内容直到最后的标签（tag）<code>&lt;/schema&gt;</code>，需要包含所有的标签（tags）。</p>
<p>这个XML架构（schema）是用来将数据做索引（index）到Solr中。粘贴这个新的架构（schema）到你的Solr安装路径下的<em>example</em>目录下的<em>blog/conf/schema.xml</em>文件中。<em>schema.xml</em>文件也被包含在本章的示例代码中，所以你可以直接从示例代码中复制出来使用。</p>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8983/solr/" class="uri">http://127.0.0.1:8983/solr/</a> 然后点击<strong>Core Admin</strong>菜单栏，再点击<strong>blog</strong> core，然后再点击<strong>Reload</strong>按钮：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-12.png" alt="django-3-12"></p>
<p>我们重新载入这个core确保<em>schema.xml</em>的改变生效。当core重新载入完毕，新的架构（schema）准备好索引（index）新数据。</p>
<h3 id="索引数据indexing-data">索引数据（Indexing data）</h3>
<p>让我们blog中的帖子编辑索引（index）到Solr中。打开终端，执行以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">rebuild_index</span></code></pre>
<p>你会看到如下警告：</p>
<pre><code class="hljs vbnet">WARNING: This will irreparably remove EVERYTHING <span class="hljs-keyword">from</span> your search index <span class="hljs-keyword">in</span> connection <span class="hljs-comment">'default'. Your choices after this are to restore from backups or rebuild via the ‘rebuild_index’ command.</span>
Are you sure you wish <span class="hljs-keyword">to</span> <span class="hljs-keyword">continue</span>? [y/N]</code></pre>
<p>输入y。Haystack将会清理搜索索引并且插入所有的发布状态的blog帖子。你会看到如下输出：</p>
<pre><code class="hljs basic"><span class="hljs-comment">Removing all documents from your index because you said so. All documents removed. Indexing 4 posts</span></code></pre>
<p>在浏览器中打开 <a href="http://127.0.0.1:8983/solr/#/blog" class="uri">http://127.0.0.1:8983/solr/#/blog</a> 。在**Statistics*下方，你会看到被编入索引（indexed）documents的数量，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-13.png" alt="django-3-13"></p>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8983/solr/#/blog/query" class="uri">http://127.0.0.1:8983/solr/#/blog/query</a> 。这是一个Solr提供的查询接口。点击<em>Execute query</em>按钮。默认的查询会请求你的core中所有被编入索引（indexde）的documents。你会看到一串带有这个查询结果的<em>JSON</em>输出。输出的documents如下所示：</p>
<pre><code class="hljs smalltalk">{
    <span class="hljs-comment">"id"</span>: <span class="hljs-comment">"blog.post.1"</span>,
    <span class="hljs-comment">"text"</span>: <span class="hljs-comment">"Who was Django Reinhardt?\njazz, music\nThe Django web framework was named after the amazing jazz guitarist Django Reinhardt."</span>,
    <span class="hljs-comment">"django_id"</span>: <span class="hljs-comment">"1"</span>,
    <span class="hljs-comment">"publish"</span>: <span class="hljs-comment">"2015-09-20T12:49:52Z"</span>,
    <span class="hljs-comment">"django_ct"</span>: <span class="hljs-comment">"blog.post"</span>
},</code></pre>
<p>这是每个帖子在搜索索引（index）中存储的数据。<em>text</em>字段包含了标题，通过逗号分隔的标签（tags），还有帖子的内容，这个字段是在我们之前定义的模板（template）上构建的。</p>
<p>你已经使用过<code>python manage.py rebuild_index</code>来删除索引（index）中的所有信息然后再次对documents进行索引（index）。为了不删除所有对象而更新你的索引（index），你可以使用<code>python manage.py update_index</code>。另外，你可以使用参数<code>--age=&lt;num_hours&gt;</code>来更新少量的对象。为了保证你的Solr索引更新，你可以为这个操作设置一个定时任务（Cron job）。</p>
<h3 id="创建一个搜索视图view">创建一个搜索视图（view）</h3>
<p>现在，我们要开始创建一个自定义视图（view）来允许我们的用户搜索帖子。首先，我们需要一个搜索表单（form）。编辑blog应用下的<em>forms.py</em>文件，加入以下表单：</p>
<pre><code class="hljs haskell"><span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-type">SearchForm</span>(<span class="hljs-title">forms</span>.<span class="hljs-type">Form</span>):
    query = forms.<span class="hljs-type">CharField</span>()</span></code></pre>
<p>我们会使用<em>query</em>字段来让用户引入搜索条件（terms）。编辑blog应用下的<em>views.py</em>文件，加入以下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> .forms <span class="hljs-keyword">import</span> EmailPostForm, CommentForm, SearchForm
<span class="hljs-keyword">from</span> haystack.query <span class="hljs-keyword">import</span> SearchQuerySet

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">post_search</span><span class="hljs-params">(request)</span>:</span>
    form = SearchForm()
    <span class="hljs-keyword">if</span> <span class="hljs-string">'query'</span> <span class="hljs-keyword">in</span> request.GET:
        form = SearchForm(request.GET)
        <span class="hljs-keyword">if</span> form.is_valid():
            cd = form.cleaned_data
            results = SearchQuerySet().models(Post).filter(content=cd[<span class="hljs-string">'query'</span>]).load_all()
            <span class="hljs-comment"># count total results</span>
            total_results = results.count()
            
    <span class="hljs-keyword">return</span> render(request, <span class="hljs-string">'blog/post/search.html'</span>,
            {<span class="hljs-string">'form'</span>: form,
            <span class="hljs-string">'cd'</span>: cd,
            <span class="hljs-string">'results'</span>: results,
            <span class="hljs-string">'total_results'</span>: total_results})
                  </code></pre>
<p>在这个视图（view）中，首先我们实例化了我们刚才创建的<em>SearchForm</em>.我们准备使用<em>GET</em>方法来提交这个表单（form），这样可以使URL结果中包含查询的参数。假设这个表单（form）已经被提交，我们将在<em>request.GET</em>字典中查找<em>query</em>参数。当表单（form）被提交后，我们通过提交的<em>GET</em>数据来实例化它，然后我们要检查传入的数据是否有效（valid）。如果这个表单是有效（valid）的，我们使用<em>SearchQuerySet</em>为所有被编入索引的并且主要内容中包含给予的查询内容的<em>Post</em>对象来执行一次搜索。<em>load_all()</em>方法会立刻加载所有在数据库中有关联的<em>Post</em>对象。通过这个方法，我们使用数据库对象填充搜索结果，避免当遍历结果访问对象数据时，每个对象访问数据库 （译者注：这话不太好翻译，看不懂的话可以看下原文）。最后，我们存储<em>total_results</em>变量中结果的总数并传递本地的变量作为上下文(context)来渲染一个模板（template）。</p>
<p>搜索视图（view）已经准备好了。我们还需要创建一个模板（template）来展示表单（form）和用户执行搜索后返回的结果。在<em>templates/blog/post/</em>目录下创建一个新的文件命名为<em>search.html</em>，添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "blog/base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Search</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> "query" <span class="hljs-keyword">in</span> request.GET <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Posts containing "</span><span class="hljs-template-variable"><span>{</span><span>{</span> cd.query <span>}</span><span>}</span></span><span class="xml">"<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span>Found </span><span class="hljs-template-variable"><span>{</span><span>{</span> total_results <span>}</span><span>}</span></span><span class="xml"> result</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_results|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span>
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> result <span class="hljs-keyword">in</span> results <span>%</span><span>}</span></span><span class="xml">
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">with</span></span> post=result.object <span>%</span><span>}</span></span><span class="xml">
                <span class="hljs-tag">&lt;<span class="hljs-name">h4</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> post.title <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">h4</span>&gt;</span>           
                </span><span class="hljs-template-variable"><span>{</span><span>{</span> post.body|<span class="hljs-name">truncatewords</span>:5 <span>}</span><span>}</span></span><span class="xml">
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endwith</span></span> <span>%</span><span>}</span></span><span class="xml">
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">empty</span></span> <span>%</span><span>}</span></span><span class="xml">
                <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>There are no results for your query.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">    
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "blog:post_search" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Search again<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Search for posts<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"get"</span>&gt;</span>
            </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
            <span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Search"</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>就像在搜索视图（view）中，我们做了区分如果这个表单（form）是基于<em>query</em>参数存在的情况下提交。在这个post提交前，我们展示了这个表单和一个提交按钮。当这个post被提交，我们就展示查询的操作结果，包含返回结果的总数和结果列表。每一个结果都是Solr返回和Haystack封装处理后的document。我们需要使用<em>result.object</em>来获取真实的和这个结果相关联的<em>Post</em>对象。</p>
<p>最后，编辑blog应用下的<em>urls.py</em>文件，添加以下URL模式：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^search/$'</span>, views.post_search, name=<span class="hljs-string">'post_search'</span>),</code></pre>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/blog/search/%E3%80%82%E4%BD%A0%E4%BC%9A%E7%9C%8B%E5%88%B0%E5%A6%82%E4%B8%8B%E5%9B%BE%E6%89%80%E7%A4%BA%E7%9A%84%E6%90%9C%E7%B4%A2%E8%A1%A8%E5%8D%95%EF%BC%88form" class="uri">http://127.0.0.1:8000/blog/search/。你会看到如下图所示的搜索表单（form</a>）：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-14.png" alt="django-3-14"></p>
<p>现在，输入一个查询条件然后点击<strong>Search</strong>按钮。你会看到查询搜索的结果，如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-3-15.png" alt="django-3-15"></p>
<p>如今，在你的项目中你已经构建了一个强大的搜索引擎，但这仅仅只是开始，还有更多丰富的功能可以通过<em>Solr</em>和Haystack做到。Haystack包含视图（views），表单（forms）以及搜索引擎的高级功能。你可以在 <a href="http://django-haystack.readthedocs.org/en/latest/" class="uri">http://django-haystack.readthedocs.org/en/latest/</a> 页面上阅读Haystack文档。</p>
<p>通过自定义架构（schema），Solr搜索引擎可以适配各种需求。你可以结合分析仪，断词，和令牌过滤器，这些是在索引或搜索的时间执行，为你的网站内容提供更准确的搜索。 你可以在 <a href="https://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters" class="uri">https://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters</a> 看到所有的可能性。</p>
<h3 id="总结">总结</h3>
<p>在这一章中，你学习了如何创建自定义的Django模板标签（template tags）和过滤器（filters），提供给模板（template）实现一些自定义的功能。你还为搜索引擎创建了一个站点地图（sitemap），爬取你的站点以及一个RSS feed 给用户来订阅。你还通过在项目中集成Slor和Haystack为blog应用构建了一个搜索引擎。</p>
<p>在下一章中，你将会学习到通过使用Django认证框架，如何构建一个社交网站，创建自定义的用户画像，以及创建社交认证。</p>
</div>
