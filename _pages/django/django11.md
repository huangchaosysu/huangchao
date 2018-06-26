---
layout: single
permalink: /django/example2/
title: "Django By Example 第二章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第二章">第二章</h1>
<h2 id="用高级特性来增强你的blog">用高级特性来增强你的blog</h2>
<p>在上一章中，你创建了一个基础的博客应用。现在你将利用一些高级的特性例如通过email来分享帖子，添加评论，给帖子打上tag，检索出相似的帖子等将它改造成为一个功能更加齐全的博客。在本章中，你将会学习以下几点：</p>
<ul>
<li>通过Django发送email</li>
<li>在视图（views）中创建并操作表单</li>
<li>通过模型（models）创建表单</li>
<li>集成第三方应用</li>
<li>构建复杂的查询集（QuerySets)</li>
</ul>
<h3 id="通过email分享帖子">通过email分享帖子</h3>
<p>首先，我们会允许用户通过发送邮件来分享他们的帖子。让我们花费一小会时间来想下，根据在上一章中学到的知识，你该如何使用views，urls和templates来创建这个功能。现在，核对一下你需要哪些才能允许你的用户通过邮件来发送帖子。你需要做到以下几点：</p>
<ul>
<li>给用户创建一个表单来填写他们的姓名，email，收件人以及评论，评论不是必选项。</li>
<li>在<em>views.py</em>文件中创建一个视图（view）来操作发布的数据和发送email</li>
<li>在blog应用的<em>urls.py</em>中为新的视图（view）添加一个URL模式</li>
<li>创建一个模板（template）来展示这个表单</li>
</ul>
<h3 id="使用django创建表单">使用Django创建表单</h3>
<p>让我们开始创建一个表单来分享帖子。Django有一个内置的表单框架允许你通过简单的方式来创建表单。这个表单框架允许你定义你的表单字段，指定这些字段必须展示的方式，以及指定这些字段如何验证输入的数据。Django表单框架还提供了一种灵活的方式来渲染表单以及操作数据。</p>
<p>Django提供了两个可以创建表单的基本类：</p>
<ul>
<li>Form: 允许你创建一个标准表单</li>
<li>ModelForm: 允许你创建一个可用于创建或者更新model实例的表单</li>
</ul>
<p>首先，在你blog应用的目录下创建一个<em>forms.py</em>文件，输入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">EmailPostForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    name <span class="op">=</span> forms.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">25</span></span>)
    email <span class="op">=</span> forms.EmailField()
    to <span class="op">=</span> forms.EmailField()
    comments <span class="op">=</span> forms.CharField(required<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>,
                                         widget<span class="op">=</span>forms.Textarea)</code></pre></div>
