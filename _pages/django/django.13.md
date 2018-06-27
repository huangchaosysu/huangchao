---
layout: single
permalink: /django/example4/
title: "Django By Example 第四章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第四章">第四章</h1>
<h2 id="创建一个社交网站">创建一个社交网站</h2>
<p>在上一章中，你学习了如何创建站点地图（sitemaps）和feeds，你还为你的blog应用创建了一个搜索引擎。在本章中，你将开发一个社交应用。你会为用户创建一些功能，例如：登录，登出，编辑，以及重置他们的密码。你会学习如何为你的用户创建一个定制的profile，你还会为你的站点添加社交认证。</p>
<p>本章将会覆盖一下几点：</p>
<ul>
<li>使用认证（authentication）框架</li>
<li>创建用户注册视图（views）</li>
<li>通过一个定制的profile模型（model）扩展<em>User</em>模型（model）</li>
<li>使用<em>python-social-auth</em>添加社交认证</li>
</ul>
<p>让我们开始创建我们的新项目吧。</p>
<h3 id="创建一个社交网站项目">创建一个社交网站项目</h3>
<p>我们要创建一个社交应用允许用户分享他们在网上找到的图片。我们需要为这个项目构建以下元素：</p>
<ul>
<li>一个用来给用户注册，登录，编辑他们的profile，以及改变或重置密码的认证（authentication）系统</li>
<li>一个允许用户用来关注其他人的关注系统（这里原文是follow，‘跟随’，感觉用‘关注’更加适合点）</li>
<li>为用户从其他任何网站分享过来的图片进行展示和打上书签</li>
<li>每个用户都有一个活动流允许用户看到他们关注的人上传的内容</li>
</ul>
<p>本章主要讲述第一点。</p>
<h3 id="开始你的社交网站项目">开始你的社交网站项目</h3>
<p>打开终端使用如下命令行为你的项目创建一个虚拟环境并且激活它：<br>
​<br>
mkdir evn<br>
virtualenv evn/bookmarks<br>
source env/bookmarks/bin/activate</p>
<p>shell提示将会展示你激活的虚拟环境，如下所示：</p>
<pre><code class="hljs groovy">(bookmarks)<span class="hljs-string">laptop:</span>~ zenx$</code></pre>
<p>通过以下命令在你的虚拟环境中安装Django：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Django==<span class="hljs-number">1.8</span>.<span class="hljs-number">6</span></code></pre>
<p>运行以下命令来创建一个新项目：</p>
<pre><code class="hljs">django-admin statproject bookmarks</code></pre>
<p>在创建好一个初始的项目结构以后，使用以下命令进入你的项目目录并且创建一个新的应用命名为<em>account</em>:</p>
<pre><code class="hljs dos"><span class="hljs-built_in">cd</span> bookmarks/
django-admin startapp account</code></pre>
<p>请记住在你的项目中激活一个新应用需要在<em>settings.py</em>文件中的<em>INSTALLED_APPS</em>设置中添加它。将新应用的名字添加在<em>INSTALLED_APPS</em>列中的所有已安装应用的最前面，如下所示：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">INSTALLED_APPS</span> = (
    <span class="hljs-string">'account'</span>,
    <span class="hljs-comment"># ...</span>
)</code></pre>
<p>运行下一条命令为<em>INSTALLED_APPS</em>中默认包含的应用模型（models）同步到数据库中：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>我们将要使用认证（authentication）框架来构建一个认证系统到我们的项目中。</p>
<h3 id="使用django认证authentication框架">使用Django认证（authentication）框架</h3>
<p>Django拥有一个内置的认证（authentication）框架用来操作用户认证（authentication），会话（sessions），权限（permissions）以及用户组。这个认证（authentication）系统包含了一些普通用户的操作视图（views），例如：登录，登出，修改密码以及重置密码。</p>
<p>这个认证（authentication）框架位于<em>django.contrib.auth</em>，被其他Django的<em>contrib</em>包调用。请记住你使用过这个认证（authentication）框架在<em>第一章 创建一个Blog应用</em>中用来为你的blog应用创建了一个超级用户来使用管理站点。</p>
<p>当你使用<em>startproject</em>命令创建一个新的Django项目，认证（authentication）框架已经在你的项目设置中默认包含。它是由<em>django.contrib.auth</em>应用和你的项目设置中的<em>MIDDLEWARE_CLASSES</em>中的两个中间件类组成，如下：</p>
<ul>
<li>AuthenticationMiddlwware:使用会话（sessions）将用户和请求（requests）进行关联</li>
<li>SessionMiddleware:通过请求（requests）操作当前会话（sessions）</li>
</ul>
<p>中间件就是一个在请求和响应阶段带有全局执行方法的类。你会在本书中的很多场景中使用到中间件。你将会学习如何创建一个定制的中间件在<em>第十三章 Going Live（译者注：啥时候能翻译到啊）</em>。</p>
<p>这个认证（authentication）系统还包含了以下模型（models）：</p>
<ul>
<li>User：一个用户模型（model）包含基础字段；这个模型（model）的主要字段有：username,password,email,first_name,last_name,is_active。</li>
<li>Group：一个组模型（model）用来分类用户</li>
<li>Permission：执行特定操作的标识</li>
</ul>
<p>这个框架还包含默认的认证（authentication）视图（views）和表单（forms），我们之后会用到。</p>
<h3 id="创建一个log-in视图view">创建一个log-in视图（view）</h3>
<p>我们将要开始使用Django认证（authentication）框架来允许用户登录我们的网站。我们的视图（view）需要执行以下操作来登录用户：</p>
<ul>
<li>1、通过提交的表单（form）获取username和password</li>
<li>2、通过存储在数据库中的数据对用户进行认证</li>
<li>3、检查用户是否可用</li>
<li>4、登录用户到网站中并且开始一个认证（authentication）会话（session）</li>
</ul>
<p>首先，我们要创建一个登录表单（form）。在你的account应用目录下创建一个新的<em>forms.py</em>文件，添加如下代码：</p>
<pre><code class="hljs haskell"><span class="hljs-title">from</span> django <span class="hljs-keyword">import</span> forms    
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-type">LoginForm</span>(<span class="hljs-title">forms</span>.<span class="hljs-type">Form</span>):        
    username = forms.<span class="hljs-type">CharField</span>()        
    password = forms.<span class="hljs-type">CharField</span>(<span class="hljs-title">widget</span>=<span class="hljs-title">forms</span>.<span class="hljs-type">PasswordInput</span>)</span></code></pre>
<p>这个表单（form）被用来通过数据库认证用户。请注意，我们使用<em>PsswordInput</em>控件来渲染HTML<code>input</code>元素，包含<code>type="password</code>属性。编辑你的<em>account</em>应用中的<em>views.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.http <span class="hljs-keyword">import</span> HttpResponse    
<span class="hljs-keyword">from</span> django.shortcuts <span class="hljs-keyword">import</span> render    
<span class="hljs-keyword">from</span> django.contrib.auth <span class="hljs-keyword">import</span> authenticate, login    
<span class="hljs-keyword">from</span> .forms <span class="hljs-keyword">import</span> LoginForm    
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">user_login</span><span class="hljs-params">(request)</span>:</span>        
    <span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:            
        form = LoginForm(request.POST)            
        <span class="hljs-keyword">if</span> form.is_valid():                
            cd = form.cleaned_data                
            user = authenticate(username=cd[<span class="hljs-string">'username'</span>],
                                password=cd[<span class="hljs-string">'password'</span>])                
            <span class="hljs-keyword">if</span> user <span class="hljs-keyword">is</span> <span class="hljs-keyword">not</span> <span class="hljs-keyword">None</span>:                    
                <span class="hljs-keyword">if</span> user.is_active:                        
                    login(request, user)                        
                    <span class="hljs-keyword">return</span> HttpResponse(<span class="hljs-string">'Authenticated successfully'</span>)
                <span class="hljs-keyword">else</span>:                        
                    <span class="hljs-keyword">return</span> HttpResponse(<span class="hljs-string">'Disabled account'</span>)
        <span class="hljs-keyword">else</span>:            
            <span class="hljs-keyword">return</span> HttpResponse(<span class="hljs-string">'Invalid login'</span>)        
    <span class="hljs-keyword">else</span>:            
        form = LoginForm()        
    <span class="hljs-keyword">return</span> render(request, <span class="hljs-string">'account/login.html'</span>, {<span class="hljs-string">'form'</span>: form})</code></pre>