<p>这是你的第一个Django表单。看下代码：我们已经创建了一个继承了基础<em>Form</em>类的表单。我们使用不同的字段类型以使Django有依据的来验证字段。</p>
<blockquote>
<p>表单可以存在你的Django项目的任何地方，但按照惯例将它们放在每一个应用下面的<em>forms.py</em>文件中</p>
</blockquote>
<p><em>name</em>字段是一个<em>CharField</em>。这种类型的字段被渲染成<code>&lt;input type=“text”&gt;</code>HTML元素。每种字段类型都有默认的控件来确定它在HTML中的展示形式。通过改变控件的属性可以重写默认的控件。在comment字段中，我们使用<code>&lt;textarea&gt;&lt;/textarea&gt;</code>HTML元素而不是使用默认的<code>&lt;input&gt;</code>元素来显示它。</p>
<p>字段验证取决于字段类型。例如，<em>email</em>和<em>to</em>字段是<em>EmailField</em>,这两个字段都需要一个有效的email地址，否则字段验证将会抛出一个<em>forms.ValidationError</em>异常导致表单验证不通过。在表单验证的时候其他的参数也会被考虑进来：我们将name字段定义为一个最大长度为25的字符串；通过设置<code>required=False</code>让comments的字段可选。所有这些也会被考虑到字段验证中去。目前我们在表单中使用的这些字段类型只是Django支持的表单字段的一部分。要查看更多可利用的表单字段，你可以访问：<a href="https://docs.djangoproject.com/en/1.8/ref/forms/fields/" class="uri">https://docs.djangoproject.com/en/1.8/ref/forms/fields/</a></p>
<h3 id="在视图views中操作表单">在视图（views）中操作表单</h3>
<p>当表单成功提交后你必须创建一个新的视图（views）来操作表单和发送email。编辑blog应用下的<em>views.py</em>文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> EmailPostForm

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_share</span><span class="hljs-params">(request, post_id)</span>:</span>
    <span class="co"><span class="hljs-comment"># retrieve post by id</span></span>
    post <span class="op">=</span> get_object_or_404(Post, <span class="bu">id</span><span class="op">=</span>post_id, status<span class="op">=</span><span class="st"><span class="hljs-string">'published'</span></span>)
    
    <span class="cf"><span class="hljs-keyword">if</span></span> request.method <span class="op">==</span> <span class="st"><span class="hljs-string">'POST'</span></span>:
        <span class="co"><span class="hljs-comment"># Form was submitted</span></span>
        form <span class="op">=</span> EmailPostForm(request.POST)
        <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
            <span class="co"><span class="hljs-comment"># Form fields passed validation</span></span>
            cd <span class="op">=</span> form.cleaned_data
            <span class="co"><span class="hljs-comment"># ... send email</span></span>
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        form <span class="op">=</span> EmailPostform()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'blog/post/share.html'</span></span>, {<span class="st"><span class="hljs-string">'post'</span></span>: post,
                                                               <span class="co"><span class="hljs-string">'form: form})</span></span></code></pre></div>
<p>该视图（view）完成了以下工作：</p>
<ul>
<li>我们定义了<em>post_share</em>视图，参数为<em>request</em>对象和<em>post_id</em>。</li>
<li>我们使用<em>get_object_or_404</em>快捷方法通过ID获取对应的帖子，并且确保获取的帖子有一个<em>published</em>状态。</li>
<li>我们使用同一个视图（view）来展示初始表单和处理提交后的数据。我们会区别被提交的表单和不基于这次请求方法的表单。我们将使用<em>POST</em>来提交表单。如果我们得到一个<em>GET</em>请求，一个空的表单必须显示，而如果我们得到一个<em>POST</em>请求，则表单需要提交和处理。因此，我们使用<code>request.method == 'POST'</code>来区分这两种场景。</li>
</ul>
<p>下面是展示和操作表单的过程：</p>
<ul>
<li><p>1.通过GET请求视图（view）被初始加载后，我们创建一个新的表单实例，用来在模板（template）中显示一个空的表单：</p>
<pre><code class="hljs ini"><span class="hljs-attr">form</span> = EmailPostForm()</code></pre></li>
<li><p>2.当用户填写好了表单并通过<em>POST</em>提交表单。之后，我们会用保存在<code>request.POST</code>中提交的数据创建一个表单实例。</p>
<pre><code class="hljs python"><span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:
   <span class="hljs-comment"># Form was submitted</span>
   form = EmailPostForm(request.POST)</code></pre></li>
<li>3.在以上步骤之后，我们使用表单的<em>is_valid()</em>方法来验证提交的数据。这个方法会验证表单引进的数据，如果所有的字段都是有效数据，将会返回<em>True</em>。一旦有任何一个字段是无效的数据，<em>is_valid()</em>就会返回<em>False</em>。你可以通过访问 <code>form.errors</code>来查看所有验证错误的列表。</li>
<li>4如果表单数据验证没有通过，我们会再次使用提交的数据在模板（template）中渲染表单。我们会在模板（template）中显示验证错误的提示。</li>
<li><p>5.如果表单数据验证通过，我们通过访问<code>form.cleaned_data</code>获取验证过的数据。这个属性是一个表单字段和值的字典。</p></li>
</ul>
<blockquote>
<p>如果你的表单数据没有通过验证，<em>cleaned_data</em>只会包含验证通过的字段</p>
</blockquote>
<p>现在，你需要学习如何使用Django来发送email,把所有的事情结合起来。</p>
<h2 id="使用django发送email">使用Django发送email</h2>
<p>使用Django发送email非常简单。首先，你需要有一个本地的SMTP服务或者通过在你项目的settings.py文件中添加以下设置去定义一个外部SMTP服务器的配置：</p>
<ul>
<li>EMAIL_HOST: SMTP服务地址。默认本地。</li>
<li>EMAIL_POSR: SMATP服务端口，默认25。</li>
<li>EMAIL_HOST_USER: SMTP服务的用户名。</li>
<li>EMAIL_HOST_PASSWORD: SMTP服务的密码。</li>
<li>EMAIL_USE_TLS: 是否使用TLS加密连接。</li>
<li>EMAIL_USE_SSL: 是否使用隐式的SSL加密连接。</li>
</ul>
<p>如果你没有本地SMTP服务，你可以使用你的email服务供应商提供的SMTP服务。下面提供了一个简单的例子展示如何通过使用Google账户的Gmail服务来发送email：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">EMAIL_HOST <span class="op">=</span> <span class="st"><span class="hljs-string">'smtp.gmail.com'</span></span>
EMAIL_HOST_USER <span class="op">=</span> <span class="st"><span class="hljs-string">'your_account@gmail.com'</span></span>
EMAIL_HOST_PASSWORD <span class="op">=</span> <span class="st"><span class="hljs-string">'your_password'</span></span>
EMAIL_PORT <span class="op">=</span> <span class="dv"><span class="hljs-number">587</span></span>
EMAIL_USE_TLS <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span></code></pre></div>
<p>运行命令python manage.py shell来打开Python shell，发送一封email如下所示：</p>
<pre class="shell"><code class="hljs python"><span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> django.core.mail <span class="hljs-keyword">import</span> send_mail
<span class="hljs-meta">&gt;&gt;&gt; </span>send_mail(<span class="hljs-string">'Django mail'</span>, <span class="hljs-string">'This e-mail was sent with Django.'</span>,<span class="hljs-string">'your_account@gmail.com'</span>, [<span class="hljs-string">'your_account@gmail.com'</span>], fail_silently=<span class="hljs-keyword">False</span>)</code></pre>
<p><em>send_mail()</em>方法需要这些参数：邮件主题，内容，发送人以及一个收件人的列表。通过设置可选参数<code>fail_silently=False</code>，我们告诉这个方法如果email没有发送成功那么需要抛出一个异常。如果你看到输出是<em>1</em>，证明你的email发送成功了。如果你使用之前的配置用Gmail来发送邮件，你可能需要去 <a href="https://www.google.com/settings/security/lesssecureapps" class="uri">https://www.google.com/settings/security/lesssecureapps</a> 去开通一下低安全级别应用的权限。(译者注：练习时老老实实用QQ邮箱吧）</p>
<p>现在，我们要将以上代码添加到我们的视图（view）中。在blog应用下的<em>views.py</em>文件中编辑<em>post_share</em>视图（view）如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.mail <span class="im"><span class="hljs-keyword">import</span></span> send_mail

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_share</span><span class="hljs-params">(request, post_id)</span>:</span>
    <span class="co"><span class="hljs-comment"># Retrieve post by id</span></span>
    post <span class="op">=</span> get_object_or_404(Post, <span class="bu">id</span><span class="op">=</span>post_id, status<span class="op">=</span><span class="st"><span class="hljs-string">'published'</span></span>)
    sent <span class="op">=</span> <span class="va"><span class="hljs-keyword">False</span></span>
    <span class="cf"><span class="hljs-keyword">if</span></span> request.method <span class="op">==</span> <span class="st"><span class="hljs-string">'POST'</span></span>:
        <span class="co"><span class="hljs-comment"># Form was submitted</span></span>
        form <span class="op">=</span> EmailPostForm(request.POST)
        <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
            <span class="co"><span class="hljs-comment"># Form fields passed validation</span></span>
            cd <span class="op">=</span> form.cleaned_data
            post_url <span class="op">=</span> request.build_absolute_uri(
                                    post.get_absolute_url())
            subject <span class="op">=</span> <span class="st"><span class="hljs-string">'{} ({}) recommends you reading "{}"'</span></span>.<span class="bu">format</span>(cd[<span class="st"><span class="hljs-string">'name'</span></span>], cd[<span class="st"><span class="hljs-string">'email'</span></span>], post.title)
            message <span class="op">=</span> <span class="st"><span class="hljs-string">'Read "{}" at {}</span></span><span class="ch"><span class="hljs-string">\n\n</span></span><span class="st"><span class="hljs-string">{}</span></span><span class="ch"><span class="hljs-string">\'</span></span><span class="st"><span class="hljs-string">s comments: {}'</span></span>.<span class="bu">format</span>(post.title, post_url, cd[<span class="st"><span class="hljs-string">'name'</span></span>], cd[<span class="st"><span class="hljs-string">'comments'</span></span>])
            send_mail(subject, message, <span class="st"><span class="hljs-string">'admin@myblog.com'</span></span>,[cd[<span class="st"><span class="hljs-string">'to'</span></span>]])
            sent <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        form <span class="op">=</span> EmailPostForm()
        
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'blog/post/share.html'</span></span>, {<span class="st"><span class="hljs-string">'post'</span></span>: post,
                                                    <span class="co"><span class="hljs-string">'form'</span></span>: form,
                                                    <span class="co"><span class="hljs-string">'sent'</span></span>: sent})</code></pre></div>