<p>以上就是我们在视图（view）中所作的基本登录操作：当<em>user_login</em>被一个GET请求（request）调用，我们实例化一个新的登录表单（form）通过<code>form = LoginForm()</code>在模板（template）中展示它。当用户通过POST方法提交表单（form），我们执行以下操作：</p>
<ul>
<li>1、使用提交的数据实例化表单（form）通过使用<code>form = LoginForm(request.POST)</code></li>
<li>2、检查这个表单是否有效。如果无效，我们在模板（template）中展示表单错误信息（举个例如，比如用户没有填写其中一个字段就进行提交）</li>
<li>3、如果提交的数据是有效的，我们通过数据库对这个用户进行认证（authentication）通过使用<code>authenticate()</code>方法。这个方法带入一个<em>username</em>和一个<em>password</em>并且返回一个用户对象如果这个用户成功的进行了认证，或者是<em>None</em>。如果用户没有被认证通过，我们返回一个<em>HttpResponse</em>展示一条消息。</li>
<li>4、如果这个用户认证（authentication）成功，我们使用<code>is_active</code>属性来检查用户是否可用。这是一个Django的<em>User</em>模型（model)属性。如果这个用户不可用，我们返回一个<em>HttpResponse</em>展示信息。</li>
<li>5、如果用户可用，我们登录这个用户到网站中。我们通过调用<code>login()</code>方法集合用户到会话（session）中然后返回一条成功消息。</li>
</ul>
<blockquote>
<p>请注意<em>authentication</em>和<em>login</em>中的不同点：<code>authenticate()</code>检查用户认证信息然后返回一个用户对象如果用户是正确的；<code>login()</code>集合用户到当前的会话（session）中。</p>
</blockquote>
<p>现在，你需要为这个视图（view）创建一个URL模式。在你的<em>account</em>应用目录下创建一个新的<em>urls.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> url    
<span class="hljs-keyword">from</span> . <span class="hljs-keyword">import</span> views    
urlpatterns = [        
    <span class="hljs-comment"># post views        </span>
    url(<span class="hljs-string">r'^login/$'</span>, views.user_login, name=<span class="hljs-string">'login'</span>),    
]</code></pre>
<p>编辑位于你的<em>bookmarks</em>项目目录下的<em>urls.py</em>文件，将<em>account</em>应用下的URL模式包含进去：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> include, url    
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin    
urlpatterns = [        
    url(<span class="hljs-string">r'^admin/'</span>, include(admin.site.urls)),
    url(<span class="hljs-string">r'^account/'</span>,include(<span class="hljs-string">'account.urls'</span>)),    
]</code></pre>
<p>这个登录视图（view）现在已经可以通过URL进行访问。现在是时候为这个视图（view）创建一个模板。因为之前你没有这个项目的任何模板，你可以开始创建一个主模板（template）可以被登录模板（template）继承使用。创建以下文件和结构在<em>account</em>应用目录中：</p>
<pre><code class="hljs dts">templates<span class="hljs-meta-keyword">/account/</span>login.htmlbase.html</code></pre>
<p>编辑<em>base.html</em>文件添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> staticfiles <span>%</span><span>}</span></span><span class="xml">    
<span class="hljs-meta">&lt;!DOCTYPE html&gt;</span>    
<span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span>    
    <span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span>      
        <span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span>      
        <span class="hljs-tag">&lt;<span class="hljs-name">link</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">static</span></span> "css/base.css" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">rel</span>=<span class="hljs-string">"stylesheet"</span>&gt;</span>    
    <span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span>    
    <span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span>      
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"header"</span>&gt;</span>        
            <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"logo"</span>&gt;</span>Bookmarks<span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>      
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>      
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"content"</span>&gt;</span>        
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">        
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">      
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>    
    <span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span>    
<span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre>
<p>以上将是这个网站的基础模板（template）。就像我们在上一章项目中做过的一样，我们在这个住模板（template）中包含了CSS样式。你可以在本章的示例代码中找到这些静态文件。复制示例代码中的<em>account</em>应用下的<em>static/</em>目录到你的项目中的相同位置，这样你就可以使用这些静态文件了。</p>
<p>基础模板（template）定义了一个<em>title</em>和一个<em>content</em>区块可以被继承的子模板（template）填充内容。</p>
<p>让我们为我们的登录表单（form）创建模板（template）。打开<em>account/login.html</em>模板（template）添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Log-in</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Log-in<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Please, use the following form to log-in:<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
        </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Log-in"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板（template）包含了视图（view）中实例化的表单（form）。因为我们的表单（form）将会通过POST方式进行提交，所以我们包含了<code><span>{</span><span>%</span> csrf_token <span>%</span><span>}</span></code>模板（template）标签(tag)用来通过CSRF保护。你已经学习过CSRF保护在<em>第二章 使用高级特性扩展你的博客应用</em>。</p>
<p>目前还没有用户在你的数据库中。你首先需要创建一个超级用户用来登录管理站点来管理其他的用户。打开命令行执行<code>python manage.py createsuperuser</code>。填写username,e-mail以及password。之后通过命令<code>python manage.py runserver</code>运行开发环境，然后在你的浏览器中打开 <a href="http://127.0.0.1:8000/admin/" class="uri">http://127.0.0.1:8000/admin/</a> 。使用你刚才创建的username和passowrd来进入管理站点。你会看到Django管理站点包含Django认证（authentication）框架中的<em>User</em>和<em>Group</em>模型（models）。如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-1.png" alt="django-4-1"></p>
<p>使用管理站点创建一个新的用户然后打开 <a href="http://127.0.0.1:8000/account/login/" class="uri">http://127.0.0.1:8000/account/login/</a> 。你会看到被渲染过的模板（template），包含一个登录表单（form）：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-2.png" alt="django-4-2"></p>
<p>现在，只填写一个字段保持另外一个字段为空进行表单（from）提交。在这例子中，你会看到这个表单（form）是无效的并且显示了一个错误信息：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-3.png" alt="django-4-3"></p>
<p>如果你输入了一个不存在的用户或者一个错误的密码，你会得到一个<strong>Invalid login</strong>信息。</p>
<p>如果你输入了有效的认证信息，你会得到一个<strong>Authenticated successfully</strong>消息：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-4.png" alt="django-4-4"></p>
<h3 id="使用django认证authentication视图views">使用Django认证（authentication）视图（views）</h3>
<p>Django包含了一些表单（forms）和视图（views）在认证（authentication）框架中让你可以立刻(straightaway)使用。你之前创建的登录视图（view）是一个非常的练习用来理解Django中的用户认证（authentication）过程。无论如何，你可以使用默认的Django认证（authentication）视图（views）在大部分的例子中。</p>
<p>Django提供以下视图（views）来处理认证（authentication）：</p>
<ul>
<li>login：操作表单（form）中的登录然后登录一个用户</li>
<li>logout：登出一个用户</li>
<li>logout_then_login：登出一个用户然后重定向这个用户到登录页面</li>
</ul>
<p>Django提供以下视图（views）来操作密码修改：</p>
<ul>
<li>password_change:操作一个表单（form）来修改用户密码</li>
<li>password_change_done:当用户成功修改他的密码后提供一个成功提示页面</li>
</ul>
<p>Django还包含了以下视图（views）允许用户重置他们的密码：</p>
<ul>
<li>password_reset:允许用户重置他的密码。它会生成一条带有一个token的一次性使用链接然后发送到用户的邮箱中。</li>
<li>password_reset_done:告知用户已经发送了一封可以用来重置密码的邮件到他的邮箱中。</li>
<li>password_reset_complete:当用户重置完成他的密码后提供一个成功提示页面。</li>
</ul>
<p>以上的视图（views）可以帮你节省很多时间当你创建一个带有用户账号的网站。这些视图（views）使用的默认值你可以进行覆盖，例如需要渲染的模板位置或者视图（view）需要使用到的表单（form）。</p>
<p>你可以通过访问 <a href="https://docs/" class="uri">https://docs</a>.<br>
djangoproject.com/en/1.8/topics/auth/default/#module-django.contrib.auth.views 获取更多内置的认证（authentication）视图（views）信息。</p>
<h3 id="登录和登出视图views">登录和登出视图（views）</h3>
<p>编辑你的<em>account</em>应用下的<em>urls.py</em>文件，如下所示：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> url
<span class="hljs-keyword">from</span> . <span class="hljs-keyword">import</span> views
urlpatterns = [
    <span class="hljs-comment"># previous login view</span>
    <span class="hljs-comment"># url(r'^login/$', views.user_login, name='login'),</span>
    <span class="hljs-comment"># login / logout urls</span>
    url(<span class="hljs-string">r'^login/$'</span>,
        <span class="hljs-string">'django.contrib.auth.views.login'</span>,
        name=<span class="hljs-string">'login'</span>),
    url(<span class="hljs-string">r'^logout/$'</span>,
        <span class="hljs-string">'django.contrib.auth.views.logout'</span>,
        name=<span class="hljs-string">'logout'</span>),
    url(<span class="hljs-string">r'^logout-then-login/$'</span>,
        <span class="hljs-string">'django.contrib.auth.views.logout_then_login'</span>,
        name=<span class="hljs-string">'logout_then_login'</span>),
]</code></pre>
<p>我们将之前创建的<em>user_login</em>视图（view）URL模式进行注释，然后使用Django认证（authentication）框架提供的<em>login</em>视图（view）。</p>
<p>在你的account应用中的template目录下创建一个新的目录命名为<em>registration</em>。这个路径是Django认证（authentication）视图（view）期望你的认证（authentication）模块（template）默认的存放路径。在这个新目录中创建一个新的文件，命名为<em>login.html</em>，然后添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Log-in</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Log-in<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> form.errors <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>
      Your username and password didn't match.
      Please try again.
    <span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Please, use the following form to log-in:<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"login-form"</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> 'login' <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
      </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
      </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
      <span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"hidden"</span> <span class="hljs-attr">name</span>=<span class="hljs-string">"next"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> next <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> /&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Log-in"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
 </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个登录模板（template）和我们之前创建的基本类似。Django默认使用位于<em>django.contrib.auth.forms</em>中的<em>AuthenticationForm</em>。这个表单（form）会尝试对用户进行认证如果登录不成功就会抛出一个验证错误。在这个例子中，我们可以在模板（template）中使用<code><span>{</span><span>%</span> if form.errors <span>%</span><span>}</span></code>来找到错误如果用户提供了错误的认证信息。请注意，我们添加了一个隐藏的HTML<code>&lt;input&gt;</code>元素来提交叫做<em>next</em>的变量值。这个变量是登录视图（view）的首个设置当你在请求（request）中传递一个<em>next</em>参数（举个例子：<a href="http://127.0.0.1:8000/account/login/?next=/account/" class="uri">http://127.0.0.1:8000/account/login/?next=/account/</a>）。</p>