<p>请注意，我们声明了一个<em>sent</em>变量并且当帖子被成功发送时赋予它<em>True</em>。当表单成功提交的时候，我们之后将在模板（template）中使用这个变量显示一条成功提示。由于我们需要在email中包含帖子的超链接，所以我们通过使用<code>post.get_absolute_url()</code>方法来获取到帖子的绝对路径。我们将这个绝对路径作为<code>request.build_absolute_uri()</code>的输入值来构建一个完整的包含了HTTP schema和主机名的url。我们通过使用验证过的表单数据来构建email的主题和消息内容并最终给表单to字段中包含的所有email地址发送email。</p>
<p>现在你的视图（view）已经完成了，别忘记为它去添加一个新的URL模式。打开你的blog应用下的<em>urls.py</em>文件添加<em>post_share</em>的URL模式如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> [
<span class="co"><span class="hljs-comment"># ...</span></span>
url(<span class="vs"><span class="hljs-string">r'^(?P&lt;post_id&gt;\d+)/share/$'</span></span>, views.post_share,
    name<span class="op">=</span><span class="st"><span class="hljs-string">'post_share'</span></span>),
]</code></pre></div>
<h2 id="在模板templates中渲染表单">在模板（templates）中渲染表单</h2>
<p>在通过创建表单，编写视图（view）以及添加URL模式后，我们就只剩下为这个视图（view）添加模板（tempalte）了。在<code>blog/templates/blog/post/</code>目录下创建一个新的文件并命名为<em>share.html</em>。在该文件中添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% extends "blog/base.html" %}

{% block title %}Share a post{% endblock %}

{% block content %}
  {% if sent %}
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>E-mail successfully sent<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
      "{{ post.title }}" was successfully sent to {{ cd.to }}.
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  {% else %}
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Share "{{ post.title }}" by e-mail<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"."</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      {{ form.as_p }}
      {% csrf_token %}
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Send e-mail"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
  {% endif %}
{% endblock %}</code></pre></div>
<p>这个模板（tempalte）专门用来显示一个表单或一条成功提示信息。如你所见，我们创建的HTML表单元素里面表明了它必须通过POST方法提交：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span></span></code></pre>
<p>接下来我们要包含真实的表单实例。我们告诉Django用<code>as_p</code>方法利用HTML的<code>&lt;p&gt;</code>元素来渲染它的字段。我们也可以使用<code>as_ul</code>利用无序列表来渲染表单或者使用<code>as_table</code>利用HTML表格来渲染。如果我们想要逐一渲染每一个字段，我们可以迭代字段。例如下方的例子：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% for field in form %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span></span>
    {{ field.errors }}
    {{ field.label_tag }} {{ field }}
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
{% endfor %}</code></pre></div>
<p><code>{% csrf_token %}</code>模板（tempalte）标签（tag）引进了可以避开<em>Cross-Site request forgery(CSRF)</em>攻击的自动生成的令牌，这是一个隐藏的字段。这些攻击由恶意的站点或者可以在你的站点中为用户执行恶意行为的程序组成。通过访问 <a href="https://en.wikipedia.org/wiki/Cross-site_request_forgery%E4%BD%A0%E5%8F%AF%E4%BB%A5%E6%89%BE%E5%88%B0%E6%9B%B4%E5%A4%9A%E7%9A%84%E4%BF%A1%E6%81%AF" class="uri">https://en.wikipedia.org/wiki/Cross-site_request_forgery你可以找到更多的信息</a> 。</p>
<p>上述的标签（tag）生成的隐藏字段就像下面一样：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">'hidden'</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">name</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">'csrfmiddlewaretoken'</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">'26JjKo2lcEtYkGoV9z4XmJIEHLXN5LDR'</span></span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span></code></pre></div>
<blockquote>
<p>默认情况下，Django在所有的POST请求中都会检查CSRF标记（token）。请记住要在所有使用POST方法提交的表单中包含<em>csrf_token</em>标签。<strong>（译者注：当然你也可以关闭这个检查，注释掉app_list中的csrf应用即可，我就是这么做的，因为我懒）</strong></p>
</blockquote>
<p>编辑你的<em>blog/post/detail.html</em>模板（template），在<code>{{ post.body|linebreaks }}</code>变量后面添加如下的链接来分享帖子的URL：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"{% url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">blog:post_share"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">post.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">%}"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    Share this post
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>请记住，我们通过使用Django提供的<code>{% url %}</code>模板（template）标签（tag）来动态的生成URL。我们以<em>blog</em>为命名空间，以<em>post_share</em>为URL，同时传递帖子ID作为参数来构建绝对的URL。</p>
<p>现在，通过<code>python manage.py runserver</code>命令来启动开发服务器，在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 。点击任意一个帖子标题查看详情页面。在帖子内容的下方，你会看到我们刚刚添加的链接，如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-1.png" alt="django-2-1"></p>
<p>点击<strong>Share this post</strong>,你会看到包含通过email分享帖子的表单的页面。看上去如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-2.png" alt="django-2-2"></p>
<p>这个表单的CSS样式被包含在示例代码中的 static/css/blog.css文件中。当你点击<strong>Send e-mail</strong>按钮，这个表单会提交并验证。如果所有的字段都通过了验证，你会得到一条成功信息如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-3.png" alt="django-2-3"></p>
<p>如果你输入了无效数据，你会看到表单被再次渲染，并展示出验证错误信息，如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-4.png" alt="django-2-4"></p>
<h2 id="创建一个评论系统">创建一个评论系统</h2>
<p>现在我们准备为blog创建一个评论系统，这样用户可以在帖子上进行评论。需要做到以下几点来创建一个评论系统：</p>
<ul>
<li>创建一个模型（model）用来保存评论</li>
<li>创建一个表单用来提交评论并且验证输入的数据</li>
<li>添加一个视图（view）来处理表单和保存新的评论到数据库中</li>
<li>编辑帖子详情模板（template）来展示评论列表以及用来添加新评论的表单</li>
</ul>
<p>首先，让我们创建一个模型（model）来存储评论。打开你的blog应用下的<em>models.py</em>文件添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Comment</span><span class="hljs-params">(models.Model)</span>:</span>
    post <span class="op">=</span> models.ForeignKey(Post, related_name<span class="op">=</span><span class="st"><span class="hljs-string">'comments'</span></span>)
    name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">80</span></span>)
    email <span class="op">=</span> models.EmailField()
    body <span class="op">=</span> models.TextField()
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    active <span class="op">=</span> models.BooleanField(default<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'created'</span></span>,)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'Comment by {} on {}'</span></span>.<span class="bu">format</span>(<span class="va">self</span>.name, <span class="va">self</span>.post)</code></pre></div>
<p>以上就是我们的<em>Comment</em>模型（model）。它包含了一个外键将一个单独的帖子和评论关联起来。在<em>Comment</em>模型（model）中定义多对一（many-to-one）的关系是因为每一条评论只能在一个帖子下生成，而每一个帖子又可能包含多个评论。<em>related_name</em>属性允许我们给这个属性命名，这样我们就可以利用这个关系从相关联的对象反向定位到这个对象。定义好这个之后，我们通过使用 <code>comment.post</code>就可以从一条评论来取到对应的帖子，以及通过使用<code>post.comments.all()</code>来取回一个帖子所有的评论。如果你没有定义<em>related_name</em>属性，Django会使用这个模型（model）的名称加上*_set*（在这里是：comment_set）来命名从相关联的对象反向定位到这个对象的manager。</p>
<p>访问https://docs.djangoproject.com/en/1.8/topics/db/examples/many_to_one/, 你可以学习更多关于多对一的关系。</p>
<p>我们用一个<em>active</em>布尔字段用来手动禁用那些不合适的评论。默认情况下，我们根据<em>created</em>字段，对评论按时间顺序进行排序。</p>
<p>你刚创建的这个新的<em>Comment</em>模型（model）并没有同步到数据库中。运行以下命令生成一个新的反映了新模型（model）创建的数据迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span> <span class="hljs-selector-tag">blog</span></code></pre>
<p>你会看到如下输出：</p>
<pre class="shell"><code class="hljs groovy">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'blog'</span>:
  <span class="hljs-number">0002</span>_comment.<span class="hljs-string">py:</span>
    - Create model Comment</code></pre>
<p>Django在blog应用下的<em>migrations/</em>目录中生成了一个<em>0002_comment.py</em>文件。现在你需要创建一个关联数据库模式并且将这些改变应用到数据库中。运行以下命令来执行已经存在的数据迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会获取以下输出：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">blog</span><span class="hljs-selector-class">.0002_comment</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>我们刚刚创建的数据迁移已经被执行，现在一张<em>blog_comment</em>表已经存在数据库中。</p>
<p>现在，我们可以添加我们新的模型（model）到管理站点中并通过简单的接口来管理评论。打开blog应用下的<em>admin.py</em>文件，添加comment model的导入,添加如下内容：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Post, Comment

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CommentAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> (<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>, <span class="st"><span class="hljs-string">'post'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'active'</span></span>)
    list_filter <span class="op">=</span> (<span class="st"><span class="hljs-string">'active'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>)
    search_fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>, <span class="st"><span class="hljs-string">'body'</span></span>)
admin.site.register(Comment, CommentAdmin)</code></pre></div>
<p>运行命令<code>python manage.py runserver</code>来启动开发服务器然后在浏览器中打开 <a href="http://127.0.0.1:8000/admin/" class="uri">http://127.0.0.1:8000/admin/</a> 。你会看到新的模型（model）在<strong>Blog</strong>区域中出现，如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-5.png" alt="django-2-5"></p>
<p>我们的模型（model）现在已经被注册到了管理站点，这样我们就可以使用简单的接口来管理评论实例。</p>
<h2 id="通过模型models创建表单">通过模型（models）创建表单</h2>
<p>我们仍然需要构建一个表单让我们的用户在blog帖子下进行评论。请记住，Django有两个用来创建表单的基础类：<em>Form</em>和<em>ModelForm</em>。你先前已经使用过第一个让用户通过email来分享帖子。在当前的例子中，你将需要使用<em>ModelForm</em>，因为你必须通过你的<em>Comment</em>模型（model）动态的创建表单。编辑blog应用下的<em>forms.py</em>，添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Comment

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CommentForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Comment
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>, <span class="st"><span class="hljs-string">'body'</span></span>)</code></pre></div>
<p>根据模型（model）创建表单，我们只需要在这个表单的<em>Meta</em>类里表明使用哪个模型（model）来构建表单。Django将会解析model并为我们动态的创建表单。每一种模型（model）字段类型都有对应的默认表单字段类型。表单验证时会考虑到我们定义模型（model）字段的方式。Django为模型（model）中包含的每个字段都创建了表单字段。然而，使用<em>fields</em> 列表你可以明确的告诉框架你想在你的表单中包含哪些字段，或者使用<em>exclude</em> 列表定义你想排除在外的那些字段。对于我们的<em>CommentForm</em>来说,我们在表单中只需要<em>name</em>,<em>email</em>,和<em>body</em>字段，因为我们只需要用到这3个字段让我们的用户来填写。</p>
<h2 id="在视图views中操作modelforms">在视图（views）中操作<em>ModelForms</em></h2>
<p>为了能更简单的处理它，我们会使用帖子的详情视图（view）来实例化表单。编辑<em>views.py</em>文件<strong>(注: 原文此处有错，应为 <code>views.py</code>)</strong>，导入<em>Comment</em>模型（model）和<em>CommentForm</em>表单，并且修改<em>post_detail</em>视图（view）如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Post, Comment
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> EmailPostForm, CommentForm

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_detail</span><span class="hljs-params">(request, year, month, day, post)</span>:</span>
    post <span class="op">=</span> get_object_or_404(Post, slug<span class="op">=</span>post,
                                   status<span class="op">=</span><span class="st"><span class="hljs-string">'published'</span></span>,
                                   publish__year<span class="op">=</span>year,
                                   publish__month<span class="op">=</span>month,
                                   publish__day<span class="op">=</span>day)
    <span class="co"><span class="hljs-comment"># List of active comments for this post</span></span>
    comments <span class="op">=</span> post.comments.<span class="bu">filter</span>(active<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    new_comment <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span> <span class="co"><span class="hljs-comment">#原文没有这行，但是经测试必须有这行</span></span>
        
    <span class="cf"><span class="hljs-keyword">if</span></span> request.method <span class="op">==</span> <span class="st"><span class="hljs-string">'POST'</span></span>:
        <span class="co"><span class="hljs-comment"># A comment was posted</span></span>
        comment_form <span class="op">=</span> CommentForm(data<span class="op">=</span>request.POST)
        <span class="cf"><span class="hljs-keyword">if</span></span> comment_form.is_valid():
            <span class="co"><span class="hljs-comment"># Create Comment object but don't save to database yet</span></span>
            new_comment <span class="op">=</span> comment_form.save(commit<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>)
            <span class="co"><span class="hljs-comment"># Assign the current post to the comment</span></span>
            new_comment.post <span class="op">=</span> post
            <span class="co"><span class="hljs-comment"># Save the comment to the database</span></span>
            new_comment.save()
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        comment_form <span class="op">=</span> CommentForm()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                  <span class="st"><span class="hljs-string">'blog/post/detail.html'</span></span>,
                  {<span class="st"><span class="hljs-string">'post'</span></span>: post,
                  <span class="co"><span class="hljs-string">'comments'</span></span>: comments, 
                  <span class="co"><span class="hljs-string">'new_comment'</span></span>: new_comment,
                  <span class="co"><span class="hljs-string">'comment_form'</span></span>: comment_form})</code></pre></div>