<p><em>next</em>参数必须是一个URL。当这个参数被给予的时候，Django登录视图（view）将会在用户登录完成后重定向到给予的URL。</p>
<p>现在，在<em>registrtion</em>模板（template）目录下创建一个<em>logged_out.html</em>模板（template）添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Logged out</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Logged out<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>You have been successfully logged out. You can <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "login" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>log-in again<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板（template）Django会在用户登出的时候进行展示。</p>
<p>在添加了URL模式以及登录和登出视图（view）的模板之后，你的网站已经准备好让用户使用Django认证（authentication）视图进行登录。</p>
<blockquote>
<p>请注意，在我们的地址配置中包含的<em>logtou_then_login</em>视图（view）不需要任何模板（template），因为它执行了一个重定向到登录视图（view）。</p>
</blockquote>
<p>现在，我们要创建一个新的视图（view）来显示一个dashboard给用户当他或她登录他们的账号。打开你的<em>account</em>应用中的<em>views.py</em>文件，添加以下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib.auth.decorators <span class="hljs-keyword">import</span> login_required
<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dashboard</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-keyword">return</span> render(request,
                 <span class="hljs-string">'account/dashboard.html'</span>,
                 {<span class="hljs-string">'section'</span>: <span class="hljs-string">'dashboard'</span>})</code></pre>
<p>我们使用认证（authentication）框架的<em>login_required</em>装饰器（decorator）装饰我们的视图（view）。<em>login_required</em>装饰器（decorator）会检查当前用户是否通过认证,如果用户通过认证，它会执行装饰的视图（view），如果用户没有通过认证，它会把用户重定向到带有一个名为<em>next</em>的GET参数的登录URL，该GET参数保存的变量为用户当前尝试访问的页面URL。通过这些动作，登录视图（view）会将登录成功的用户重定向到用户登录之前尝试访问过的URL。请记住我们在登录模板（template）中的登录表单（form）中添加的隐藏<code>&lt;input&gt;</code>就是为了这个目的。</p>
<p>我们还定义了一个<em>section</em>变量。我们会使用该变量来跟踪用户在站点中正在查看的页面。多个视图（views）可能会对应相同的section。这是一个简单的方法用来定义每个视图（view）对应的section。</p>
<p>现在，你需要创建一个给dashboard视图（view）使用的模板（template）。在<em>templates/account/</em>目录下创建一个新文件命名为<em>dashboard.html</em>。添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Dashboard</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Dashboard<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Welcome to your dashboard.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>之后，为这个视图（view）在<em>account</em>应用中的<em>urls.py</em>文件中添加如下URL模式：</p>
<pre><code class="hljs python">urlpatterns = [
    <span class="hljs-comment"># ...</span>
    url(<span class="hljs-string">r'^$'</span>, views.dashboard, name=<span class="hljs-string">'dashboard'</span>),
]</code></pre>
<p>编辑你的项目的<em>settings.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.core.urlresolvers <span class="hljs-keyword">import</span> reverse_lazy
LOGIN_REDIRECT_URL = reverse_lazy(<span class="hljs-string">'dashboard'</span>)
LOGIN_URL = reverse_lazy(<span class="hljs-string">'login'</span>)
LOGOUT_URL = reverse_lazy(<span class="hljs-string">'logout'</span>)</code></pre>
<p>这些设置的意义：</p>
<ul>
<li>LOGIN_REDIRECT_URL：告诉Django用户登录成功后如果<em>contrib.auth.views.login</em>视图（view）没有获取到<em>next</em>参数将会默认重定向到哪个URL。</li>
<li>LOGIN_URL：重定向用户登录的URL（例如：使用<em>login_required</em>装饰器（decorator））。</li>
<li>LOGOUT_URL：重定向用户登出的URL。</li>
</ul>
<p>我们使用<em>reverse_laze()</em>来动态构建URL通过它们的名字。<em>reverse_laze()</em>方法reverses URLs就像<em>reverse()</em>所做的一样，但是你可以使用它当你需要reverse URLs在你项目的URL配置读取之前。</p>
<p>让我们总结下目前为止我们都做了哪些工作：</p>
<ul>
<li>你在项目中添加了Django内置的认证（authentication）登录和登出视图（views）。</li>
<li>你为每一个视图（view)创建了定制的模板（templates），并且定义了一个简单的视图（view）让用户登录后进行重定向。</li>
<li>最后，你配置了你的Django设置使用默认的URLs。</li>
</ul>
<p>现在，我们要给我们的主模板（template）添加登录和登出链接将所有的一切都连接起来。</p>
<p>为了做到这点，我们必须确定无论用户是已登录状态还是没有登录的时候，都会显示适当的链接。通过认证（authentication）中间件当前的用户被集合在HTTP请求（request)对象中。你可以访问用户信息通过使用<code>request.user</code>。你会防线一个用户对象在请求（request)中，即使这个用户并没有认证通过。一个非认证的用户在请求（request）被设置成一个<em>AnonymousUser</em>的实例。一个最好的方法来检查当前的用户是否通过认证是通过调用<code>request.user.is_authenticated()</code>。</p>
<p>编辑你的<em>base.html</em>文件修改ID为<em>header</em>的<code>&lt;div&gt;</code>元素，如下所示：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"header"</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"logo"</span>&gt;</span>Bookmarks<span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user.is_authenticated <span>%</span><span>}</span></span><span class="xml">
     <span class="hljs-tag">&lt;<span class="hljs-name">ul</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"menu"</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">li</span> </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> section == "dashboard" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-attr">class</span>=<span class="hljs-string">"selected"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "dashboard" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>My dashboard<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">li</span> </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> section == "images" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-attr">class</span>=<span class="hljs-string">"selected"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"#"</span>&gt;</span>Images<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
      <span class="hljs-tag">&lt;<span class="hljs-name">li</span> </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> section == "people" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-attr">class</span>=<span class="hljs-string">"selected"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"#"</span>&gt;</span>People<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
       <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
      <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span>
     </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
     <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"user"</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user.is_authenticated <span>%</span><span>}</span></span><span class="xml">
         Hello </span><span class="hljs-template-variable"><span>{</span><span>{</span> request.user.first_name <span>}</span><span>}</span></span><span class="xml">,
         <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "logout" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Logout<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
         <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "login" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Log-in<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre>
<p>就像你所看到的，我们只给通过认证（authentication）的用户显示站点菜单。我们还检查当前的section来给对应的<code>&lt;li&gt;</code>组件添加一个<em>selected</em>的class属性来使当前section在菜单中进行高亮显示通过使用CSS。我们还显示用户的第一个名字和一个登出的链接如果用户已经通过认证（authentication），否则，就是一个登出链接。</p>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/account/login/" class="uri">http://127.0.0.1:8000/account/login/</a> 。你会看到登录页面。输入可用的用户名和密码然后点击<strong>Log-in</strong>按钮。你会看到如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-5.png" alt="django-4-5"></p>
<p>你能看到<em>My Dashboard</em> section 通过CSS的作用高亮显示因为它拥有一个<em>selected</em> class。因为当前用户已经通过了认证（authentication）所有用户的第一个名字在右上角进行了显示。点击<strong>Logout</strong>链接。你会看到如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-6.png" alt="django-4-6"></p>
<p>在这个页面中，你能看到用户已经登出，然后，你无法看到当前网站的任何菜单。在右上角现在显示的是<strong>Log-in</strong>链接。</p>
<p>如果你在你的登出页面中看到了Django管理站点的登出页面，检查项目<em>settings.py</em>中的<em>INSTALLED_APPS</em>,确保<em>django.contrib.admin</em>在<em>account</em>应用的后面。每个模板（template）被定位在同样的相对路径时，Django模板（template）读取器将会使用它找到的第一个应用中的模板（templates）。</p>
<h3 id="修改密码视图views">修改密码视图（views）</h3>
<p>我们还需要我们的用户在登录成功后可以进行修改密码。我们要集成Django认证（authentication）视图（views）来修改密码。打开<em>account</em>应用中的<em>urls.py</em>文件，添加如下URL模式：</p>
<pre><code class="hljs python"><span class="hljs-comment"># change password urls</span>
url(<span class="hljs-string">r'^password-change/$'</span>,
   <span class="hljs-string">'django.contrib.auth.views.password_change'</span>,
   name=<span class="hljs-string">'password_change'</span>),
url(<span class="hljs-string">r'^password-change/done/$'</span>,
   <span class="hljs-string">'django.contrib.auth.views.password_change_done'</span>,
   name=<span class="hljs-string">'password_change_done'</span>),</code></pre>
<p><em>password_change</em>视图（view）将会操作表单（form）进行修改密码，<em>password_change_done</em>将会显示一条成功信息当用户成功的修改他的密码。让我们为每个视图（view）创建一个模板（template）。</p>
<p>在你的<em>account</em>应用<em>templates/registration/</em>目录下添加一个新的文件命名为<em>password_form.html</em>，在文件中添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Change you password</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Change you password<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Use the form below to change your password.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
    </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Change"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板（template）包含了修改密码的表单（form）。现在，在相同的目录下创建另一个文件，命名为<em>password_change_done.html</em>，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Password changed</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Password changed<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Your password has been successfully changed.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
 </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
 </span></code></pre>