<p>让我们来回顾一下我们刚才对视图（view）添加了哪些操作。我们使用<em>post_detail</em>视图（view）来显示帖子和该帖子的评论。我们添加了一个查询集（QuerySet）来获取这个帖子所有有效的评论：</p>
<pre><code class="hljs ini"><span class="hljs-attr">comments</span> = post.comments.filter(active=<span class="hljs-literal">True</span>)</code></pre>
<p>我们从<em>post</em>对象开始构建这个查询集（QuerySet）。我们使用关联对象的manager，这个manager是我们在<em>Comment</em> 模型（model）中使用<em>related_name</em>关系属性为<em>comments</em>定义的。<br>
我们还在这个视图（view）中让我们的用户添加一条新的评论。因此，如果这个视图（view）是通过GET请求被加载的，那么我们用<code>comment_fomr = commentForm()</code>来创建一个表单实例。如果是通过POST请求，我们使用提交的数据并且用<em>is_valid()</em>方法验证这些数据去实例化表单。如果这个表单是无效的，我们会用验证错误信息渲染模板（template）。如果表单通过验证，我们会做以下的操作：</p>
<ul>
<li>1.我们通过调用这个表单的<em>save()</em>方法创建一个新的<em>Comment</em>对象，如下所示：</li>
</ul>
<p><code>new_comment = comment_form.save(commit=False)</code></p>
<p><em>Save()</em>方法创建了一个表单链接的model的实例，并将它保存到数据库中。如果你调用这个方法时设置<code>commit=False</code>，你创建的模型（model）实例不会即时保存到数据库中。当你想在最终保存之前修改这个model对象会非常方便，我们接下来将做这一步骤。<em>save()</em>方法是给<em>ModelForm</em>用的，而不是给<em>Form</em>实例用的，因为<em>Form</em>实例没有关联上任何模型（model）。</p>
<ul>
<li><p>2.我们为我们刚创建的评论分配一个帖子：</p>
<pre><code class="hljs">new_comment.post = post</code></pre>
<p>通过以上动作，我们指定新的评论是属于这篇给定的帖子。</p></li>
<li><p>3.最后，我们用下面的代码将新的评论保存到数据库中：</p>
<pre><code class="hljs fortran">new_comment.<span class="hljs-keyword">save</span>()</code></pre></li>
</ul>
<p>我们的视图（view）已经准备好显示和处理新的评论了。</p>
<h2 id="在帖子详情模板template中添加评论">在帖子详情模板（template）中添加评论</h2>
<p>我们为帖子创建了一个管理评论的功能。现在我们需要修改我们的<em>post_detail.html</em>模板（template）来适应这个功能，通过做到以下步骤：</p>
<ul>
<li>显示这篇帖子的评论总数</li>
<li>显示评论的列表</li>
<li>显示一个表单给用户来添加新的评论</li>
</ul>
<p>首先，我们来添加评论的总数。打开<em>blog_detail.html</em>模板（template）在<em>content</em>区块中添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% with comments.count as total_comments %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>
    {{ total_comments }} comment{{ total_comments|pluralize }}
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
{% endwith %}</code></pre></div>
<p>在模板（template）中我们使用Django ORM执行<code>comments.count()</code> 查询集（QuerySet）。注意，Django模板（template）语言中不使用圆括号来调用方法。<code>{% with %}</code> 标签（tag）允许我们分配一个值给新的变量，这个变量可以一直使用直到遇到<code>{% endwith %}</code>标签（tag）。</p>
<blockquote>
<p><code>{% with %}</code>模板（template）标签（tag）是非常有用的，可以避免直接操作数据库或避免多次调用花费较多的方法。</p>
</blockquote>
<p>根据<em>total_comments</em>的值，我们使用<em>pluralize </em>模板（template）过滤器（filter）为单词<em>comment</em>显示复数后缀。模板（Template）过滤器（filters）获取到他们输入的变量值，返回计算后的值。我们将会在<em>第三章 扩展你的博客应用</em>中讨论更多的模板过滤器（tempalte filters）。</p>
<p><em>pluralize</em>模板（template）过滤器（filter）在值不为1时，会在值的末尾显示一个"s"。之前的文本将会被渲染成类似：<em>0 comments</em>, <em>1 comment</em> 或者 <em>N comments</em>。Django内置大量的模板（template）标签（tags）和过滤器（filters）来帮助你以你想要的方式来显示信息。</p>
<p>现在，让我们加入评论列表。在模板（template）中之前的代码后面加入以下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% for comment in comments %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"comment"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"info"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      Comment {{ forloop.counter }} by {{ comment.name }}
      {{ comment.created }}
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
    {{ comment.body|linebreaks }}
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
{% empty %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>There are no comments yet.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
{% endfor %}</code></pre></div>
<p>我们使用<code>{% for %}</code>模板（template）标签（tag）来循环所有的评论。如果<em>comments</em>列为空我们会显示一个默认的信息，告诉我们的用户这篇帖子还没有任何评论。我们使用 <code>{{ forloop.counter }}</code>变量来枚举所有的评论，在每次迭代中该变量都包含循环计数。之后我们显示发送评论的用户名，日期，和评论的内容。</p>
<p>最后，当表单提交成功后,你需要渲染表单或者显示一条成功的信息来代替之前的内容。在之前的代码后面添加如下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% if new_comment %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Your comment has been added.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
{% else %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Add a new comment<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"."</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    {{ comment_form.as_p }}
    {% csrf_token %}
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Add comment"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
{% endif %}</code></pre></div>
<p>这段代码非常简洁明了：如果<em>new_comment</em>对象存在，我们会展示一条成功信息因为成功创建了一条新评论。否则，我们用段落<code>&lt;p&gt;</code>元素渲染表单中每一个字段，并且包含<em>POST</em>请求需要的<em>CSRF</em>令牌。在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 然后点击任意一篇帖子的标题查看它的详情页面。你会看到如下页面展示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-6.png" alt="django-2-6"></p>
<p>使用该表单添加数条评论。这些评论会在你的帖子下面根据时间排序来展示，类似下图：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-7.png" alt="django-2-7"></p>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/admin/blog/comment/" class="uri">http://127.0.0.1:8000/admin/blog/comment/</a> 。你会在管理页面中看到你创建的评论列表。点击其中一个进行编辑，取消选择<strong>Active</strong>复选框，然后点击<strong>Save</strong>按钮。你会再次被重定向到评论列表页面，刚才编辑的评论<strong>Save</strong>列将会显示成一个没有激活的图标。类似下图的第一个评论：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-8.png" alt="django-2-8"></p>
<h2 id="增加标签tagging功能">增加标签（tagging）功能</h2>
<p>在实现了我们的评论系统之后，我们准备创创建一个方法来给我们的帖子添加标签。我们将通过在我们的项目中集成第三方的Django标签应用来完成这个功能。<em>django-taggit</em>是一个可复用的应用，它会提供给你一个<em>Tag</em>模型（model）和一个管理器（manager）来方便的给任何模型（model）添加标签。你可以在 <a href="https://github.com/alex/django-taggit" class="uri">https://github.com/alex/django-taggit</a> 看到它的源码。</p>
<p>首先，你需要通过pip安装django-taggit，运行以下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-taggit==<span class="hljs-number">0.17</span>.<span class="hljs-number">1</span>**（译者注：根据@孤独狂饮 验证，直接 `pip <span class="hljs-keyword">install</span> django-taggit` 安装最新版即可，原作者提供的版本过旧会有问题，感谢@孤独狂饮）**</code></pre>
<p>之后打开<em>mysite</em>项目下的<em>settings.py</em>文件，在<em>INSTALLED_APPS</em>设置中设置如下：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'blog'</span></span>,
    <span class="co"><span class="hljs-string">'taggit'</span></span>,
)</code></pre></div>
<p>打开你的blog应用下的<em>model.py</em>文件，给<em>Post</em>模型（model）添加<em>django-taggit</em>提供的<em>TaggableManager</em>管理器（manager），使用如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> taggit.managers <span class="im"><span class="hljs-keyword">import</span></span> TaggableManager
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Post</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="co"><span class="hljs-comment"># ...</span></span>
    tags <span class="op">=</span> TaggableManager()</code></pre></div>
<p>这个<em>tags</em>管理器（manager）允许你给<em>Post</em>对象添加，获取以及移除标签。</p>
<p>运行以下命令为你的模型（model）改变创建一个数据库迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span> <span class="hljs-selector-tag">blog</span></code></pre>
<p>你会看到如下输出：</p>
<pre class="shell"><code class="hljs delphi">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'blog'</span>:
  <span class="hljs-number">0003</span>_post_tags.py:
    - Add field tags <span class="hljs-keyword">to</span> post</code></pre>
<p>现在，运行以下代码在数据库中生成<em>django-taggit</em>模型（model）对应的表以及同步你的模型（model）的改变：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会看到以下输出,：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">taggit</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">taggit</span><span class="hljs-selector-class">.0002_auto_20150616_2121</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">blog</span><span class="hljs-selector-class">.0003_post_tags</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>你的数据库现在已经可以使用<em>django-taggit</em>模型（model）。打开终端运行命令<code>python manage.py shell</code>来学习如何使用<em>tags</em>管理器（manager）。首先，我们取回我们的其中一篇帖子（该帖子的ID为1）：</p>
<pre class="shell"><code class="hljs python"><span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> blog.models <span class="hljs-keyword">import</span> Post
<span class="hljs-meta">&gt;&gt;&gt; </span>post = Post.objects.get(id=<span class="hljs-number">1</span>)</code></pre>
<p>之后为它添加一些标签并且取回它的标签来检查标签是否添加成功：</p>
<pre class="shell"><code class="hljs fsharp">&gt;&gt;&gt; post.tags.add('music', 'jazz', 'django')
&gt;&gt;&gt; post.tags.all()
<span class="hljs-meta">[&lt;Tag: jazz&gt;, &lt;Tag: django&gt;, &lt;Tag: music&gt;]</span></code></pre>
<p>最后，移除一个标签并且再次检查标签列表：</p>
<pre class="shell"><code class="hljs fsharp">&gt;&gt;&gt; post.tags.remove('django')
&gt;&gt;&gt; post.tags.all()
<span class="hljs-meta">[&lt;Tag: jazz&gt;, &lt;Tag: music&gt;]</span></code></pre>
<p>非常简单，对吧？运行命令<code>python manage.py runserver</code>启动开发服务器，在浏览器中打开 <a href="http://127.0.0.1:8000/admin/taggit/tag/" class="uri">http://127.0.0.1:8000/admin/taggit/tag/</a> 。你会看到管理页面包含了<em>taggit</em>应用的<em>Tag</em>对象列表：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-9.png" alt="django-2-9"></p>
<p>转到 <a href="http://127.0.0.1:8000/admin/blog/post/" class="uri">http://127.0.0.1:8000/admin/blog/post/</a> 并点击一篇帖子进行编辑。你会看到帖子中包含了一个新的<strong>Tags</strong>字段如下所示，你可以非常容易的编辑它：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-10.png" alt="django-2-10"></p>
<p>现在，我们准备编辑我们的blog帖子来显示这些标签。打开<em>blog/post/list.html</em> 模板（template）在帖子标题下方添加如下HTML代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"tags"</span>&gt;</span>Tags: </span><span class="hljs-template-variable">{{ post.tags.all|<span class="hljs-name">join</span>:<span class="hljs-string">", "</span> }}</span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre>
<p><em>join</em>模板（template）过滤器（filter）的功能类似python字符串的join()方法，将给定的字符串连接起来。在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 。 你会看到每一个帖子的标题下面的标签列表：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-11.png" alt="django-2-11"></p>
<p>现在，让我们来编辑我们的<em>post_list</em>视图（view）让用户可以列出打上了特定标签的所有帖子。打开blog应用下的<em>views.py</em>文件，从<em>django-taggit</em>中导入<em>Tag</em>模型（model），然后修改<em>post_list</em>视图（view）让它可以通过标签选择性的过滤，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> taggit.models <span class="im"><span class="hljs-keyword">import</span></span> Tag

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_list</span><span class="hljs-params">(request, tag_slug</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span> 
    object_list <span class="op">=</span> Post.published.<span class="bu">all</span>() 
    tag <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    
    <span class="cf"><span class="hljs-keyword">if</span></span> tag_slug:
        tag <span class="op">=</span> get_object_or_404(Tag, slug<span class="op">=</span>tag_slug) 
        object_list <span class="op">=</span>   object_list.<span class="bu">filter</span>(tags__in<span class="op">=</span>[tag]) 
        <span class="co"><span class="hljs-comment"># ...</span></span></code></pre></div>
<p>这个视图（view）做了以下工作：</p>
<ul>
<li>1.视图（view）带有一个可选的<em>tag_slug</em>参数，默认是一个<em>None</em>值。这个参数会带进URL中。</li>
<li>2.视图（view）的内部，我们构建了初始的查询集（QuerySet），取回所有发布状态的帖子，假如给予一个标签 slug，我们通过<code>get_object_or_404()</code>用给定的slug来获取标签对象。</li>
<li>3.之后我们过滤所有帖子只留下包含给定标签的帖子。因为有一个多对多（many-to-many）的关系，我们必须通过给定的标签列表来过滤，在我们的例子中标签列表只包含一个元素。</li>
</ul>
<p>要记住查询集（QuerySets)是惰性的。这个查询集（QuerySets)只有当我们在模板（template）中循环渲染帖子列表时才会被执行。</p>
<p>最后，修改视图（view）最底部的<em>render()</em>函数来，传递<em>tag</em>变量给模板（template）。这个视图（view）完成后如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post_list</span><span class="hljs-params">(request, tag_slug</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
   object_list <span class="op">=</span> Post.published.<span class="bu">all</span>()
   tag <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
   
   <span class="cf"><span class="hljs-keyword">if</span></span> tag_slug:
       tag <span class="op">=</span> get_object_or_404(Tag, slug<span class="op">=</span>tag_slug)
       object_list <span class="op">=</span> object_list.<span class="bu">filter</span>(tags__in<span class="op">=</span>[tag])
       
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
   <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'blog/post/list.html'</span></span>, {<span class="st"><span class="hljs-string">'page'</span></span>: page,
                                                  <span class="co"><span class="hljs-string">'posts'</span></span>: posts,
                                                  <span class="co"><span class="hljs-string">'tag'</span></span>: tag})</code></pre></div>