<p>这个模板（template）只包含显示一条成功的信息 当用户成功的修改他们的密码。</p>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/account/password-change/" class="uri">http://127.0.0.1:8000/account/password-change/</a> 。如果你的用户没有登录，浏览器会重定向你到登录页面。在你成功认证（authentication）登录后，你会看到如下图所示的修改密码页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-7.png" alt="django-4-7"></p>
<p>在表单（form）中填写你的旧密码和新密码，然后点击<strong>Change</strong>按钮。你会看到如下所示的成功页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-8.png" alt="django-4-8"></p>
<p>登出再使用新的密码进行登录来验证每件事是否如预期一样工作。</p>
<h3 id="重置密码视图views">重置密码视图（views）</h3>
<p>在<em>account</em>应用<em>urls.py</em>文件中为密码重置添加如下URL模式：</p>
<pre><code class="hljs python"><span class="hljs-comment"># restore password urls</span>
 url(<span class="hljs-string">r'^password-reset/$'</span>,
    <span class="hljs-string">'django.contrib.auth.views.password_reset'</span>,
    name=<span class="hljs-string">'password_reset'</span>),
 url(<span class="hljs-string">r'^password-reset/done/$'</span>,
     <span class="hljs-string">'django.contrib.auth.views.password_reset_done'</span>,
     name=<span class="hljs-string">'password_reset_done'</span>),
  url(<span class="hljs-string">r'^password-reset/confirm/(?P&lt;uidb64&gt;[-\w]+)/(?P&lt;token&gt;[-\w]+)/$'</span>,
       <span class="hljs-string">'django.contrib.auth.views.password_reset_confirm'</span>,
     name=<span class="hljs-string">'password_reset_confirm'</span>),
 url(<span class="hljs-string">r'^password-reset/complete/$'</span>,
     <span class="hljs-string">'django.contrib.auth.views.password_reset_complete'</span>,
      name=<span class="hljs-string">'password_reset_complete'</span>),</code></pre>
<p>在你的<em>account</em>应用<em>templates/registration/</em>目录下添加一个新的文件命名为<em>password_reset_form.html</em>，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Reset your password</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Forgotten your password?<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Enter your e-mail address to obtain a new password.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
    </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Send e-mail"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>现在，在相同的目录下添加另一个文件命名为<em>password_reset_email.html</em>，为它添加如下代码：</p>
<pre><code class="hljs sql">Someone asked for password <span class="hljs-keyword">reset</span> <span class="hljs-keyword">for</span> email <span>{</span><span>{</span> email <span>}</span><span>}</span>. Follow the <span class="hljs-keyword">link</span> below:
<span>{</span><span>{</span> protocol <span>}</span><span>}</span>://<span>{</span><span>{</span> <span class="hljs-keyword">domain</span> <span>}</span><span>}</span><span>{</span><span>%</span> <span class="hljs-keyword">url</span> <span class="hljs-string">"password_reset_confirm"</span> uidb64=uid token=token <span>%</span><span>}</span>
Your username, <span class="hljs-keyword">in</span> <span class="hljs-keyword">case</span> you<span class="hljs-string">'ve forgotten: <span>{</span><span>{</span> user.get_username <span>}</span><span>}</span></span></code></pre>
<p>这个模板（template）会被用来渲染发送给用户的重置密码邮件。</p>
<p>在相同目录下添加另一个文件命名为*password_reset_done.html，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Reset your password</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Reset your password<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>We've emailed you instructions for setting your password.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>If you don't receive an email, please make sure you've entered the address you registered with.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>再创建另一个模板（template）命名为<em>passowrd_reset_confirm.html</em>，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Reset your password</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Reset your password<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> validlink <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Please enter your new password twice:<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
      </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
      </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
      <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Change my password"</span> /&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>The password reset link was invalid, possibly because it has already been used. Please request a new password reset.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>在以上模板中，如果重置链接是有效的我们将会进行检查。Django重置密码视图（view）会设置这个变量然后将它带入这个模板（template）的上下文环境中。如果重置链接有效，我们展示用户密码重置表单（form）。</p>
<p>创建另一个模板（template）命名为<em>password_reset_complete.html</em>，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Password reset</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Password set<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Your password has been set. You can <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "login" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>log in now<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>最后，编辑<em>account</em>应用中的<em>/registration/login.html</em>模板（template），添加如下代码在<code>&lt;form&gt;</code>元素之后：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "password_reset" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Forgotten your password?<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/account/login/" class="uri">http://127.0.0.1:8000/account/login/</a> 然后点击<strong>Forgotten your password?</strong>链接。你会看到如下图所示页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-9.png" alt="django-4-9"></p>
<p>在这部分，你需要在你项目中的<em>settings.py</em>中添加一个SMTP配置，这样Django才能发送e-mails。我们已经学习过如何添加e-mail配置在<em>第二章 使用高级特性来优化你的blog</em>。当然，在开发期间，我们可以配置Django在标准输出中输出e-mail内容来代替通过SMTP服务发送邮件。Django提供一个e-mail后端来输出e-mail内容到控制器中。编辑项目中<em>settings.py</em>文件，添加如下代码：</p>
<pre><code class="hljs ini"><span class="hljs-attr">EMAIL_BACKEND</span> = <span class="hljs-string">'django.core.mail.backends.console.EmailBackend'</span></code></pre>
<p><em>EMAIL_BACKEND</em>设置这个类用来发送e-mails。</p>
<p>回到你的浏览器中，填写一个存在用户的e-mail地址，然后点击<strong>Send e-mail</strong>按钮。你会看到如下图所示页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-10.png" alt="django-4-10"></p>
<p>当你运行开发服务的时候看眼控制台输出。你会看到如下所示生成的e-mail：</p>
<pre><code class="hljs sql">IME-Version: 1.0
 Content-Type: text/plain; charset="utf-8"
Content-Transfer-Encoding: 7bit
Subject: Password <span class="hljs-keyword">reset</span> <span class="hljs-keyword">on</span> <span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8000</span>
<span class="hljs-keyword">From</span>: webmaster@localhost
<span class="hljs-keyword">To</span>: <span class="hljs-keyword">user</span>@<span class="hljs-keyword">domain</span>.com
<span class="hljs-built_in">Date</span>: Thu, <span class="hljs-number">24</span> Sep <span class="hljs-number">2015</span> <span class="hljs-number">14</span>:<span class="hljs-number">35</span>:<span class="hljs-number">08</span> <span class="hljs-number">-0000</span>
Message-<span class="hljs-keyword">ID</span>: &lt;<span class="hljs-number">20150924143508.62996</span><span class="hljs-number">.55653</span>@zenx.<span class="hljs-keyword">local</span>&gt;
Someone asked <span class="hljs-keyword">for</span> <span class="hljs-keyword">password</span> <span class="hljs-keyword">reset</span> <span class="hljs-keyword">for</span> email <span class="hljs-keyword">user</span>@<span class="hljs-keyword">domain</span>.com. Follow the <span class="hljs-keyword">link</span> below:
<span class="hljs-keyword">http</span>://<span class="hljs-number">127.0</span><span class="hljs-number">.0</span><span class="hljs-number">.1</span>:<span class="hljs-number">8000</span>/<span class="hljs-keyword">account</span>/<span class="hljs-keyword">password</span>-<span class="hljs-keyword">reset</span>/<span class="hljs-keyword">confirm</span>/MQ/<span class="hljs-number">45</span><span class="hljs-keyword">f</span><span class="hljs-number">-9</span>c3f30caafd523055fcc/
Your username, <span class="hljs-keyword">in</span> <span class="hljs-keyword">case</span> you<span class="hljs-string">'ve forgotten: zenx</span></code></pre>
<p>这封e-mail被我们之前创建的<em>password_reset_email.html</em>模板给渲染。这个给你重置密码的URL带有一个Django动态生成的token。在浏览器中打开这个链接，你会看到如下所示页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-11.png" alt="django-4-11"></p>
<p>这个页面用来设置新密码对应<em>password_reset_confirm.html</em>模板（template）。填写新的密码然后点击<strong>Change my password button</strong>。Django会创建一个新的加密后密码保存进数据库，你会看到如下图所示页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-12.png" alt="django-4-12"></p>
<p>现在你可以使用你的新密码登录你的账号。每个用来设置新密码的token只能使用一次。如果你再次打开你之前获取的链接，你会得到一条信息告诉你这个token已经无效了。</p>
<p>你在项目中集成了Django认证（authentication）框架的视图（views）。这些视图（views）对大部分的例子都适合。当然，你能创建你的自己视图如果你需要一种不同的行为。</p>
<h2 id="用户注册和用户profiles">用户注册和用户profiles</h2>
<p>现有的用户已经可以登录，登出，修改他们的密码，以及当他们忘记密码的时候重置他们的密码。现在，我们需要构建一个视图（view）允许访问者创建他们的账号。</p>
<h3 id="用户注册">用户注册</h3>
<p>让我们创建一个简单的视图（view）允许用户在我们的网站中进行注册。首先，我们需要创建一个表单（form）让用户填写用户名，他们的真实姓名以及密码。编辑<em>account</em>应用新目录下的<em>forms.py</em>文件添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserRegistrationForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    password = forms.CharField(label=<span class="hljs-string">'Password'</span>,
                               widget=forms.PasswordInput)
    password2 = forms.CharField(label=<span class="hljs-string">'Repeat password'</span>,
                                widget=forms.PasswordInput)
    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        model = User
        fields = (<span class="hljs-string">'username'</span>, <span class="hljs-string">'first_name'</span>, <span class="hljs-string">'email'</span>)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">clean_password2</span><span class="hljs-params">(self)</span>:</span>
        cd = self.cleaned_data
        <span class="hljs-keyword">if</span> cd[<span class="hljs-string">'password'</span>] != cd[<span class="hljs-string">'password2'</span>]:
            <span class="hljs-keyword">raise</span> forms.ValidationError(<span class="hljs-string">'Passwords don\'t match.'</span>)
        <span class="hljs-keyword">return</span> cd[<span class="hljs-string">'password2'</span>]
        </code></pre>