<p>打开blog应用下的<em>url.py</em>文件，注释基于类的<em>PostListView</em> URL模式，然后取消<em>post_list</em>视图（view）的注释，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, views.post_list, name<span class="op">=</span><span class="st"><span class="hljs-string">'post_list'</span></span>),
<span class="co"><span class="hljs-comment"># url(r'^$', views.PostListView.as_view(), name='post_list'),</span></span></code></pre></div>
<p>添加下面额外的URL pattern到通过标签过滤过的帖子列表中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^tag/(?P&lt;tag_slug&gt;[-\w]+)/$'</span></span>,views.post_list,
    name<span class="op">=</span><span class="st"><span class="hljs-string">'post_list_by_tag'</span></span>),</code></pre></div>
<p>如你所见，两个模式都指向了相同的视图（view），但是我们可以给它们不同的命名。第一个模式会调用<em>post_list</em>视图（view）并且不带上任何可选参数。然而第二个模式会调用这个视图（view）带上<em>tag_slug</em>参数。</p>
<p>因为我们要使用<em>post_list</em>视图（view）,编辑<em>blog/post/list.html</em> 模板（template），使用<em>posts</em>对象修改pagination，如下所示：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag">{% <span class="hljs-name"><span class="hljs-name">include</span></span> "pagination.html" with page=posts %}</span><span class="xml"></span></code></pre>
<p>在<code>{% for %}</code>循环上方添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">{% if tag %}
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Posts tagged with "{{ tag.name }}"<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
{% endif %}</code></pre></div>
<p>如果用户正在访问blog，他会看到所有帖子列表。如果他指定一个标签来过滤所有的帖子，他就会看到以上的信息。现在，修改标签的显示方式，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"tags"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
  Tags:
  {% for tag in post.tags.all %}
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"{% url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">blog:post_list_by_tag"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">tag.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">%}"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      {{ tag.name }}
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    {% if not forloop.last %}, {% endif %}
  {% endfor %}
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>现在，我们循环一个帖子的所有标签，通过某一标签来显示一个自定义的链接URL。我们通过<code>{% url "blog:post_list_by_tag" tag.slug %}</code>，用URL的名称以及标签 slug作为参数来构建URL。我们使用逗号分隔这些标签。</p>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/blog/" class="uri">http://127.0.0.1:8000/blog/</a> 然后点击任意的标签链接，你会看到通过该标签过滤过的帖子列表，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-12.png" alt="django-2-12"></p>
<h2 id="检索类似的帖子">检索类似的帖子</h2>
<p>如今，我们已经可以给我们的blog帖子加上标签，我们可以通过它们做更多有意思的事情。通过使用标签，我们能够很好的分类我们的blog帖子。拥有类似主题的帖子一般会有几个共同的标签。我们准备创建一个功能：通过帖子共享的标签数量来显示类似的帖子。这样的话，当一个用户阅读一个帖子，我们可以建议他们去读其他有关联的帖子。</p>
<p>为了通过一个特定的帖子检索到类似的帖子，我们需要做到以下几点：</p>
<ul>
<li>返回当前帖子的所有标签。</li>
<li>返回所有带有这些标签的帖子。</li>
<li>在返回的帖子列表中排除当前的帖子，避免推荐相同的帖子。</li>
<li>通过和当前帖子共享的标签数量来排序所有的返回结果。</li>
<li>假设有两个或多个帖子拥有相同数量的标签，推荐最近的帖子。</li>
<li>限制我们想要推荐的帖子数量。</li>
</ul>
<p>这些步骤可以转换成一个复杂的查询集（QuerySet），该查询集（QuerySet）我们需要包含在我们的<em>post_detail</em>视图（view）中。打开blog应用中的<em>view.py</em>文件，在顶部添加如下导入：</p>
<pre><code class="hljs swift">from django.db.models <span class="hljs-keyword">import</span> Count</code></pre>
<p>这是Django ORM的<em>Count</em>聚合函数。这个函数允许我们处理聚合计算。然后在<em>post_detail</em>视图（view）的<em>render()</em>函数之前添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># List of similar posts</span></span>
post_tags_ids <span class="op">=</span> post.tags.values_list(<span class="st"><span class="hljs-string">'id'</span></span>, flat<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
similar_posts <span class="op">=</span> Post.published.<span class="bu">filter</span>(tags__in<span class="op">=</span>post_tags_ids)<span class="op">\</span>
                               .exclude(<span class="bu">id</span><span class="op">=</span>post.<span class="bu">id</span>)
similar_posts <span class="op">=</span> similar_posts.annotate(same_tags<span class="op">=</span>Count(<span class="st"><span class="hljs-string">'tags'</span></span>))<span class="op">\</span>
                            .order_by(<span class="st"><span class="hljs-string">'-same_tags'</span></span>,<span class="st"><span class="hljs-string">'-publish'</span></span>)[:<span class="dv"><span class="hljs-number">4</span></span>]</code></pre></div>
<p>以上代码的解释如下：</p>
<ul>
<li>1.我们取回了一个包含当前帖子所有标签的ID的Python列表。<em>values_list()</em> 查询集（QuerySet）返回包含给定的字段值的元祖。我们传给元祖<code>flat=True</code>来获取一个简单的列表类似<code>[1,2,3,...]</code>。</li>
<li>2.我们获取所有包含这些标签的帖子排除了当前的帖子。</li>
<li>3.我们使用<em>Count</em>聚合函数来生成一个计算字段<em>same_tags</em>，该字段包含与查询到的所有 标签共享的标签数量。</li>
<li>4.我们通过共享的标签数量来排序（降序）结果并且通过<em>publish</em>字段来挑选拥有相同共享标签数量的帖子中的最近的一篇帖子。我们对返回的结果进行切片只保留最前面的4篇帖子。</li>
</ul>
<p>在<em>render()</em>函数中给上下文字典增加<em>similar_posts</em>对象，如下所示：</p>
<pre><code class="hljs lisp">return render(<span class="hljs-name">request</span>,
                'blog/post/detail.html',
                {'post': post,
                'comments': comments,
                'comment_form': comment_form,
                'similar_posts': similar_posts})</code></pre>
<p>现在，编辑<em>blog/post/detail.html</em>模板（template）在帖子评论列表前添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Similar posts<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
  {% for post in similar_posts %}
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"{{ post.get_absolute_url }}"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>{{ post.title }}<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
  {% empty %}
    There are no similar posts yet.
  {% endfor %}</code></pre></div>
<p>推荐你在你的帖子详情模板中添加标签列表，就像我们在帖子列表模板所做的一样。现在，你的帖子详情页面看上去如下所示：<br>
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-2-13.png" alt="django-2-13"></p>
<p>你已经成功的为你的用户推荐了类似的帖子。<em>django-taggit</em>还内置了一个<code>similar_objects()</code> 管理器（manager）使你可以通过共享的标签返回所有对象。你可以通过访问 <a href="http://django-taggit.readthedocs.org/en/latest/api.html" class="uri">http://django-taggit.readthedocs.org/en/latest/api.html</a> 看到所有django-taggit管理器。</p>
<h2 id="总结">总结</h2>
<p>在本章中，你学习了如何使用Django的表单和模型（model）表单。你创建了一个通过email分享你的站点内容的系统，还为你的博客创建了一个评论系统。通过集成一个可复用的应用，你为你的帖子增加了打标签的功能。同时，你还构建了一个复杂的查询集（QuerySets)用来返回类似的对象。</p>
<p>在下一章中，你会学习到如何创建自定义的模板（temaplate）标签（tags）和过滤器（filters）。你还会为你的博客应用构建一个自定义的站点地图，集成一个高级的搜索引擎。</p>
<p>本教程出自由Antonio Melé编写的《Django By Example》, 后由<a href="http://www.cnblogs.com/levelksk/">@夜夜月月</a>等童鞋辛劳翻译。请各位读者给他们的主页添点人气</p>