<p>我们为<em>User</em>模型（model）创建了一个model表单（form）。在我们的表单（form）中我们只包含了模型（model）中的<em>username,first_name,email</em>字段。这些字段会在它们对应的模型（model）字段上进行验证。例如：如果用户选择了一个已经存在的用户名，将会得到一个验证错误。我们还添加了两个额外的字段<em>password</em>和<em>password2</em>给用户用来填写他们的新密码和确定密码。我们定义了一个<code>clean_password2()</code>方法去检查第二次输入的密码是否和第一次输入的保持一致，如果不一致这个表单将会是无效的。这个检查会被执行当我们验证这个表单（form）通过调用它的<code>is_valid()</code>方法。你可以提供一个<code>clean_&lt;fieldname&gt;()</code>方法给任何一个你的表单（form）字段用来清理值或者抛出表单（from）指定的字段的验证错误。表单（forms）还包含了一个<code>clean()</code>方法用来验证表单（form）的所有内容，这是非常有用的用来验证需要依赖其他字段的字段。</p>
<p>Django还提供一个<em>UserCreationForm</em>表单（form）给你使用，它位于<em>django.contrib.auth.forms</em>非常类似与我们刚才创建的表单（form）。</p>
<p>编辑<em>account</em>应用中的<em>views.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> .forms <span class="hljs-keyword">import</span> LoginForm, UserRegistrationForm
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">register</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:
        user_form = UserRegistrationForm(request.POST)
        <span class="hljs-keyword">if</span> user_form.is_valid():
            <span class="hljs-comment"># Create a new user object but avoid saving it yet</span>
            new_user = user_form.save(commit=<span class="hljs-keyword">False</span>)
            <span class="hljs-comment"># Set the chosen password</span>
            new_user.set_password(
                user_form.cleaned_data[<span class="hljs-string">'password'</span>])
            <span class="hljs-comment"># Save the User object</span>
            new_user.save()
            <span class="hljs-keyword">return</span> render(request,
                         <span class="hljs-string">'account/register_done.html'</span>,
                         {<span class="hljs-string">'new_user'</span>: new_user})
    <span class="hljs-keyword">else</span>:
        user_form = UserRegistrationForm()
    <span class="hljs-keyword">return</span> render(request,
                 <span class="hljs-string">'account/register.html'</span>,
                 {<span class="hljs-string">'user_form'</span>: user_form})</code></pre>
<p>这个创建用户账号的视图（view）非常简单。为了保护用户的隐私，我们使用<em>User</em>模型（model）的<code>set_password()</code>方法将用户的原密码进行加密后再进行保存操作。</p>
<p>现在，编辑<em>account</em>应用中的<em>urls.py</em>文件，添加如下URL模式：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^register/$'</span>, views.register, name=<span class="hljs-string">'register'</span>),</code></pre>
<p>最后，创建一个新的模板（template）在<em>account/</em>模板（template）目录下，命名为<em>register.html</em>,为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Create an account</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Create an account<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Please, sign up using the following form:<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
    </span><span class="hljs-template-variable"><span>{</span><span>{</span> user_form.as_p <span>}</span><span>}</span></span><span class="xml">
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Create my account"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>在相同的目录中添加一个模板（template）文件命名为<em>register_done.html</em>，为它添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Welcome</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
  <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Welcome </span><span class="hljs-template-variable"><span>{</span><span>{</span> new_user.first_name <span>}</span><span>}</span></span><span class="xml">!<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Your account has been successfully created. Now you can <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "login" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>log in<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>现在，在浏览器中打开 <a href="http://127.0.0.1:8000/account/register/" class="uri">http://127.0.0.1:8000/account/register/</a> 。你会看到你创建的注册页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-13.png" alt="django-4-13"></p>
<p>填写用户信息然后点击<strong>Create my account</strong>按钮。如果所有的字段都验证都过，这个用户将会被创建然后会得到一条成功信息，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-14.png" alt="django-4-14"></p>
<p>点击<strong>log-in</strong>链接输入你的用户名和密码来验证账号是否成功创建。</p>
<p>现在，你还可以添加一个注册链接在你的登录模板（template）中。编辑<em>registration/login.html</em>模板（template）然后替换以下内容：</p>
<pre><code class="hljs fortran">&lt;p&gt;Please, <span class="hljs-keyword">use</span> the following <span class="hljs-keyword">form</span> to <span class="hljs-built_in">log</span>-<span class="hljs-keyword">in</span>:&lt;/p&gt;</code></pre>
<p>为：</p>
<pre><code class="hljs vbnet">&lt;p&gt;Please, use the following form <span class="hljs-keyword">to</span> log-<span class="hljs-keyword">in</span>. <span class="hljs-keyword">If</span> you don<span class="hljs-comment">'t have an account <span class="hljs-doctag">&lt;a href="<span>{</span><span>%</span> url "register" <span>%</span><span>}</span>"&gt;</span>register here<span class="hljs-doctag">&lt;/a&gt;</span><span class="hljs-doctag">&lt;/p&gt;</span></span></code></pre>
<p>如此，我们就可以从登录页面进入注册页面。</p>
<h3 id="扩展user模型model">扩展User模型（model）</h3>
<p>当你需要处理用户账号，你会发现Django认证（authentication）框架的<em>User</em>模型（model）只适应一般的案例。无论如何，<em>User</em>模型（model）只有一些最基本的字段。你可能希望扩展<em>User</em>模型包含额外的数据。最好的办法就是创建一个<em>profile</em>模型（model）包含所有额外的字段并且和Django的<em>User</em>模型（model）做一对一的关联。</p>
<p>编辑<em>account</em>应用中的<em>model.py</em>文件，添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.db <span class="hljs-keyword">import</span> models
<span class="hljs-keyword">from</span> django.conf <span class="hljs-keyword">import</span> settings
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Profile</span><span class="hljs-params">(models.Model)</span>:</span>
    user = models.OneToOneField(settings.AUTH_USER_MODEL)
    date_of_birth = models.DateField(blank=<span class="hljs-keyword">True</span>, null=<span class="hljs-keyword">True</span>)
    photo = models.ImageField(upload_to=<span class="hljs-string">'users/%Y/%m/%d'</span>, blank=<span class="hljs-keyword">True</span>)
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__str__</span><span class="hljs-params">(self)</span>:</span>
    <span class="hljs-keyword">return</span> <span class="hljs-string">'Profile for user {}'</span>.format(self.user.username)</code></pre>
<blockquote>
<p>为了保持你的代码通用化，使用<code>get_user_model()</code>方法来取回用户模型（model），当定义了模型（model）和用户模型的关系使用<em>AUTH_USER_MODEL</em>设置来引用它，替代直接引用auth的<em>User</em>模型（model)。</p>
</blockquote>
<p><em>user</em>一对一字段允许我们关联用户和profiles。<em>photo</em>字段是一个<em>ImageField</em>字段。你需要安装一个Python包来管理图片，使用<strong>PIL(Python Imaging Library)</strong>或者<strong>Pillow</strong>（PIL的分叉）,在shell中运行一下命令来安装<em>Pillow</em>：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> Pillow==<span class="hljs-number">2.9</span>.<span class="hljs-number">0</span></code></pre>
<p>为了Django能在开发服务中管理用户上传的多媒体文件，在项目<em>setting.py</em>文件中添加如下设置：</p>
<pre><code class="hljs ini"><span class="hljs-attr">MEDIA_URL</span> = <span class="hljs-string">'/media/'</span>
<span class="hljs-attr">MEDIA_ROOT</span> = os.path.join(BASE_DIR, <span class="hljs-string">'media/'</span>)</code></pre>
<p>MEDIA_URL 是管理用户上传的多媒体文件的主URL，MEDIA_ROOT是这些文件在本地保存的路径。我们动态的构建这些路径相对我们的项目路径来确保我们的代码更通用化。</p>
<p>现在，编辑<em>bookmarks</em>项目中的主<em>urls.py</em>文件，修改代码如下所示：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> include, url
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> django.conf <span class="hljs-keyword">import</span> settings
<span class="hljs-keyword">from</span> django.conf.urls.static <span class="hljs-keyword">import</span> static
urlpatterns = [
    url(<span class="hljs-string">r'^admin/'</span>, include(admin.site.urls)),
    url(<span class="hljs-string">r'^account/'</span>, include(<span class="hljs-string">'account.urls'</span>)),
]
<span class="hljs-keyword">if</span> settings.DEBUG:
    urlpatterns += static(settings.MEDIA_URL,
                        document_root=settings.MEDIA_ROOT)</code></pre>
<p>在这种方法中，Django开发服务器将会在开发时改变对多媒体文件的服务。</p>
<p><em>static()</em>帮助函数最适合在开发环境中使用而不是在生产环境使用。绝对不要在生产环境中使用Django来服务你的静态文件。</p>
<p>打开终端运行以下命令来为新的模型（model）创建数据库迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你会获得以下输出：</p>
<pre><code class="hljs groovy">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'account'</span>:
    <span class="hljs-number">0001</span>_initial.<span class="hljs-string">py:</span>
        - Create model Profile
        </code></pre>
<p>接着，同步数据库通过以下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会看到包含以下内容的输出：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">account</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>编辑<em>account</em>应用中的<em>admin.py</em>文件，在管理站点注册<em>Profiel</em>模型（model），如下所示：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Profile
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ProfileAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display = [<span class="hljs-string">'user'</span>, <span class="hljs-string">'date_of_birth'</span>, <span class="hljs-string">'photo'</span>]
admin.site.register(Profile, ProfileAdmin)</code></pre>
<p>使用`python manage.py runnserver<em>命令重新运行开发服务。现在，你可以看到</em>Profile*模型已经存在你项目中的管理站点中，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-15.png" alt="django-4-15"></p>
<p>现在，我们要让用户可以在网站编辑它们的<em>profile</em>。添加如下的模型（model）表单（forms）到<em>account</em>应用中的<em>forms.py</em>文件：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Profile
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">UserEditForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        model = User
        fields = (<span class="hljs-string">'first_name'</span>, <span class="hljs-string">'last_name'</span>, <span class="hljs-string">'email'</span>)
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ProfileEditForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        model = Profile
        fields = (<span class="hljs-string">'date_of_birth'</span>, <span class="hljs-string">'photo'</span>)</code></pre>
<p>这两个表单（forms）的功能：</p>
<ul>
<li>UserEditForm：允许用户编辑它们的<em>first name,last name, e-mail</em>,这些储存在<em>User</em>模型（model）中的内置字段。</li>
<li>ProfileEditForm：允许用户编辑我们存储在定制的<em>Profile</em>模型（model）中的额外数据。用户可以编辑它们的生日数据以及为他们的<em>profile</em>上传一张图片。</li>
</ul>
<p>编辑<em>account</em>应用中的<em>view.py</em>文件，导入<em>Profile</em>模型（model），如下所示：</p>
<pre><code class="hljs swift">from .models <span class="hljs-keyword">import</span> Profile</code></pre>
<p>然后添加如下内容到<em>register</em>视图（view）中的<code>new_user.save()</code>下方：</p>
<pre><code class="hljs ini"><span class="hljs-comment"># Create the user profile</span>
<span class="hljs-attr">profile</span> = Profile.objects.create(user=new_user)</code></pre>
<p>当用户在我们的站点中注册，我们会创建一个对应的空的<em>profile</em>给他们。你需要为之前创建的用户们一个个手动创建一个<em>Profile</em>对象在管理站点中。</p>
<p>现在，我们要让用于能够编辑他们的<em>pfofile</em>。添加如下代码到相同文件中：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> .forms <span class="hljs-keyword">import</span> LoginForm, UserRegistrationForm, \
UserEditForm, ProfileEditForm
<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">edit</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:
        user_form = UserEditForm(instance=request.user,
                                data=request.POST)
        profile_form = ProfileEditForm(instance=request.user.profile,
                                        data=request.POST,
                                        files=request.FILES)
        <span class="hljs-keyword">if</span> user_form.is_valid() <span class="hljs-keyword">and</span> profile_form.is_valid():
            user_form.save()
            profile_form.save()
    <span class="hljs-keyword">else</span>:
        user_form = UserEditForm(instance=request.user)
        profile_form = ProfileEditForm(instance=request.user.profile)
    <span class="hljs-keyword">return</span> render(request,
                 <span class="hljs-string">'account/edit.html'</span>,
                 {<span class="hljs-string">'user_form'</span>: user_form,
                 <span class="hljs-string">'profile_form'</span>: profile_form})</code></pre>
<p>我们使用<em>login_required</em>装饰器<em>decorator</em>是因为用户编辑他们的<em>profile</em>必须是认证通过的状态。在这个例子中，我们使用两个模型（model）表单（forms）：<em>UserEditForm</em>用来存储数据到内置的<em>User</em>模型（model）中，<em>ProfileEditForm</em>用来存储额外的<em>profile</em>数据。为了验证提交的数据，我们检查每个表单（forms）通过<code>is_valid()</code>方法是否都返回<em>True</em>。在这个例子中，我们保存两个表单（form）来更新数据库中对应的对象。</p>
<p>在<em>account</em>应用中的<em>urls.py</em>文件中添加如下URL模式：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^edit/$'</span>, views.edit, name=<span class="hljs-string">'edit'</span>),</code></pre>
<p>最后，在<em>templates/account/</em>中创建一个新的模板（template）命名为<em>edit.html</em>，为它添加如下内容：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Edit your account</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Edit your account<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>You can edit your account using the following form:<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span> <span class="hljs-attr">enctype</span>=<span class="hljs-string">"multipart/form-data"</span>&gt;</span>
        </span><span class="hljs-template-variable"><span>{</span><span>{</span> user_form.as_p <span>}</span><span>}</span></span><span class="xml">
        </span><span class="hljs-template-variable"><span>{</span><span>{</span> profile_form.as_p <span>}</span><span>}</span></span><span class="xml">
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Save changes"</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<blockquote>
<p>我们在表单（form）中包含<code>enctype="multipart/form-data"</code>用来支持文件上传。我们使用一个HTML表单来提交两个<em>user_form</em>和<em>profile_form</em>表单（forms）。</p>
</blockquote>
<p>注册一个新用户然后打开 <a href="http://127.0.0.1:8000/account/edit/%E3%80%82%E4%BD%A0%E4%BC%9A%E7%9C%8B%E5%88%B0%E5%A6%82%E4%B8%8B%E6%89%80%E7%A4%BA%E9%A1%B5%E9%9D%A2" class="uri">http://127.0.0.1:8000/account/edit/。你会看到如下所示页面</a>：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-16.png" alt="django-4-16"></p>
<p>现在，你可以编辑dashboard页面包含编辑<em>profile</em>的页面链接和修改密码的页面链接。打开<em>account/dashboard.html</em>模板（model）替换如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Welcome to your dashboard.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre>
<p>为：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Welcome to your dashboard. You can <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "edit" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>edit your profile<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span> or <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "password_change" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>change your password<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre>
<p>用户现在可以从他们的dashboard访问编辑他们的<em>profile</em>的表单。</p>
<h3 id="使用一个定制user模型model">使用一个定制<em>User</em>模型（model）</h3>
<p>Django还提供一个方法可以使用你自己定制的模型（model）来替代整个<em>User</em>模型（model）。你自己的用户类需要继承Django的<em>AbstractUser</em>类，这个类提供了一个抽象的模型（model）用来完整执行默认用户。你可访问 <a href="https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#substituting-a-custom-user-model%E6%9D%A5%E8%8E%B7%E5%BE%97%E8%BF%99%E4%B8%AA%E6%96%B9%E6%B3%95%E7%9A%84%E6%9B%B4%E5%A4%9A%E4%BF%A1%E6%81%AF" class="uri">https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#substituting-a-custom-user-model来获得这个方法的更多信息</a>。</p>
<p>使用一个定制的用户模型（model）将会带给你很多的灵活性，但是它也可能给一些需要与<em>User</em>模型（model）交互的即插即用的应用集成带来一定的困难。</p>
<h2 id="使用messages框架">使用messages框架</h2>
<p>当处理用户的操作时，你可能想要通知你的用户关于他们操作的结果。Django有一个内置的messages框架允许你给你的用户显示一次性的提示。messages框架位于<em>django.contrib.messages</em>，当你使用<code>python manage.py startproject</code>命令创建一个新项目的时候，messages框架就被默认包含在<em>settings.py</em>文件中的<em>INSTALLED_APPS</em>中。你会注意到你的设置文件包含了一个名为<em>django.contrib.messages.middleware.MessageMiddleware</em>的中间件在<em>MIDDLEWARE_CLASSES</em>设置中。messages框架提供了一个简单的方法添加消息给用户。消息被存储在数据库中并且会在用户的下一次请求中展示。你可以在你的视图（views）中导入<em>messages</em>模块使用消息messages框架，用简单的快捷方式添加新的messages，如下所示：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> messages
messages.error(request, <span class="hljs-string">'Something went wrong'</span>)</code></pre>
<p>你可以使用<code>add_message()</code>方法创建新的messages或用下方任意一个快捷方法：</p>
<ul>
<li>success()：当操作成功后显示成功的messages</li>
<li>info()：展示messages</li>
<li>warning()：某些还没有达到失败的程度但已经包含有失败的风险，警报用</li>
<li>error()：操作没有成功或者某些事情失败</li>
<li>debug()：在生产环境中这种messages会移除或者忽略</li>
</ul>
<p>让我们显示messages给用户。因为messages框架是被项目全局应用，我们可以在主模板（template）诶用户展示messages。打开<em>base.html</em>模板（template）在id为<em>header</em>的<code>&lt;div&gt;</code>和id为<em>content</em>的<code>&lt;div&gt;</code>之间添加如下内容：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> messages <span>%</span><span>}</span></span><span class="xml">
 <span class="hljs-tag">&lt;<span class="hljs-name">ul</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"messages"</span>&gt;</span>
   </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> message <span class="hljs-keyword">in</span> messages <span>%</span><span>}</span></span><span class="xml">
     <span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> message.tags <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
    </span><span class="hljs-template-variable"><span>{</span><span>{</span> message|<span class="hljs-name">safe</span> <span>}</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"#"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"close"</span>&gt;</span> <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
     <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
   </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
 <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>messages框架带有一个上下文环境（context）处理器用来添加一个<em>messages</em>变量给请求的上下文环境（context）。所以你可以在模板（template）中使用这个变量用来给用户显示当前的messages。</p>
<p>现在，让我们修改<em>edit</em>视图（view）来使用messages框架。编辑应用中的<em>views.py</em>文件，使<em>edit</em>视图（view）如下所示：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> messages
<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">edit</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:
    <span class="hljs-comment"># ...</span>
        <span class="hljs-keyword">if</span> user_form.is_valid() <span class="hljs-keyword">and</span> profile_form.is_valid():
            user_form.save()
            profile_form.save()
            messages.success(request, <span class="hljs-string">'Profile updated '</span>\
                                     <span class="hljs-string">'successfully'</span>)
        <span class="hljs-keyword">else</span>:
            messages.error(request, <span class="hljs-string">'Error updating your profile'</span>)
    <span class="hljs-keyword">else</span>:
        user_form = UserEditForm(instance=request.user)
        <span class="hljs-comment"># ...    </span>
        </code></pre>
<p>当用户成功的更新他们的profile时我们就添加了一条成功的message，但如果某个表单（form）无效，我们就添加一个错误message。</p>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/account/edit/" class="uri">http://127.0.0.1:8000/account/edit/</a> 编辑你的profile。当profile更新成功，你会看到如下message：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-17.png" alt="django-4-17"></p>
<p>当表单（form）是无效的，你会看到如下message：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-18.png" alt="django-4-18"></p>
<h2 id="创建一个定制的认证authentication后台">创建一个定制的认证（authentication）后台</h2>
<p>Django允许你通过不同的来源进行认证（authentication）。<em>AUTHENTICATION_BACKENDS</em>设置包含了所有的认证（authentication）后台给你的项目。默认的，这个设置如下所示：</p>
<pre><code class="hljs clojure">(<span class="hljs-name">'django.contrib.auth.backends.ModelBackend'</span>,)</code></pre>
<p>这个默认的<em>ModelBackend</em>认证（authentication）用户通过数据库使用的是<em>django.contrib.auth</em>中的<em>User</em>模型（model）。这适用于你的大部分项目。当然，你还可以创建定制的后台用来认证你的用户通过其他的来源例如一个<em>LDAP</em>目录或者其他任何系统。</p>
<p>你可以通过访问 <a href="https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#other-authentication-sources" class="uri">https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#other-authentication-sources</a> 获得更多的信息关于自定义的认证（authentication）。</p>
<p>当你使用<em>django.contrib.auth</em>的<em>authenticate()</em>函数，Django会尝试认证（authentication）用户一个接一个通过每一个定义在<em>AUTHENTICATION_BACKENDS</em>中的后台，直到其中有一个后台成功的认证该用户才会停止进行认证。只有所有的后台都无法进行用户认证（authentication），他或她才不会在你的站点中认证（authentication）通过。</p>
<p>Django提供了一个简单的方法来定义你自己的认证（authentication）后台。一个认证（authentication）后台就是一个类提供了如下两种方法：</p>
<ul>
<li>authenticate()：将用户信息当成参数，如果用户成功的认证（authentication）就需要返回<em>True</em>，反之，需要返回<em>False</em>。</li>
<li>get_user()：将用户的ID当成参数然后需要返回一个用户对象。</li>
</ul>
<p>创建一个定制认证（authentication）后台非常容易就是编写一个Python类实现上面两个方法。我们要创建一个认证（authentication）后台让用户在我们的站点中使用他们e-mail替代他们的用户名来进行认证（authentication）。</p>
<p>在你的<em>account</em>应用中创建一个新的文件命名为<em>authentication.py</em>，为它添加如下代码：</p>
<pre><code class="hljs python"><span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">EmailAuthBackend</span><span class="hljs-params">(object)</span>:</span>
    <span class="hljs-string">"""
    Authenticate using e-mail account.
    """</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">authenticate</span><span class="hljs-params">(self, username=None, password=None)</span>:</span>
        <span class="hljs-keyword">try</span>:
            user = User.objects.get(email=username)
            <span class="hljs-keyword">if</span> user.check_password(password):
                <span class="hljs-keyword">return</span> user
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">None</span>
        <span class="hljs-keyword">except</span> User.DoesNotExist:
            <span class="hljs-keyword">return</span> <span class="hljs-keyword">None</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_user</span><span class="hljs-params">(self, user_id)</span>:</span>
        <span class="hljs-keyword">try</span>:
            <span class="hljs-keyword">return</span> User.objects.get(pk=user_id)
        <span class="hljs-keyword">except</span> User.DoesNotExist:
<span class="hljs-keyword">return</span> <span class="hljs-keyword">None</span></code></pre>
<p>这是一个简单的认证（authentication）后台。<code>authenticate()</code>方法接收了<em>username</em>和<em>password</em>两个可选参数。我们可以使用不同的参数，但是我们需要使用<em>username</em>和<em>password</em>来确保我们的后台可以立马在认证（authentication）框架视图（views）中工作。以上代码完成了以下工作内容：</p>
<ul>
<li>authenticate()：我们尝试获取一个用户通过给予的e-mail地址和使用<em>User</em>模型（model）中内置的<code>check_password()</code>方法来检查密码。这个方法会对给予密码进行哈希化来和数据库中存储的加密密码进行匹配。</li>
<li>get_user()：我们获取一个用户通过<em>user_id</em>参数。Django使用这个后台来认证用户之后取回<em>User</em>对象放置到持续的用户会话中。</li>
</ul>
<p>编辑项目中的<em>settings.py</em>文件添加如下设置：</p>
<pre><code class="hljs lisp">AUTHENTICATION_BACKENDS = (
   'django.contrib.auth.backends.ModelBackend',
   'account.authentication.EmailAuthBackend',
)</code></pre>
<p>我们保留默认的<em>ModelBacked</em>用来保证用户仍然可以通过用户名和密码进行认证，接着我们包含进了我们自己的email-based认证（authentication）后台。现在，在浏览器中打开 <a href="http://127.0.0.1:8000/account/login/" class="uri">http://127.0.0.1:8000/account/login/</a> 。请记住，Django会对每个后台都尝试进行用户认证（authentication），所以你可以使用用户名或者使用email来进行无缝登录。</p>
<blockquote>
<p><em>AUTHENTICATION_BACKENDS</em>设置中的后台排列顺序。如果相同的认证信息在多个后台都是有效的，Django会停止在第一个成功认证（authentication）通过的后台，不再继续进行认证（authentication）。</p>
</blockquote>
<h2 id="为你的站点添加社交认证authentication">为你的站点添加社交认证（authentication）</h2>
<p>你可以希望给你的站点添加一些社交认证（authentication）服务，例如 <em>Facebook</em>，<em>Twitter</em>或者<em>Google</em>(国内就算了- -|||)。<em>Python-social-auth</em>是一个Python模块提供了简化的处理为你的网站添加社交认证（authentication）。通过使用这个模块，你可以让你的用户使用他们的其他服务的账号来登录你的网站。你可以访问 <a href="https://github.com/omab/python-social-auth" class="uri">https://github.com/omab/python-social-auth</a> 得到这个模块的代码。</p>
<p>这个模块自带很多认证（authentication）后台给不同的Python框架，其中就包含Django。</p>
<p>使用pip来安装这个包，打开终端运行如下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> python-social-auth==<span class="hljs-number">0.2</span>.<span class="hljs-number">12</span></code></pre>
<p>安装成功后，我们需要在项目<em>settings.py</em>文件中的<em>INSTALLED_APPS</em>设置中添加<em>social.apps.django_app.default</em>：</p>
<pre><code class="hljs lisp">INSTALLED_APPS = (
    <span class="hljs-name">#</span>...
    'social.apps.django_app.default',
)</code></pre>
<p>这个<em>default</em>应用会在Django项目中添加<em>python-social-auth</em>。现在，运行以下命令来同步<em>python-social-auth</em>模型（model）到你的数据库中：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会看到如下<em>default</em>应用的数据迁移输出：</p>
<pre><code class="hljs haskell"><span class="hljs-type">Applying</span> <span class="hljs-keyword">default</span>.0001_initial... <span class="hljs-type">OK</span>
<span class="hljs-type">Applying</span> <span class="hljs-keyword">default</span>.0002_add_related_name... <span class="hljs-type">OK</span>
<span class="hljs-type">Applying</span> <span class="hljs-keyword">default</span>.0003_alter_email_max_length... <span class="hljs-type">OK</span></code></pre>
<p><em>python-social-auth</em>包含了很多服务的后台。你可以访问 <a href="https://python-social-auth.readthedocs.org/en/latest/backends/index.html#supported-backends" class="uri">https://python-social-auth.readthedocs.org/en/latest/backends/index.html#supported-backends</a> 看到所有的后台支持。</p>
<p>我们要包含的认证（authentication）后台包括<em>Facebook</em>，<em>Twitter</em>，<em>Google</em>。</p>
<p>你需要在你的项目中添加社交登录URL模型。打开<em>bookmarks</em>项目中的主<em>urls.py</em>文件，添加如下URL模型：</p>
<pre><code class="hljs cs">url(<span class="hljs-string">'social-auth/'</span>,
    include(<span class="hljs-string">'social.apps.django_app.urls'</span>, <span class="hljs-keyword">namespace</span>=<span class="hljs-string">'social'</span>)),</code></pre>
<p>为了确保社交认证（authentication）可以工作，你还需要配置一个<em>hostname</em>，因为有些服务不允许重定向到<em>127.0.0.1</em>或<em>localhost</em>。为了解决这个问题，在<em>Linux</em>或者<em>Mac OSX</em>下，编辑你的<em>/etc/hosts</em>文件添加如下内容：</p>
<pre><code class="hljs css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span> <span class="hljs-selector-tag">mysite</span><span class="hljs-selector-class">.com</span></code></pre>
<p>这是用来告诉你的计算机指定<em>mysite.com</em> <em>hostname</em>指向你的本地机器。如果你使用<em>Windows</em>,你的hosts文件在 <em>C:\Winwows\ System32\Drivers\etc\hosts</em>。</p>
<p>为了验证你的<em>host</em>重定向是否可用，在浏览器中打开 <a href="http://mysite.com:8000/account/login/" class="uri">http://mysite.com:8000/account/login/</a> 。如果你看到你的应用的登录页面，<em>host</em>重定向已经可用。</p>
<h3 id="使用facebook认证authentication">使用Facebook认证（authentication）</h3>
<p><strong>（译者注：以下的几种社交认证操作步骤可能已经过时，请根据实际情况操作）</strong><br>
为了让你的用户能够使用他们的Facebook账号来登录你的网站，在项目<em>settings.py</em>文件中的<em>AUTHENTICATION_BACKENDS</em>设置中添加如下内容：</p>
<pre><code class="hljs python"><span class="hljs-string">'social.backends.facebook.Facebook2OAuth2'</span>,</code></pre>
<p>为了添加Facebook的社交认证（authentication），你需要一个Facebook开发者账号，然后你必须创建一个新的Facebook应用。在浏览器中打开 <a href="https://developers.facebook.com/apps/?action=create" class="uri">https://developers.facebook.com/apps/?action=create</a> 点击<strong>Add new app</strong>按钮。点击<strong>Website</strong>平台然后为你的应用取名为<em>Bookmarks</em>，输入 <a href="http://mysite.com:8000/" class="uri">http://mysite.com:8000/</a> 作为你的网站URL。跟随快速开始步骤然后点击<strong>Create App ID</strong>。</p>
<p>回到网站的Dashboard。你会看到类似下图所示的：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-19.png" alt="django-4-19"></p>
<p>拷贝<strong>App ID</strong>和<strong>App Secret</strong>关键值，将它们添加在项目中的*settings.py**文件中，如下所示：</p>
<pre><code class="hljs ini"><span class="hljs-attr">SOCIAL_AUTH_FACEBOOK_KEY</span> = <span class="hljs-string">'XXX'</span> # Facebook App ID
<span class="hljs-attr">SOCIAL_AUTH_FACEBOOK_SECRET</span> = <span class="hljs-string">'XXX'</span> # Facebook App Secret</code></pre>
<p>此外，你还可以定义一个<em>SOCIAL_AUTH_FACEBOOK_SCOPE</em>设置如果你想要访问Facebook用户的额外权限，例如：</p>
<pre><code class="hljs ini"><span class="hljs-attr">SOCIAL_AUTH_FACEBOOK_SCOPE</span> = [<span class="hljs-string">'email'</span>]</code></pre>
<p>最后，打开<em>registration/login.html</em>模板（template）然后添加如下代码到<em>content</em> block中：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"social"</span>&gt;</span>
  <span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"facebook"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "social:begin" "facebook" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Sign in with Facebook<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
  <span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span> 
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre>
<p>在浏览器中打开 <a href="http://mysite.com:8000/account/login/" class="uri">http://mysite.com:8000/account/login/</a> 。现在你的登录页面会如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-20.png" alt="django-4-20"></p>
<p>点击**Login with Facebook<em>按钮。你会被重定向到Facebook，然后你会看到一个对话询问你的权限是否让</em>Bookmarks*应用访问你的公共Facebook profile：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-21.png" alt="django-4-21"></p>
<p>点击<strong>Okay</strong>按钮。Python-social-auth会对认证（authentication）进行操作。如果每一步都没有出错，你会登录成功然后被重定向到你的网站的dashboard页面。请记住，我们已经使用过这个URL在<em>LOGIN_REDIRECT_URL</em>设置中。就像你所看到的，在你的网站中添加社交认证（authentication）是非常简单的。</p>
<h3 id="使用twitter认证authentication">使用Twitter认证（authentication）</h3>
<p>为了使用Twitter进行认证（authentication），在项目<em>settings.py</em>中的<em>AUTHENTICATION_BACKENDS</em>设置中添加如下内容：</p>
<pre><code class="hljs python"><span class="hljs-string">'social.backends.twitter.TwitterOAuth'</span>,</code></pre>
<p>你需要在你的Twitter账户中创建一个新的应用。在浏览器中打开 <a href="https://apps.twitter.com/app/new" class="uri">https://apps.twitter.com/app/new</a> 然后输入你应用信息，包含以下设置：</p>
<ul>
<li>Website：<a href="http://mysite.com:8000/" class="uri">http://mysite.com:8000/</a></li>
<li>Callback URL：<a href="http://mysite.com:8000/social-auth/complete/twitter/" class="uri">http://mysite.com:8000/social-auth/complete/twitter/</a></li>
</ul>
<p>确保你勾选了复选款<strong>Allow this application to be used to Sign in with Twitter</strong>。之后点击<strong>Keys and Access Tokens</strong>。你会看到如下所示信息：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-22.png" alt="django-4-22"></p>
<p>拷贝<strong>Consumer Key</strong>和<strong>Consumer Secret</strong>关键值，将它们添加到项目<em>settings.py</em>的设置中，如下所示：</p>
<pre><code class="hljs ini"><span class="hljs-attr">SOCIAL_AUTH_TWITTER_KEY</span> = <span class="hljs-string">'XXX'</span> # Twitter Consumer Key
<span class="hljs-attr">SOCIAL_AUTH_TWITTER_SECRET</span> = <span class="hljs-string">'XXX'</span> # Twitter Consumer Secret</code></pre>
<p>现在，编辑<em>login.html</em>模板（template），在<code>&lt;ul&gt;</code>元素中添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"twitter"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "social:begin" "twitter" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Login with Twitter<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span></code></pre>
<p>在浏览器中打开 <a href="http://mysite.com:8000/account/login/" class="uri">http://mysite.com:8000/account/login/</a> 然后点击<strong>Login with Twitter</strong>链接。你会被重定向到Twitter然后它会询问你授权给应用，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-23.png" alt="django-4-23"></p>
<p>点击<strong>Authorize app</strong>按钮。你会登录成功并且重定向到你的网站dashboard页面。</p>
<h3 id="使用google认证authentication">使用Google认证（authentication）</h3>
<p>Google提供<em>OAuth2</em>认证（authentication）。你可以访问 <a href="https://developers.google.com/accounts/docs/OAuth2" class="uri">https://developers.google.com/accounts/docs/OAuth2</a> 获得关于Google OAuth2的信息。</p>
<p>首先，你徐闯创建一个<em>API key</em>在你的Google开发者控制台。在浏览器中打开 <a href="https://console.developers.google.com/project" class="uri">https://console.developers.google.com/project</a> 然后点击<strong>Create project</strong>按钮。输入一个名字然后点击<strong>Create</strong>按钮，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-24.png" alt="django-4-24"></p>
<p>在项目创建之后，点击在左侧菜单的<strong>APIs &amp; auth</strong>链接，然后点击<strong>Credentials</strong>部分。点击<strong>Add credentials</strong>按钮，然后选择<strong>OAuth2.0 client ID</strong>，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-25.png" alt="django-4-25"></p>
<p>Google首先会询问你配置同意信息页面。这个页面将会展示给用户告知他们是否同意使用他们的Google账号来登录访问你的网站。点击<em>Configure consent screen</em>按钮。选择你的e-mail地址，填写<em>Bookmarks</em>为<strong>Product name</strong>，然后点击<strong>Save</strong>按钮。这个给你的项目使用的同意信息页面将会配置完成然后你会被重定向去完成创建你的Client ID。</p>
<p>在表单（form）中填写以下内容：</p>
<ul>
<li>Application type： 选择<strong>Web application</strong></li>
<li>Name： 输入<em>Bookmarks</em></li>
<li>Authorized redirect URLs：输入 <a href="http://mysite.com:8000/social-auth/complete/google-oauth2/" class="uri">http://mysite.com:8000/social-auth/complete/google-oauth2/</a></li>
</ul>
<p>这表单（form）将会如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-26.png" alt="django-4-26"></p>
<p>点击<strong>Create</strong>按钮。你将会获得<strong>Client ID</strong>和<strong>Client Secret</strong>关键值。在你的<em>settings.py</em>中添加它们，如下所示：</p>
<pre><code class="hljs ini"><span class="hljs-attr">SOCIAL_AUTH_GOOGLE_OAUTH2_KEY</span> = <span class="hljs-string">''</span> # Google Consumer Key
<span class="hljs-attr">SOCIAL_AUTH_GOOGLE_OAUTH2_SECRET</span> = <span class="hljs-string">''</span> # Google Consumer Secret</code></pre>
<p>在Google开发者控制台的左方菜单，<strong>APIs &amp; auth</strong>部分的下方，点击<strong>APIs</strong>链接。你会看到包含所有Google Apis的列表。点击 <strong>Google+ API</strong>然后点击<strong>Enable API</strong>按钮在以下页面中：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-27.png" alt="django-4-27"></p>
<p>编辑<em>login.html</em>模板（template）在<code>&lt;ul&gt;</code>元素中添加如下代码：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"google"</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "social:begin" "google" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Login with Google<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span></code></pre>
<p>在浏览器中打开 <a href="http://mysite.com:8000/account/login/" class="uri">http://mysite.com:8000/account/login/</a> 。登录页面将会看上去如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-28.png" alt="django-4-28"></p>
<p>点击<strong>Login with Google</strong>按钮。你将会被重定向到Google并且被询问权限通过我们之前配置的同意信息页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-4-29.png" alt="django-4-29"></p>
<p>点击<strong>Accept</strong>按钮。你会登录成功并重定向到你的网站的dashboard页面。</p>
<p>我们已经添加了社交认证（authentication）到我们的项目中。python-social-auth模块还包含更多其他非常热门的在线服务。</p>
<h3 id="总结">总结</h3>
<p>在本章中，你学习了如何创建一个认证（authentication）系统到你的网站并且创建了定制的用户profile。你还为你的网站添加了社交认证（authentication）。</p>
<p>在下一章中，你会学习如何创建一个图片收藏系统（image bookmarking system），生成图片缩微图，创建AJAX视图（views）。</p>
</div>