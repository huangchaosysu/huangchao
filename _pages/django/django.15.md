---
layout: single
permalink: /django/example6/
title: "Django By Example 第六章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" target="_blank">https://www.packtpub.com/web-development/django-example</a><br>原作者：Antonio Melé</p>
<h1>第六章</h1>
<h2>跟踪用户动作</h2>
<p>在上一章中，你在你的项目中实现了AJAX视图（views），通过使用jQuery并创建了一个JavaScript书签在你的平台中分享别的网站的内容。</p>
<p>在本章中，你会学习如何创建一个粉丝系统以及创建一个用户活动流（activity stream）。你会学习到Django信号（signals）的工作方式以及在你的项目中集成Redis快速 I/O 仓库用来存储视图（views）项。</p>
<p>本章将会覆盖以下几点：</p>
<ul>
<li>通过一个中介模型（intermediate model）创建多对对的关系</li>
<li>创建 AJAX 视图（views）</li>
<li>创建一个活动流（activity stream）应用</li>
<li>给模型（modes）添加通用关系</li>
<li>取回对象的最优查询集（QuerySets）</li>
<li>使用信号（signals）给非规范化的计数</li>
<li>存储视图（views）项到 Redis 中</li>
</ul>
<h3>创建一个粉丝系统</h3>
<p>我们将要在我们的项目中创建一个粉丝系统。我们的用户在平台中能够彼此关注并且跟踪其他用户的分享。这个关系在用户中的是多对多的关系，一个用户能够关注多个用户并且能被多个用户关注。</p>
<h3>通过一个中介模型（intermediate model）（intermediary model）创建多对对的关系</h3>
<p>在上一章中，你创建了多对对关系通过在其中一个有关联的模型（model）上添加了一个<em>ManyToManyField</em>然后让Django为这个关系创建了数据库表。这种方式支持大部分的场景，但是有时候你需要为这种关系创建一个中介模型（intermediate model）。创建一个中介模型（intermediate model）是非常有必要的当你想要为当前关系存储额外的信息，例如当前关系创建的时间点或者一个描述当前关系类型的字段。</p>
<p>我们会创建一个中介模型（intermediate model）用来在用户之间构建关系。有两个原因可以解释为什么我们要用一个中介模型（intermediate model）：</p>
<ul>
<li>我们使用Django提供的<em>user</em>模型（model）并且我们想要避免修改它。</li>
<li>我们想要存储关系建立的时间</li>
</ul>
<p>编辑你的<em>account</em>应用中的<em>models.py</em>文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User
   <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Contact</span><span class="hljs-params">(models.Model)</span>:</span>
       user_from = models.ForeignKey(User,
                                     related_name=<span class="hljs-string">'rel_from_set'</span>)
       user_to = models.ForeignKey(User,
                                   related_name=<span class="hljs-string">'rel_to_set'</span>)
       created = models.DateTimeField(auto_now_add=<span class="hljs-keyword">True</span>,
                                      db_index=<span class="hljs-keyword">True</span>)
       <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
           ordering = (<span class="hljs-string">'-created'</span>,)
       <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__str__</span><span class="hljs-params">(self)</span>:</span>
           <span class="hljs-keyword">return</span> <span class="hljs-string">'{} follows {}'</span>.format(self.user_from,
self.user_to)</code></pre>
<p>这个<em>Contact</em>模型我们将会给用户关系使用。它包含以下字段：</p>
<ul>
<li>user_form：一个<em>ForeignKey</em>指向创建关系的用户</li>
<li>user_to：一个<em>ForeignKey</em>指向被关注的用户</li>
<li>created：一个<code>auto_now_add=True</code>的<em>DateTimeField</em>字段用来存储关系创建时的时间</li>
</ul>
<p>在<em>ForeignKey</em>字段上会自动生成一个数据库索引。我们使用<code>db_index=True</code>来创建一个数据库索引给<em>created</em>字段。这会提升查询执行的效率当通过这个字段对查询集（QuerySets）进行排序的时候。</p>
<p>使用 ORM ，我们可以创建一个关系给一个用户 <em>user1</em> 关注另一个用户 <em>user2</em>，如下所示：</p>
<pre class="hljs python"><code class="python">user1 = User.objects.get(id=<span class="hljs-number">1</span>)
user2 = User.objects.get(id=<span class="hljs-number">2</span>)
Contact.objects.create(user_from=user1, user_to=user2)</code></pre>
<p>关系管理器 <em>rel_form_set</em> 和 <em>rel_to_set</em> 会返回一个查询集（QuerySets）给<em>Contace</em>模型（model）。为了<br>从<em>User</em>模型（model）中存取最终的关系侧，<em>Contace</em>模型（model）会期望<em>User</em>包含一个<em>ManyToManyField</em>，如下所示（译者注：以下代码是作者假设的，实际上<em>User</em>不会包含以下代码）：</p>
<pre class="hljs python"><code class="python">following = models.ManyToManyField(<span class="hljs-string">'self'</span>,
                                   through=Contact,
                                   related_name=<span class="hljs-string">'followers'</span>,
                                   symmetrical=<span class="hljs-keyword">False</span>)</code></pre>
<p>在这个例子中，我们告诉Django去使用我们定制的中介模型（intermediate model）来创建关系通过给<em>ManyToManyField</em>添加<code>through=Contact</code>。这是一个从<em>User</em>模型到本身的多对对关系：我们在<em>ManyToMnyfIELD</em>字段中引用 <code>'self'</code>来创建一个关系给相同的模型（model）。</p>
<blockquote><p>当你在多对多关系中需要额外的字段，创建一个定制的模型（model)，一个关系侧就是一个<em>ForeignKey</em>。添加一个 <em>ManyToManyField</em> 在其中一个有关联的模型（models）中然后通过在<em>through</em>参数中包含该中介模型（intermediate model）指示Django去使用你的定制中介模型（intermediate model）。</p></blockquote>
<p>如果<em>User</em>模型（model）是我们应用的一部分，我们可以添加以上的字段给模型（model）（译者注：所以说，上面的代码是作者假设存在）。但实际上，我们无法直接修改<em>User</em>类，因为它是属于<em>django.contrib.auth</em>应用的。我们将要做些轻微的改动，给<em>User</em>模型动态的添加这个字段。编辑<em>account</em>应用中的<em>model.py</em>文件，添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-comment"># Add following field to User dynamically</span>
User.add_to_class(<span class="hljs-string">'following'</span>,
                   models.ManyToManyField(<span class="hljs-string">'self'</span>,
                                          through=Contact,
                                          related_name=<span class="hljs-string">'followers'</span>,
                                          symmetrical=<span class="hljs-keyword">False</span>))</code></pre>
<p>在以上代码中，我们使用Django模型（models）的<code>add_to_class()</code>方法给<em>User</em>模型（model）添加<em>monkey-patch</em>(译者注：猴子补丁 <em>Monkey patch</em> 就是在运行时对已有的代码进行修改，而不需要修改原始代码）。你需要意识到，我们不推荐使用<code>add_to_class()</code>为模型（models）添加字段。我们在这个场景中利用这种方法是因为以下的原因：</p>
<ul>
<li>我们可以非常简单的取回关系对象使用Django ORM的<code>user.followers.all()</code>以及<code>user.following.all()</code>。我们使用中介（intermediary） <em>Contact</em> 模型（model）可以避免复杂的查询例如使用到额外的数据库操作<em>joins</em>，如果在我们的定制<em>Profile</em>模型（model）中定义过了关系。</li>
<li>这个多对多关系的表将会被创建通过使用<em>Contact</em>模型（model）。因此，动态的添加<em>ManyToManyField</em>将不会对Django <em>User</em> 模型（model）的数据库进行任意改变。</li>
<li>我们避免了创建一个定义的用户模型（model），保持了所有Django内置<em>User</em>的特性。</li>
</ul>
<p>请记住，在大部分的场景中，在我们之前创建的<em>Profile</em>模型（model）添加字段是更好的方法，可以替代在<em>User</em>模型（model）上打上<em>monkey-patch</em>。Django还允许你使用定制的用户模型（models）。如果你想要使用你的定制用户模型（model），可以访问 <a href="https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#specifying-a-custom-user-model" target="_blank">https://docs.djangoproject.com/en/1.8/topics/auth/customizing/#specifying-a-custom-user-model</a> 获得更多信息。</p>
<p>你能看到上述代码中的关系包含了<code>symmetrical=Flase</code>来定义一个非对称（non-symmetric）关系。这表示如果我关注了你，你不会自动的关注我。</p>
<blockquote><p>当你使用了一个中介模型（intermediate model）给多对多关系，一些关系管理器的方法将不可用，例如：<code>add()</code>，<code>create()</code>以及<code>remove()</code>。你需要创建或删除中介模型（intermediate model）的实例来代替。</p></blockquote>
<p>运行如下命令来生成<em>account</em>应用的初始迁移：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> makemigrations account</code></pre>
<p>你会看到如下输出：</p>
<pre class="hljs python"><code class="python">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'account'</span>:
     <span class="hljs-number">0002</span>_contact.py:
       - Create model Contact</code></pre>
<p>现在继续运行以下命令来同步应用到数据库中：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> migrate account</code></pre>
<p>你会看到如下内容包含在输出中：</p>
<pre class="hljs clean"><code class="clean">Applying account<span class="hljs-number">.0002</span>_contact... OK</code></pre>
<p><em>Contact</em>模型（model）现在已经被同步进了数据库，我们可以在用户之间创建关系。但是，我们的网站还没有提供一个方法来浏览用户或查看详细的用户profile。让我们为<em>User</em>模型构建列表和详情视图（views）。</p>
<h3>为用户profiles创建列表和详情视图（views）</h3>
<p>打开<em>account</em>应用中的<em>views.py</em>文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.shortcuts <span class="hljs-keyword">import</span> get_object_or_404
<span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User
<span class="hljs-meta">   @login_required</span>
   <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">user_list</span><span class="hljs-params">(request)</span>:</span>
       users = User.objects.filter(is_active=<span class="hljs-keyword">True</span>)
       <span class="hljs-keyword">return</span> render(request,
                     <span class="hljs-string">'account/user/list.html'</span>,
                     {<span class="hljs-string">'section'</span>: <span class="hljs-string">'people'</span>,
                      <span class="hljs-string">'users'</span>: users})
<span class="hljs-meta">   @login_required</span>
   <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">user_detail</span><span class="hljs-params">(request, username)</span>:</span>
       user = get_object_or_404(User,
                                username=username,
                                is_active=<span class="hljs-keyword">True</span>)
       <span class="hljs-keyword">return</span> render(request,
                     <span class="hljs-string">'account/user/detail.html'</span>,
                     {<span class="hljs-string">'section'</span>: <span class="hljs-string">'people'</span>,
                      <span class="hljs-string">'user'</span>: user})</code></pre>
<p>以上是<em>User</em>对象的简单列表和详情视图（views）。<code>user_list</code>视图（view）获得了所有的可用用户。Django <em>User</em> 模型（model）包含了一个标志（flag）<code>is_active</code>来指示用户账户是否可用。我们通过<code>is_active=True</code>来过滤查询只返回可用的用户。这个视图（vies）返回了所有结果，但是你可以改善它通过添加页码，这个方法我们在<em>image_list</em>视图（view）中使用过。</p>
<p><em>user_detail</em>视图（view）使用<code>get_object_or_404()</code>快捷方法来返回所有可用的用户通过传入的用户名。当使用传入的用户名无法找到可用的用户这个视图（view）会返回一个HTTP 404响应。</p>
<p>编辑<em>account</em>应用的<em>urls.py</em>文件，为以上两个视图（views）添加URL模式，如下所示：</p>
<pre class="hljs python"><code class="python">urlpatterns = [
       <span class="hljs-comment"># ...</span>
       url(<span class="hljs-string">r'^users/$'</span>, views.user_list, name=<span class="hljs-string">'user_list'</span>),
       url(<span class="hljs-string">r'^users/(?P&lt;username&gt;[-\w]+)/$'</span>,
           views.user_detail,
           name=<span class="hljs-string">'user_detail'</span>),
]</code></pre>
<p>我们会使用 <code>user_detail</code> URL模式来给用户生成规范的URL。你之前就在模型（model）中定义了一个<code>get_absolute_url()</code>方法来为每个对象返回规范的URL。另外一种方式为一个模型（model）指定一个URL是为你的项目添加<em>ABSOLUTE_URL_OVERRIDES</em>设置。</p>
<p>编辑项目中的<em>setting.py</em>文件，添加如下代码：</p>
<pre class="hljs python"><code class="python">ABSOLUTE_URL_OVERRIDES = {
    <span class="hljs-string">'auth.user'</span>: <span class="hljs-keyword">lambda</span> u: reverse_lazy(<span class="hljs-string">'user_detail'</span>,
                                        args=[u.username])
}</code></pre>
<p>Django会为所有出现在<em>ABSOLUTE_URL_OVERRIDES</em>设置中的模型（models）动态添加一个<code>get_absolute_url()</code>方法。这个方法会给设置中指定的模型返回规范的URL。我们给传入的用户返回<em>user_detail</em> URL。现在你可以在一个User实例上使用<code>get_absolute_url()</code>来取回他自身的规范URL。打开Python shell输入命令<code>python manage.py shell</code>运行以下代码来进行测试：</p>
<pre class="hljs stylus"><code class="stylus">&gt;&gt;&gt; from django<span class="hljs-selector-class">.contrib</span><span class="hljs-selector-class">.auth</span><span class="hljs-selector-class">.models</span> import User
&gt;&gt;&gt; user = User<span class="hljs-selector-class">.objects</span><span class="hljs-selector-class">.latest</span>(<span class="hljs-string">'id'</span>)
&gt;&gt;&gt; str(user.get_absolute_url())
<span class="hljs-string">'/account/users/ellington/'</span></code></pre>
<p>返回的URL如同期望的一样。我们需要为我们刚才创建的视图（views）创建模板（templates）。在<em>account</em>应用下的*templates/account/目录下添加以下目录和文件：</p>
<pre class="hljs stylus"><code class="stylus">/user/
    detail<span class="hljs-selector-class">.html</span>
    list.html</code></pre>
<p>编辑<em>account/user/list.html</em>模板（template）给它添加如下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> thumbnail <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">People</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>People<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"people-list"</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> user <span class="hljs-keyword">in</span> users <span>%</span><span>}</span></span><span class="xml">
         <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"user"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
             </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">thumbnail</span> user.profile.photo "180x180" crop="100%" <span class="hljs-keyword">as</span> im <span>%</span><span>}</span></span><span class="xml">
               ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> im.url <span>}</span><span>}</span></span><span class="xml">)
             </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">endthumbnail</span> <span>%</span><span>}</span></span><span class="xml">
           <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
           <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"info"</span>&gt;</span>
             <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"title"</span>&gt;</span>
               </span><span class="hljs-template-variable"><span>{</span><span>{</span> user.get_full_name <span>}</span><span>}</span></span><span class="xml">
             <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span> 
           <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
         <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板（template）允许我们在网站中排列所有可用的用户。我们对给予的用户进行迭代并且使用`<span>{</span><span>%</span> thumbnail <span>%</span><span>}</span>模板（template）标签（tag）来生成profile图片缩微图。</p>
<p>打开项目中的<em>base.html</em>模板（template），在以下菜单项的<em>href</em>属性中包含<em>user_list</em>URL：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> section == "people" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-attr">class</span>=<span class="hljs-string">"selected"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "user_list" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>People<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
<span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span></code></pre>
<p>通过命令<code>python manage.py runserver</code>启动开发服务器然后在浏览器打开 <a href="http://127.0.0.1:8000/account/users/" target="_blank">http://127.0.0.1:8000/account/users/</a> 。你会看到如下所示的用户列：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-d2b101ca88a607d4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-d2b101ca88a607d4.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-1</div>
</div>
<p>（译者注：图灵，特斯拉，爱因斯坦，都是大牛啊）</p>
<p>编辑<em>account</em>应用下的<em>account/user/detail.html</em>模板，添加如下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> thumbnail <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.get_full_name <span>}</span><span>}</span></span><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.get_full_name <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"profile-info"</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">thumbnail</span> user.profile.photo "180x180" crop="100%" <span class="hljs-keyword">as</span> im <span>%</span><span>}</span></span><span class="xml">
        ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> im.url <span>}</span><span>}</span></span><span class="xml">)
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">endthumbnail</span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">with</span></span> total_followers=user.followers.count <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"count"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"total"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> total_followers <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
        follower</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_followers|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"#"</span> <span class="hljs-attr">data-id</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.id <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">data-action</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user <span class="hljs-keyword">in</span> user.followers.all <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">un</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">follow"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"followbutton"</span>&gt;</span>
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user not <span class="hljs-keyword">in</span> user.followers.all <span>%</span><span>}</span></span><span class="xml">
            Follow
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
            Unfollow
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"image-list"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"imget-container"</span>&gt;</span>
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">include</span></span> "images/image/list_ajax.html" with images = user.images_create.all <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endwith</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>在详情模板（template）中我们展示用户profile并且我们使用<code><span>{</span><span>%</span> thumbnail <span>%</span><span>}</span></code>模板（template）标签（tag）来显示profile图片。我们显示粉丝的总数以及一个链接可以 <em>follow/unfollow</em> 该用户。我们会隐藏关注链接当用户在查看他们自己的profile，防止用户自己关注自己。我们会执行一个AJAX请求来 <em>follow/unfollow</em> 一个指定用户。我们给 <code>&lt;a&gt;</code> HTML元素添加<em>data-id</em>和<em>data-action</em>属性包含用户ID以及当该链接被点击的时候会执行的初始操作，<em>follow/unfollow</em> ，这个操作依赖当前页面的展示的用户是否已经被正在浏览的用户所关注。我们展示当前页面用户的图片书签通过<em>list_ajax.html</em>模板。</p>
<p>再次打开你的浏览器，点击一个拥有图片书签的用户链接，你会看到一个profile详情如下所示：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-6dffd599f25f35aa.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-6dffd599f25f35aa.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-2</div>
</div>
<h3>创建一个AJAX视图（view）来关注用户</h3>
<p>我们将会创建一个简单的视图（view）使用AJAX来 <em>follow/unfollow</em> 用户。编辑<em>account</em>应用中的<em>views.py</em>文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.http <span class="hljs-keyword">import</span> JsonResponse
<span class="hljs-keyword">from</span> django.views.decorators.http <span class="hljs-keyword">import</span> require_POST
<span class="hljs-keyword">from</span> common.decorators <span class="hljs-keyword">import</span> ajax_required
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Contact
<span class="hljs-meta">@ajax_required</span>
<span class="hljs-meta">@require_POST</span>
<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">user_follow</span><span class="hljs-params">(request)</span>:</span>
    user_id = request.POST.get(<span class="hljs-string">'id'</span>)
    action = request.POST.get(<span class="hljs-string">'action'</span>)
    <span class="hljs-keyword">if</span> user_id <span class="hljs-keyword">and</span> action:
        <span class="hljs-keyword">try</span>:
            user = User.objects.get(id=user_id)
            <span class="hljs-keyword">if</span> action == <span class="hljs-string">'follow'</span>:
                Contact.objects.get_or_create(
                    user_from=request.user,
                    user_to=user)
            <span class="hljs-keyword">else</span>:
                Contact.objects.filter(user_from=request.user,
                                        user_to=user).delete()
            <span class="hljs-keyword">return</span> JsonResponse({<span class="hljs-string">'status'</span>:<span class="hljs-string">'ok'</span>})
        <span class="hljs-keyword">except</span> User.DoesNotExist:
            <span class="hljs-keyword">return</span> JsonResponse({<span class="hljs-string">'status'</span>:<span class="hljs-string">'ko'</span>})
    <span class="hljs-keyword">return</span> JsonResponse({<span class="hljs-string">'status'</span>:<span class="hljs-string">'ko'</span>})</code></pre>
<p><em>user_follow</em>视图（view）有点类似与我们之前创建的<em>image_like</em>视图（view）。因为我们使用了一个定制中介模型（intermediate model）给用户的多对多关系，所以<em>ManyToManyField</em>管理器默认的<code>add()</code>和<code>remove()</code>方法将不可用。我们使用中介<em>Contact</em>模型（model）来创建或删除用户关系。</p>
<p>在<em>account</em>应用中的<em>urls.py</em>文件中导入你刚才创建的视图（view）然后为它添加URL模式：</p>
<pre class="hljs python"><code class="python">    url(<span class="hljs-string">r'^users/follow/$'</span>, views.user_follow, name=<span class="hljs-string">'user_follow'</span>),</code></pre>
<p>请确保你放置的这个URL模式的位置在<em>user_detail</em>URL模式之前。否则，任何对 <em>/users/follow/</em> 的请求都会被<em>user_detail</em>模式给正则匹配然后执行。请记住，每一次的HTTP请求Django都会对每一条存在的URL模式进行匹配直到第一条匹配成功才会停止继续匹配。</p>
<p>编辑<em>account</em>应用下的<em>user/detail.html</em>模板添加如下代码：</p>
<pre class="hljs javascript"><code class="javascript"><span>{</span><span>%</span> block domready <span>%</span><span>}</span>
     $(<span class="hljs-string">'a.follow'</span>).click(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">e</span>)</span>{
       e.preventDefault();
       $.post(<span class="hljs-string">'<span>{</span><span>%</span> url "user_follow" <span>%</span><span>}</span>'</span>,
         {
           <span class="hljs-attr">id</span>: $(<span class="hljs-keyword">this</span>).data(<span class="hljs-string">'id'</span>),
           <span class="hljs-attr">action</span>: $(<span class="hljs-keyword">this</span>).data(<span class="hljs-string">'action'</span>)
         },
         <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">data</span>)</span>{
           <span class="hljs-keyword">if</span> (data[<span class="hljs-string">'status'</span>] == <span class="hljs-string">'ok'</span>) {
             <span class="hljs-keyword">var</span> previous_action = $(<span class="hljs-string">'a.follow'</span>).data(<span class="hljs-string">'action'</span>);

             <span class="hljs-comment">// toggle data-action</span>
             $(<span class="hljs-string">'a.follow'</span>).data(<span class="hljs-string">'action'</span>,
               previous_action == <span class="hljs-string">'follow'</span> ? <span class="hljs-string">'unfollow'</span> : <span class="hljs-string">'follow'</span>);
             <span class="hljs-comment">// toggle link text</span>
             $(<span class="hljs-string">'a.follow'</span>).text(
               previous_action == <span class="hljs-string">'follow'</span> ? <span class="hljs-string">'Unfollow'</span> : <span class="hljs-string">'Follow'</span>);

             <span class="hljs-comment">// update total followers</span>
             <span class="hljs-keyword">var</span> previous_followers = <span class="hljs-built_in">parseInt</span>(
               $(<span class="hljs-string">'span.count .total'</span>).text());
             $(<span class="hljs-string">'span.count .total'</span>).text(previous_action == <span class="hljs-string">'follow'</span> ? previous_followers + <span class="hljs-number">1</span> : previous_followers - <span class="hljs-number">1</span>);
          }
        }
      });
    });
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre>
<p>这段JavaScript代码执行AJAX请求来关注或不关注一个指定用户并且触发 <em>follow/unfollow</em> 链接。我们使用jQuery去执行AJAX请求的同时会设置 <em>follow/unfollow</em> 两种链接的<em>data-aciton</em>属性以及HTML<code>&lt;a&gt;</code>元素的文本基于它上一次的值。当AJAX操作执行完成，我们还会对显示在页面中的粉丝总数进行更新。打开一个存在的用户的详情页面，然后点击<strong>Follow</strong>链接尝试下我们刚才构建的功能是否正常。</p>
<h3>创建一个通用的活动流（activity stream）应用</h3>
<p>许多社交网站会给他们的用户显示一个活动流（activity stream），这样他们可以跟踪其他用户在平台中的操作。一个活动流（activity stream）是一个用户或一个用户组最近活动的列表。举个例子，<em>Facebook</em>的<em>News Feed</em>就是一个活动流（activity stream）。用户X给Y图片打上了书签或者用户X关注了用户Y也是例子操作。我们将会构建一个活动流（activity stream）应用这样每个用户都能看到他关注的用户最近进行的交互。为了做到上述功能，我们需要一个模型（modes）来保存用户在网站上的操作执行，还需要一个简单的方法来添加操作给feed。</p>
<p>运行以下命令在你的项目中创建一个新的应用命名为<em>actions</em>：</p>
<pre class="hljs python"><code class="python">django-admin startapp actions</code></pre>
<p>在你的项目中的<em>settings.py</em>文件中的<em>INSTALLED_APPS</em>设置中添加<em>'actions'</em>，这样可以让Django知道这个新的应用是可用状态：</p>
<pre class="hljs python"><code class="python">INSTALLED_APPS = (
    <span class="hljs-comment"># ...</span>
    <span class="hljs-string">'actions'</span>,    
)</code></pre>
<p>编辑<em>actions</em>应用下的<em>models.py</em>文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.db <span class="hljs-keyword">import</span> models
<span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Action</span><span class="hljs-params">(models.Model)</span>:</span>
    user = models.ForeignKey(User,
                            related_name=<span class="hljs-string">'actions'</span>,
                            db_index=<span class="hljs-keyword">True</span>)
    verb = models.CharField(max_length=<span class="hljs-number">255</span>)
    created = models.DateTimeField(auto_now_add=<span class="hljs-keyword">True</span>,
                                    db_index=<span class="hljs-keyword">True</span>)

    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        ordering = (<span class="hljs-string">'-created'</span>,)</code></pre>
<p>这个<em>Action</em>模型（model）将会用来记录用户的活动。模型（model）中的字段解释如下：</p>
<ul>
<li>user：执行该操作的用户。这个一个指向Django <em>User</em>模型（model）的 <em>ForeignKey</em>。</li>
<li>verb：这是用户执行操作的动作描述。</li>
<li>created：这个时间日期会在动作执行的时候创建。我们使用<code>auto_now_add=True</code>来动态设置它为当前的时间当这个对象第一次被保存在数据库中。</li>
</ul>
<p>通过这个基础模型（model），我们只能够存储操作例如用户X做了哪些事情。我们需要一个额外的<em>ForeignKey</em>字段为了保存操作会涉及到的一个<em>target</em>（目标）对象，例如用户X给图片Y打上了暑期那或者用户X现在关注了用户Y。就像你之前知道的，一个普通的<em>ForeignKey</em>只能指向一个其他的模型（model）。但是，我们需要一个方法，可以让操作的<em>target</em>（目标）对象是任何一个已经存在的模型（model）的实例。这个场景就由Django内容类型框架来上演。</p>
<h3>使用内容类型框架</h3>
<p>Django包含了一个内容类型框架位于<em>django.contrib.contenttypes</em>。这个应用可以跟踪你的项目中所有的模型（models）以及提供一个通用接口来与你的模型（models）进行交互。</p>
<p>当你使用<em>startproject</em>命令创建一个新的项目的时候这个<em>contenttypes</em>应用就被默认包含在<em>INSTALLED_APPS</em>设置中。它被其他的<em>contrib</em>包使用，例如认证(authentication）框架以及admin应用。</p>
<p><em>contenttypes</em>应用包含一个<em>ContentType</em>模型（model）。这个模型（model）的实例代表了你的应用中真实存在的模型（models），并且新的<em>ContentTYpe</em>实例会动态的创建当新的模型（models）安装在你的项目中。<em>ContentType</em>模型（model）有以下字段：</p>
<ul>
<li>app_label：模型（model）属于的应用名，它会自动从模型（model）<em>Meta</em>选项中的<em>app_label</em>属性获取到。举个例子：我们的<em>Image</em>模型（model）属于<em>images</em>应用</li>
<li>model：模型（model）类的名字</li>
<li>name：模型的可读名，它会自动从模型（model）<em>Meta</em>选项中的<em>verbose_name</em>获取到。</li>
</ul>
<p>让我们看一下我们如何实例化<em>ContentType</em>对象。打开Python终端使用<code>python manage.py shell</code>命令。你可以获取一个指定模型（model）对应的<em>ContentType</em>对象通过执行一个带有<em>app_label</em>和<em>model</em>属性的查询，例如：</p>
<pre class="hljs python"><code class="python"><span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> django.contrib.contenttypes.models <span class="hljs-keyword">import</span> ContentType
<span class="hljs-meta">&gt;&gt;&gt; </span>image_type = ContentType.objects.get(app_label=<span class="hljs-string">'images'</span>,model=<span class="hljs-string">'image'</span>)
<span class="hljs-meta">&gt;&gt;&gt; </span>image_type
&lt;ContentType: image&gt;</code></pre>
<p>你还能反过来获取到模型（model）类从一个<em>ContentType</em>对象中通过调用它的<code>model_class()</code>方法：</p>
<pre class="hljs python"><code class="python"><span class="hljs-meta">&gt;&gt;&gt; </span><span class="hljs-keyword">from</span> images.models <span class="hljs-keyword">import</span> Image
<span class="hljs-meta">&gt;&gt;&gt; </span>ContentType.objects.get_for_model(Image)
&lt;ContentType: image&gt;</code></pre>
<p>以上就是内容类型的一些例子。Django提供了更多的方法来使用他们进行工作。你可以访问 <a href="https://docs.djangoproject.com/en/1.8/ref/contrib/contenttypes/" target="_blank">https://docs.djangoproject.com/en/1.8/ref/contrib/contenttypes/</a> 找到关于内容类型框架的官方文档。</p>
<h3>添加通用的关系给你的模型（models）</h3>
<p>在通用关系中<em>ContentType</em>对象扮演指向模型（model）的角色被关联所使用。你需要3个字段在模型（model）中组织一个通用关系：</p>
<ul>
<li>一个<em>ForeignKey</em>字段<em>ContentType</em>。这个字段会告诉我们给这个关联的模型（model）。</li>
<li>一个字段用来存储被关联对象的primary key。这个字段通常是一个<em>PositiveIntegerField</em>用来匹配Django自动的primary key字段。</li>
<li>一个字段用来定义和管理通用关系通过使用前面的两个字段。内容类型框架提供一个<em>GenericForeignKey</em>字段来完成这个目标。</li>
</ul>
<p>编辑<em>actions</em>应用的<em>models.py</em>文件，添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.db <span class="hljs-keyword">import</span> models
<span class="hljs-keyword">from</span> django.contrib.auth.models <span class="hljs-keyword">import</span> User
<span class="hljs-keyword">from</span> django.contrib.contenttypes.models <span class="hljs-keyword">import</span> ContentType
<span class="hljs-keyword">from</span> django.contrib.contenttypes.fields <span class="hljs-keyword">import</span> GenericForeignKey
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Action</span><span class="hljs-params">(models.Model)</span>:</span>
    user = models.ForeignKey(User,
                             related_name=<span class="hljs-string">'actions'</span>,
                             db_index=<span class="hljs-keyword">True</span>)
    verb = models.CharField(max_length=<span class="hljs-number">255</span>)
    target_ct = models.ForeignKey(ContentType,
                                  blank=<span class="hljs-keyword">True</span>,
                                  null=<span class="hljs-keyword">True</span>,
                                  related_name=<span class="hljs-string">'target_obj'</span>)
    target_id = models.PositiveIntegerField(null=<span class="hljs-keyword">True</span>,
                                            blank=<span class="hljs-keyword">True</span>,
                                            db_index=<span class="hljs-keyword">True</span>)
    target = GenericForeignKey(<span class="hljs-string">'target_ct'</span>, <span class="hljs-string">'target_id'</span>)
    created = models.DateTimeField(auto_now_add=<span class="hljs-keyword">True</span>,
                                   db_index=<span class="hljs-keyword">True</span>)
    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        ordering = (<span class="hljs-string">'-created'</span>,)</code></pre>
<p>我们给<em>Action</em>模型添加了以下字段：</p>
<ul>
<li>target_ct：一个<em>ForeignKey</em>字段指向<em>ContentType</em>模型（model）。</li>
<li>target_id：一个<em>PositiveIntegerField</em>用来存储被关联对象的primary key。</li>
<li>target：一个<em>GenericForeignKey</em>字段指向被关联的对象基于前面两个字段的组合之上。</li>
</ul>
<p>Django没有创建任何字段在数据库中给<em>GenericForeignKey</em>字段。只有<em>target_ct</em>和<em>target_id</em>两个字段被映射到数据库字段。两个字段都有<code>blank=True</code>和<code>null=True</code>属性所以一个<em>target</em>（目标）对象不是必须的当保存<em>Action</em>对象的时候。</p>
<blockquote><p>你可以让你的应用更加灵活通过使用通用关系替代外键当它对拥有一个通用关系有意义。</p></blockquote>
<p>运行以下命令来创建初始迁移为这个应用：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> makemigrations actions</code></pre>
<p>你会看到如下输出：</p>
<pre class="hljs python"><code class="python">    Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'actions'</span>:
        <span class="hljs-number">0001</span>_initial.py:
            - Create model Action</code></pre>
<p>接着，运行下一条命令来同步应用到数据库中：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> migrate</code></pre>
<p>这条命令的输出表明新的迁移已经被应用：</p>
<pre class="hljs clean"><code class="clean">Applying actions<span class="hljs-number">.0001</span>_initial... OK</code></pre>
<p>让我们在管理站点中添加<em>Action</em>模型（model）。编辑<em>actions</em>应用的<em>admin.py</em>文件，添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Action

<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ActionAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display = (<span class="hljs-string">'user'</span>, <span class="hljs-string">'verb'</span>, <span class="hljs-string">'target'</span>, <span class="hljs-string">'created'</span>)
    list_filter = (<span class="hljs-string">'created'</span>,)
    search_fields = (<span class="hljs-string">'verb'</span>,)

admin.site.register(Action, ActionAdmin)</code></pre>
<p>你已经将<em>Action</em>模型（model）注册到了管理站点中。运行命令<code>python manage.py runserver</code>来初始化开发服务器然后在浏览器中打开 <a href="http://127.0.0.1:8000/admin/actions/action/add/" target="_blank">http://127.0.0.1:8000/admin/actions/action/add/</a> 。你会看到如下页面可以创建一个新的<em>Action</em>对象：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-365fd0854708ae28.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-365fd0854708ae28.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-3</div>
</div>
<p>如你所见，只有<em>target_ct</em>和<em>target_id</em>两个字段是映射为真实的数据库字段显示，并且<em>GenericForeignKey</em>字段不在这儿出现。<em>target_ct</em>允许你选择任何一个在你的Django项目中注册的模型（models）。你可以限制内容类型从一个限制的模型（models）集合中选择通过在<em>target-ct</em>字段中使用<em>limit_choices_to</em>属性：<em>limit_choices_to</em>属性允许你限制<em>ForeignKey</em>字段的内容通过给予一个特定值的集合。</p>
<p>在<em>actions</em>应用目录下创建一个新的文件命名为<em>utils.py</em>。我们会定义一个快捷函数，该函数允许我们使用一种简单的方式创建新的<em>Action</em>对象。编辑这个新的文件添加如下代码给它：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.contrib.contenttypes.models <span class="hljs-keyword">import</span> ContentType
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Action
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create_action</span><span class="hljs-params">(user, verb, target=None)</span>:</span>
    action = Action(user=user, verb=verb, target=target)
    action.save()</code></pre>
<p><code>create_action()</code>函数允许我们创建<em>actions</em>，该<em>actions</em>可以包含一个<em>target</em>对象或不包含。我们可以使用这个函数在我们代码的任何地方添加新的<em>actions</em>给活动流（activity stream）。</p>
<h3>在活动流（activity stream）中避免重复的操作</h3>
<p>有时候你的用户可能多次执行同个动作。他们可能在短时间内多次点击 <em>like/unlike</em> 按钮或者多次执行同样的动作。这会导致你停止存储和显示重复的动作。为了避免这种情况我们需要改善<code>create_action()</code>函数来避免大部分的重复动作。</p>
<p>编辑<em>actions</em>应用中的<em>utils.py</em>文件使它看上去如下所示：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">import</span> datetime
<span class="hljs-keyword">from</span> django.utils <span class="hljs-keyword">import</span> timezone
<span class="hljs-keyword">from</span> django.contrib.contenttypes.models <span class="hljs-keyword">import</span> ContentType
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Action

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">create_action</span><span class="hljs-params">(user, verb, target=None)</span>:</span>
    <span class="hljs-comment"># check for any similar action made in the last minute</span>
    now = timezone.now()
    last_minute = now - datetime.timedelta(seconds=<span class="hljs-number">60</span>)
    similar_actions = Action.objects.filter(user_id=user.id,
                                            verb= verb,
                                        timestamp__gte=last_minute)
    <span class="hljs-keyword">if</span> target:
        target_ct = ContentType.objects.get_for_model(target)
        similar_actions = similar_actions.filter(
                                            target_ct=target_ct,
                                            target_id=target.id)
    <span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> similar_actions:
        <span class="hljs-comment"># no existing actions found</span>
        action = Action(user=user, verb=verb, target=target)
        action.save()
        <span class="hljs-keyword">return</span> <span class="hljs-keyword">True</span>
    <span class="hljs-keyword">return</span> <span class="hljs-keyword">False</span></code></pre>
<p>我们通过修改<code>create_action()</code>函数来避免保存重复的动作并且返回一个布尔值来告诉该动作是否保存。下面来解释我们是如何避免重复动作的：</p>
<ul>
<li>首先，我们通过Django提供的<code>timezone.now()</code>方法来获取当前时间。这个方法同<code>datetime.datetime.now()相同，但是返回的是一个*timezone-aware*对象。Django提供一个设置叫做*USE_TZ*用来启用或关闭时区的支持。通过使用*startproject*命令创建的默认*settings.py*包含</code>USE_TZ=True`。</li>
<li>我们使用<em>last_minute</em>变量来保存一分钟前的时间，然后我们取回用户从那以后执行的任意一个相同操作。</li>
<li>我们会创建一个<em>Action</em>对象如果在最后的一分钟内没有存在同样的动作。我们会返回<em>True</em>如果一个<em>Action</em>对象被创建，否则返回<em>False</em>。</li>
</ul>
<h3>添加用户动作给活动流（activity stream）</h3>
<p>是时候添加一些动作给我们的视图(views)来给我的用户构建活动流（activity stream）了。我们将要存储一个动作为以下的每一个实例：</p>
<ul>
<li>一个用户给某张图片打上书签</li>
<li>一个用户喜欢或不喜欢某张图片</li>
<li>一个用户创建一个账户</li>
<li>一个用户关注或不关注某个用户</li>
</ul>
<p>编辑<em>images</em>应用下的<em>views.py</em>文件添加以下导入：</p>
<pre class="hljs capnproto"><code class="capnproto"><span class="hljs-keyword">from</span> actions.utils <span class="hljs-keyword">import</span> create_action</code></pre>
<p>在<em>image_create</em>视图（view）中，在保存图片之后添加<code>create-action()</code>，如下所示：</p>
<pre class="hljs python"><code class="python">new_item.save()
create_action(request.user, <span class="hljs-string">'bookmarked image'</span>, new_item)</code></pre>
<p>在<em>image_like</em>视图（view）中，在添加用户给<em>users_like</em>关系之后添加<code>create_action()</code>，如下所示：</p>
<pre class="hljs python"><code class="python">image.users_like.add(request.user)
create_action(request.user, <span class="hljs-string">'likes'</span>, image)</code></pre>
<p>现在编辑<em>account</em>应用中的<em>view.py</em>文件添加以下导入：</p>
<pre class="hljs capnproto"><code class="capnproto"><span class="hljs-keyword">from</span> actions.utils <span class="hljs-keyword">import</span> create_action</code></pre>
<p>在<em>register</em>视图（view）中，在创建<em>Profile</em>对象之后添加<code>create-action()</code>，如下所示：</p>
<pre class="hljs python"><code class="python">new_user.save()
profile = Profile.objects.create(user=new_user)
create_action(new_user, <span class="hljs-string">'has created an account'</span>)</code></pre>
<p>在<em>user_follow</em>视图（view）中添加<code>create_action()</code>，如下所示：</p>
<pre class="hljs python"><code class="python">Contact.objects.get_or_create(user_from=request.user,user_to=user)
create_action(request.user, <span class="hljs-string">'is following'</span>, user)</code></pre>
<p>就像你所看到的，感谢我们的<em>Action</em>模型（model）和我们的帮助函数，现在保存新的动作给活动流（activity stream）是非常简单的。</p>
<h3>显示活动流（activity stream）</h3>
<p>最后，我们需要一种方法来给每个用户显示活动流（activity stream）。我们将会在用户的dashboard中包含活动流（activity stream）。编辑<em>account</em>应用的<em>views.py</em>文件。导入<em>Action</em>模型然后修改<em>dashboard</em>视图（view）如下所示：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> actions.models <span class="hljs-keyword">import</span> Action

<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">dashboard</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-comment"># Display all actions by default</span>
    actions = Action.objects.exclude(user=request.user)
    following_ids = request.user.following.values_list(<span class="hljs-string">'id'</span>,flat=<span class="hljs-keyword">True</span>)
    <span class="hljs-keyword">if</span> following_ids:
        <span class="hljs-comment"># If user is following others, retrieve only their actions</span>
        actions = actions.filter(user_id__in=following_ids)
    actions = actions[:<span class="hljs-number">10</span>]

    <span class="hljs-keyword">return</span> render(request,
                  <span class="hljs-string">'account/dashboard.html'</span>,
                  {<span class="hljs-string">'section'</span>: <span class="hljs-string">'dashboard'</span>,
                    <span class="hljs-string">'actions'</span>: actions})</code></pre>
<p>在这个视图（view），我们从数据库取回所有的动作（actions），不包含当前用户执行的动作。如果当前用户还没有关注过任何人，我们展示在平台中的其他用户的最新动作执行。这是一个默认的行为当当前用户还没有关注过任何其他的用户。如果当前用户已经关注了其他用户，我们就限制查询只显示当前用户关注的用户的动作执行。最后，我们限制结果只返回最前面的10个动作。我们在这儿并不使用<code>order_by()</code>，因为我们依赖之前已经在<em>Action</em>模型（model）的<em>Meta</em>的排序选项。最新的动作会首先返回，因为我们在<em>Action</em>模型（model）中设置过<code>ordering = ('-created',)</code>。</p>
<h2>优化涉及被关联的对想的查询集（QuerySets）</h2>
<p>每次你取回一个<em>Aciton</em>对象，你都可能存取它的有关联的<em>User</em>对象，<br>并且可能这个用户也关联它的<em>Profile</em>对象。Django ORM提供了一个简单的方式一次性取回有关联的对象，避免对数据库进行额外的查询。</p>
<h3>使用<em>select_related</em>
</h3>
<p>Django提供了一个叫做<code>select_related()</code>的查询集（QuerySets）方法允许你取回关系为一对多的关联对象。该方法将会转化成一个单独的，更加复杂的查询集（QuerySets），但是你可以避免额外的查询当存取这些关联对象。<em>select_relate</em>方法是给<em>ForeignKey</em>和<em>OneToOne</em>字段使用的。它通过执行一个 <em>SQL JOIN</em>并且包含关联对象的字段在<em>SELECT</em> 声明中。</p>
<p>为了利用<code>select_related()</code>,编辑之前代码中的以下行(译者注：请注意双下划线）：</p>
<pre class="hljs ini"><code class="ini"><span class="hljs-attr">actions</span> = actions.filter(user_id__in=following_ids)</code></pre>
<p>添加<em>select_related</em>在你将要使用的字段上：</p>
<pre class="hljs python"><code class="python">actions = actions.filter(user_id__in=following_ids)\
                    .select_related(<span class="hljs-string">'user'</span>, <span class="hljs-string">'user__profile'</span>)</code></pre>
<p>我们使用<code>user__profile</code>（译者注：请注意是双下划线）来连接profile表在一个单独的<em>SQL</em>查询中。如果你调用<code>select_related()</code>而不传入任何参数，它会取回所有<em>ForeignKey</em>关系的对象。给<code>select_related()</code>限制的关系将会在随后一直访问。</p>
<blockquote><p>小心的使用<code>select_related()</code>将会极大的提高执行时间</p></blockquote>
<h3>使用<em>prefetch_related</em>
</h3>
<p>如你所见，<code>select_related()</code>将会帮助你提高取回一对多关系的关联对象的执行效率。但是，<code>select_related()</code>无法给多对多或者多对一关系（ManyToMany或者倒转<em>ForeignKey</em>字段）工作。Django提供了一个不同的查询集（QuerySets）方法叫做<em>prefetch_realted</em>，该方法在<code>select_related()</code>方法支持的关系上增加了多对多和多对一的关系。<code>prefetch_related()</code>方法为每一种关系执行单独的查找然后对各个结果进行连接通过使用Python。这个方法还支持<em>GeneriRelation</em>和<em>GenericForeignKey</em>的预先读取。</p>
<p>完成你的查询通过为它添加<code>prefetch_related()</code>给目标<em>GenericForeignKey</em>字段，如下所示：</p>
<pre class="hljs python"><code class="python">actions = actions.filter(user_id__in=following_ids)\
                 .select_related(<span class="hljs-string">'user'</span>, <span class="hljs-string">'user__profile'</span>)\
                 .prefetch_related(<span class="hljs-string">'target'</span>)</code></pre>
<p>这个查询现在已经被充分利用用来取回包含关联对象的用户动作（actions）。</p>
<h3>为<em>actions</em>创建模板（templates）</h3>
<p>我们要创建一个模板（template）用来显示一个独特的<em>Action</em>对象。在<em>actions</em>应用中创建一个新的目录命名为<em>templates</em>。添加如下文件结构：</p>
<pre class="hljs fortran"><code class="fortran">actions/
    <span class="hljs-keyword">action</span>/
        detail.html</code></pre>
<p>编辑<em>actions/action/detail.html</em>模板（template）文件添加如下代码：</p>
<pre class="hljs undefined"><code class="html">明天添加</code></pre>
<p>这个模板用来显示一个<em>Action</em>对象。首先，我们使用<code><span>{</span><span>%</span> with <span>%</span><span>}</span></code>模板标签（template tag）来获取用户操作的动作（action）和他们的profile。然后，我们显示目标对象的图片如果<em>Action</em>对象有一个关联的目标对象。最后，如果有执行过的动作（action），包括动作和目标对象，我们就显示链接给用户。</p>
<p>现在，编辑<em>account/dashboard.html</em>模板（template）添加如下代码到<em>content</em>区块下方：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span>What's happening<span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"action-list"</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> action <span class="hljs-keyword">in</span> actions <span>%</span><span>}</span></span><span class="xml">
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">include</span></span> "actions/action/detail.html" <span>%</span><span>}</span></span><span class="xml">
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/account/" target="_blank">http://127.0.0.1:8000/account/</a> 。登录一个存在的用户并且该用户执行过一些操作已经被存储在数据库中。然后，登录其他用户，关注之前登录的用户，在dashboard页面可以看到生成的动作流。如下所示：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-b25d0a8feb617640.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-b25d0a8feb617640.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-4</div>
</div>
<p>我们刚刚创建了一个完整的活动流（activity stream）给我们的用户并且我们还能非常容易的添加新的用户动作给它。你还可以添加无限的滚动功能给活动流（activity stream）通过集成AJAX分页处理，和我们之前在<em>image_list</em>视图（view）使用过的一样。</p>
<h3>给非规范化（denormalizing）计数使用信号</h3>
<p>有一些场景，你想要使你的数据非规范化。非规划化使指在一定的程度上制造一些数据冗余用来优化读取的性能。你必须十分小心的使用非规划化并且只有在你真的非常需要它的时候才能使用。你会发现非规划化的最大问题就是保持你的非规范化数据更新是非常困难的。</p>
<p>我们将会看到一个例子关于如何改善（improve）我们的查询通过使用非规范化计数。缺点就是我们不得不保持冗余数据的更新。我们将要从我们的<em>Image</em>模型（model）中使数据非规范化然后使用Django信号来保持数据的更新。</p>
<h3>使用信号进行工作</h3>
<p>Django自带一个信号调度程序允许<em>receiver</em>函数在某个动作出现的时候去获取通知。信号非常有用，当你需要你的代码去执行某些事件的时候同时正在发生其他事件。你还能够创建你自己的信号这样一来其他人可以在某个事件发生的时候获得通知。</p>
<p>Django模型（models）提供了几个信号，它们位于<em>django.db.models.signales</em>。举几个例子：</p>
<ul>
<li>
<em>pre_save</em> 和 <em>post_save</em>：前者会在调用模型（model）的<code>save()</code>方法前发送信号，后者反之。</li>
<li>
<em>pre_delete</em> 和 <em>post_delete</em>：前者会在调用模型（model）或查询集（QuerySets）的<code>delete()</code>方法之前发送信号，后者反之。</li>
<li>
<em>m2m_changed</em>：当在一个模型（model）上的<em>ManayToManayField</em>被改变的时候发送信号。</li>
</ul>
<p>以上只是Django提供的一小部分信号。你可以通过访问 <a href="https://docs.djangoproject.com/en/1.8/ref/signals/" target="_blank">https://docs.djangoproject.com/en/1.8/ref/signals/</a> 获得更多信号资料。</p>
<p>打个比方，你想要获取热门图片。你可以使用Django的聚合函数来获取图片，通过图片获取的用户喜欢数量来进行排序。要记住你已经使用过Django聚合函数在<em>第三章 扩展你的blog应用</em>。以下代码将会获取图片并进行排序通过它们被用户喜欢的数量：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.db.models <span class="hljs-keyword">import</span> Count
<span class="hljs-keyword">from</span> images.models <span class="hljs-keyword">import</span> Image
images_by_popularity = Image.objects.annotate(
    total_likes=Count(<span class="hljs-string">'users_like'</span>)).order_by(<span class="hljs-string">'-total_likes'</span>)</code></pre>
<p>但是，通过统计图片的总喜欢数量进行排序比直接使用一个已经存储总统计数的字段进行排序要消耗更多的性能。你可以添加一个字段给<em>Image</em>模型（model）用来非规范化喜欢的数量用来提升涉及该字段的查询的性能。那么，问题来了，我们该如何保持这个字段是最新更新过的。</p>
<p>编辑<em>images</em>应用下的<em>models.py</em>文件，给<em>Image</em>模型（model）添加以下字段：</p>
<pre class="hljs python"><code class="python">total_likes = models.PositiveIntegerField(db_index=<span class="hljs-keyword">True</span>,
                                          default=<span class="hljs-number">0</span>)</code></pre>
<p><em>total_likes</em>字段允许我们给每张图片存储被用户喜欢的总数。非规范化数据非常有用当你想要使用他们来过滤或排序查询集（QuerySets）。</p>
<blockquote><p>在你使用非规范化字段之前你必须考虑下其他几种提高性能的方法。考虑下数据库索引，最佳化查询以及缓存在开始规范化你的数据之前。</p></blockquote>
<p>运行以下命令将新添加的字段迁移到数据库中：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> makemigrations images</code></pre>
<p>你会看到如下输出：</p>
<pre class="hljs python"><code class="python">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'images'</span>:
    <span class="hljs-number">0002</span>_image_total_likes.py:
        - Add field total_likes to image</code></pre>
<p>接着继续运行以下命令来应用迁移：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> migrate images</code></pre>
<p>输出中会包含以下内容：</p>
<pre class="hljs clean"><code class="clean">Applying images<span class="hljs-number">.0002</span>_image_total_likes... OK</code></pre>
<p>我们要给<em>m2m_changed</em>信号附加一个<em>receiver</em>函数。在<em>images</em>应用目录下创建一个新的文件命名为<em>signals.py</em>。给该文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.db.models.signals <span class="hljs-keyword">import</span> m2m_changed
<span class="hljs-keyword">from</span> django.dispatch <span class="hljs-keyword">import</span> receiver
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Image
<span class="hljs-meta">@receiver(m2m_changed, sender=Image.users_like.through)</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">users_like_changed</span><span class="hljs-params">(sender, instance, **kwargs)</span>:</span>
    instance.total_likes = instance.users_like.count()
    instance.save()</code></pre>
<p>首先，我们使用<code>receiver()</code>装饰器将<code>users_like_changed</code>函数注册成一个<em>receiver</em>函数，然后我们将该函数附加给<em>m2m_changed</em>信号。我们将这个函数与<em>Image.users_like.through</em>连接，这样这个函数只有当<em>m2m_changed</em>信号被<em>Image.users_like.through</em>执行的时候才被调用。还有一个可以替代的方式来注册一个<em>receiver</em>函数，由使用<em>Signal</em>对象的<code>connect()</code>方法组成。</p>
<blockquote><p>Django信号是同步阻塞的。不要使用异步任务导致信号混乱。但是，你可以联合两者来执行异步任务当你的代码只接受一个信号的通知。</p></blockquote>
<p>你必须连接你的<em>receiver</em>函数给一个信号，只有这样它才会被调用当连接的信号发送的时候。有一个推荐的方法用来注册你的信号是在你的应用配置类中导入它们到<code>ready()</code>方法中。Django提供一个应用注册允许你对你的应用进行配置和内省。</p>
<h3>典型的应用配置类</h3>
<p>django允许你指定配置类给你的应用们。为了提供一个自定义的配置给你的应用，创建一个继承<em>django.apps</em>的<em>Appconfig</em>类的自定义类。这个应用配置类允许你为应用存储元数据和配置并且提供<br>内省。</p>
<p>你可以通过访问 <a href="https://docs" target="_blank">https://docs</a>. djangoproject.com/en/1.8/ref/applications/ 获取更多关于应用配置的信息。</p>
<p>为了注册你的信号<em>receiver</em>函数，当你使用<code>receiver()</code>装饰器的时候，你只需要导入信号模块，这些信号模块被包含在你的应用的<em>AppConfig</em>类中的<code>ready()</code>方法中。这个方法在应用注册被完整填充的时候就调用。其他给你应用的初始化都可以被包含在这个方法中。</p>
<p>在<em>images</em>应用目录下创建一个新的文件命名为<em>apps.py</em>。为该文件添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.apps <span class="hljs-keyword">import</span> AppConfig
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ImagesConfig</span><span class="hljs-params">(AppConfig)</span>:</span>
    name = <span class="hljs-string">'images'</span>
    verbose_name = <span class="hljs-string">'Image bookmarks'</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">ready</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-comment"># import signal handlers</span>
        <span class="hljs-keyword">import</span> images.signals</code></pre>
<p><em>name</em>属性定义该应用完整的Python路径。<em>verbose_name</em>属性设置了这个应用可读的名字。它会在管理站点中显示。<code>ready()</code>方法就是我们为这个应用导入信号的地方。</p>
<p>现在我们需要告诉Django我们的应用配置位于哪里。编辑位于<em>images</em>应用目录下的<em><strong>init</strong>.py</em>文件添加如下内容：</p>
<pre class="hljs ini"><code class="ini"><span class="hljs-attr">default_app_config</span> = <span class="hljs-string">'images.apps.ImagesConfig'</span></code></pre>
<p>打开你的浏览器浏览一个图片的详细页面然后点击<strong>like</strong>按钮。再进入管理页面看下该图片的<em>total_like</em>属性。你会看到<em>total_likes</em>属性已经更新了最新的like数如下所示：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-5e1e8cfc6b3cfcbe.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-5e1e8cfc6b3cfcbe.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-5</div>
</div>
<p>现在，你可以使用<em>totla_likes</em>属性来进行热门图片的排序或者在任何地方显示这个值，从而避免了复杂的查询操作。以下获取图片的查询通过图片的喜欢数量进行排序：</p>
<pre class="hljs python"><code class="python">images_by_popularity = Image.objects.annotate(
    likes=Count(<span class="hljs-string">'users_like'</span>)).order_by(<span class="hljs-string">'-likes'</span>)</code></pre>
<p>现在我们可以用新的查询来代替上面的查询：</p>
<pre class="hljs stylus"><code class="stylus">images_by_popularity = Image<span class="hljs-selector-class">.objects</span><span class="hljs-selector-class">.order_by</span>(<span class="hljs-string">'-total_likes'</span>)</code></pre>
<p>以上查询的返回结果只需要很少的SQL查询性能。以上就是一个例子关于如何使用Django信号。</p>
<blockquote><p>小心使用信号，因为它们会给理解控制流制造困难。在很多场景下你可以避免使用信号如果你知道哪个接收器需要被通知。</p></blockquote>
<h3>使用Redis来存储视图（views）项</h3>
<p>Redis是一个高级的<em>key-value</em>（键值）数据库允许你保存不同类型的数据并且在<em>I/O</em>(输入/输出）操作上非常非常的快速。Redis可以在内存中存储任何东西，但是这些数据能够持续通过偶尔存储数据集到磁盘中或者添加每一条命令到日志中。Redis是非常出彩的通过与其他的键值存储对比：它提供了一个强大的设置命令，并且支持多种数据结构，例如string，hashes，lists，sets，ordered sets，甚至bitmaps和HyperLogLogs。</p>
<p>SQL最适合用于模式定义的持续数据存储，而Redis提供了许多优势当需要处理快速变化的数据，易失性存储，或者需要一个快速缓存的时候。让我们看下Redis是如何被使用的，当构建新的功能到我们的项目中。</p>
<h3>安装Redis</h3>
<p>从 <a href="http://redis.io/download" target="_blank">http://redis.io/download</a> 下载最新的Redis版本。解压<em>tar.gz</em>文件，进入<em>redis</em>目录然后编译Redis通过使用以下make命令：</p>
<pre class="hljs vim"><code class="vim"><span class="hljs-keyword">cd</span> redis-<span class="hljs-number">3.0</span>.<span class="hljs-number">4</span>(译者注：版本根据自己下载的修改)
<span class="hljs-keyword">make</span> （译者注：这里是假设你使用的是linux或者mac系统才有<span class="hljs-keyword">make</span>命令，windows如何操作请看下官方文档）</code></pre>
<p>在Redis安装完成后允许以下shell命令来初始化Redis服务：</p>
<pre class="hljs axapta"><code class="axapta">src/redis-<span class="hljs-keyword">server</span></code></pre>
<p>你会看到输出的结尾如下所示：</p>
<pre class="hljs applescript"><code class="applescript"><span class="hljs-comment"># Server started, Redis version 3.0.4</span>
* DB loaded <span class="hljs-keyword">from</span> disk: <span class="hljs-number">0.001</span> seconds
* The server <span class="hljs-keyword">is</span> now ready <span class="hljs-keyword">to</span> accept connections <span class="hljs-keyword">on</span> port <span class="hljs-number">6379</span></code></pre>
<p>默认的，Redis运行会占用6379端口，但是你也可以指定一个自定义的端口通过使用<code>--port</code>标志，例如：<code>redis-server --port 6655</code>。当你的服务启动完毕，你可以在其他的终端中打开Redis客户端通过使用如下命令：</p>
<pre class="hljs avrasm"><code class="avrasm">src/redis-<span class="hljs-keyword">cli</span></code></pre>
<p>你会看到Redis客户端shell如下所示：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt;</code></pre>
<p>Redis客户端允许你在当前shell中立即执行Rdis命令。来我们来尝试一些命令。键入<em>SET</em>命令在Redis客户端中存储一个值到一个键中：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">SET</span> <span class="hljs-selector-tag">name</span> "<span class="hljs-selector-tag">Peter</span>"
<span class="hljs-selector-tag">ok</span></code></pre>
<p>以上的命令创建了一个带有字符串“Peter”值的<em>name</em>键到Redis数据库中。<em>OK</em>输出表明该键已经被成功保存。然后，使用<em>GET</em>命令获取之前的值，如下所示：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">GET</span> <span class="hljs-selector-tag">name</span>
"<span class="hljs-selector-tag">Peter</span>"</code></pre>
<p>你还可以检查一个键是否存在通过使用<em>EXISTS</em>命令。如果检查的键存在会返回1，反之返回0：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">EXISTS</span> <span class="hljs-selector-tag">name</span>
(<span class="hljs-selector-tag">integer</span>) 1</code></pre>
<p>你可以给一个键设置到期时间通过使用<em>EXPIRE</em>命令，该命令允许你设置该键能在几秒内存在。另一个选项使用<em>EXPIREAT</em>命令来期望一个Unix时间戳。键的到期消失是非常有用的当将Redis当做缓存使用或者存储易失性的数据：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">GET</span> <span class="hljs-selector-tag">name</span>
"<span class="hljs-selector-tag">Peter</span>"
127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">EXPIRE</span> <span class="hljs-selector-tag">name</span> 2
(<span class="hljs-selector-tag">integer</span>) 1
<span class="hljs-selector-tag">Wait</span> <span class="hljs-selector-tag">for</span> 2 <span class="hljs-selector-tag">seconds</span> <span class="hljs-selector-tag">and</span> <span class="hljs-selector-tag">try</span> <span class="hljs-selector-tag">to</span> <span class="hljs-selector-tag">get</span> <span class="hljs-selector-tag">the</span> <span class="hljs-selector-tag">same</span> <span class="hljs-selector-tag">key</span> <span class="hljs-selector-tag">again</span>:
127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">GET</span> <span class="hljs-selector-tag">name</span>
(<span class="hljs-selector-tag">nil</span>)</code></pre>
<p>（nil）响应是一个空的响应说明没有找到键。你还可以通过使用<em>DEL</em>命令删除任意键，如下所示：</p>
<pre class="hljs css"><code class="css">127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">SET</span> <span class="hljs-selector-tag">total</span> 1
<span class="hljs-selector-tag">OK</span>
127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">DEL</span> <span class="hljs-selector-tag">total</span>
(<span class="hljs-selector-tag">integer</span>) 1
127<span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.0</span><span class="hljs-selector-class">.1</span><span class="hljs-selector-pseudo">:6379</span>&gt; <span class="hljs-selector-tag">GET</span> <span class="hljs-selector-tag">total</span>
(<span class="hljs-selector-tag">nil</span>)</code></pre>
<p>以上只是一些键选项的基本命令。Redis包含了庞大的命令设置给一些数据类型，例如strings，hashes，sets，ordered sets等等。你可以通过访问 <a href="http://redis.io/commands" target="_blank">http://redis.io/commands</a> 看到所有Reids命令以及通过访问 <a href="http://redis.io/topics/data-types" target="_blank">http://redis.io/topics/data-types</a> 看到所有Redis支持的数据类型。</p>
<h3>通过Python使用Redis</h3>
<p>我们需要绑定Python和Redis。通过pip渠道安装<em>redis-py</em>命令如下：</p>
<pre class="hljs lsl"><code class="lsl">pip install redis==<span class="hljs-number">2.10</span><span class="hljs-number">.3</span>（译者注：版本可能有更新，如果需要最新版本，可以不带上'==<span class="hljs-number">2.10</span><span class="hljs-number">.3</span>'后缀）</code></pre>
<p>你可以访问 <a href="http://redis-py.readthedocs.org/" target="_blank">http://redis-py.readthedocs.org/</a> 得到redis-py文档。</p>
<p><em>redis-py</em>提供两个类用来与Redis交互：<em>StrictRedis</em>和<em>Redis</em>。两者提供了相同的功能。<em>StrictRedis</em>类尝试遵守官方的Redis命令语法。<em>Redis</em>类型继承<em>Strictredis</em>重写了部分方法来提供向后的兼容性。我们将会使用<em>StrictRedis</em>类，因为它遵守Redis命令语法。打开Python shell执行以下命令：</p>
<pre class="hljs ruby"><code class="ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; import redis
<span class="hljs-meta">&gt;&gt;</span>&gt; r = redis.StrictRedis(host=<span class="hljs-string">'localhost'</span>, port=<span class="hljs-number">6379</span>, db=<span class="hljs-number">0</span>)</code></pre>
<p>上面的代码创建了一个与Redis数据库的连接。在Redis中，数据库通过一个整形索引替代数据库名字来辨识。默认的，一个客户端被连接到数据库 0 。Reids数据库可用的数字设置到16，但是你可以在<em>redis.conf</em>文件中修改这个值。</p>
<p>现在使用Python shell设置一个键：</p>
<pre class="hljs ruby"><code class="ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; r.set(<span class="hljs-string">'foo'</span>, <span class="hljs-string">'bar'</span>)
True</code></pre>
<p>以上命令返回Ture表明这个键已经创建成功。现在你可以使用<code>get()</code>命令取回该键：</p>
<pre class="hljs ruby"><code class="ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; r.get(<span class="hljs-string">'foo'</span>)
<span class="hljs-string">'bar'</span></code></pre>
<p>如你所见，<em>StrictRedis</em>方法遵守Redis命令语法。</p>
<p>让我们集成Rdies到我们的项目中。编辑<em>bookmarks</em>项目的<em>settings.py</em>文件添加如下设置：</p>
<pre class="hljs python"><code class="python">REDIS_HOST = <span class="hljs-string">'localhost'</span>
REDIS_PORT = <span class="hljs-number">6379</span>
REDIS_DB = <span class="hljs-number">0</span></code></pre>
<p>以上设置了Redis服务器和我们将要在项目中使用到的数据库。</p>
<h3>存储视图（vies）项到Redis中</h3>
<p>让我们存储一张图片被查看的总次数。如果我们通过Django ORM来完成这个操作，它会在每次该图片显示的时候执行一次SQL UPDATE声明。使用Redis，我们只需要对一个计数器进行增量存储在内存中，从而带来更好的性能。</p>
<p>编辑<em>images</em>应用下的<em>views.py</em>文件，添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">import</span> redis
<span class="hljs-keyword">from</span> django.conf <span class="hljs-keyword">import</span> settings
<span class="hljs-comment"># connect to redis</span>
r = redis.StrictRedis(host=settings.REDIS_HOST,
                      port=settings.REDIS_PORT,
                      db=settings.REDIS_DB)</code></pre>
<p>在这儿我们建立了Redis的连接为了能在我们的视图（views)中使用它。编辑<em>images_detail</em>视图（view）使它看上去如下所示：</p>
<pre class="hljs python"><code class="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_detail</span><span class="hljs-params">(request, id, slug)</span>:</span>
image = get_object_or_404(Image, id=id, slug=slug)
<span class="hljs-comment"># increment total image views by 1</span>
total_views = r.incr(<span class="hljs-string">'image:{}:views'</span>.format(image.id)) 
<span class="hljs-keyword">return</span> render(request,
              <span class="hljs-string">'images/image/detail.html'</span>,
              {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>,
               <span class="hljs-string">'image'</span>: image,
               <span class="hljs-string">'total_views'</span>: total_views})</code></pre>
<p>在这个视图（view）中，我们使用<em>INCR</em>命令，它会从1开始增量一个键的值，在执行这个操作之前如果键不存在，它会将值设定为0.<code>incr()</code>方法在执行操作后会返回键的值，然后我们可以存储该值到<em>total_views</em>变量中。我们构建Rddis键使用一个符号，比如 <em>object-type:id:field (for example image:33:id)</em> 。</p>
<blockquote><p>对Redis的键进行命名有一个惯例是使用冒号进行分割来创建键的命名空间。做到这点，键的名字会特别冗长，有关联的键会分享部分相同的模式在它们的名字中。</p></blockquote>
<p>编辑<em>image/detail.html</em>模板（template）在已有的<code>&lt;span class="count"&gt;</code>元素之后添加如下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"count"</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"total"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> total_views <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
     view</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_views|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span></code></pre>
<p>现在在浏览器中打开一张图片的详细页面然后多次加载该页面。你会看到每次该视图（view）被执行的时候，总的观看次数会增加 1 。如下所示：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-7faeef32eacf1b01.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-7faeef32eacf1b01.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-6</div>
</div>
<p>你已经成功的集成Redis到你的项目中来存储项统计。</p>
<h3>存储一个排名到Reids中</h3>
<p>让我们使用Reids构建更多的功能。我们要在我们的平台中创建一个最多浏览次数的图片排行。为了构建这个排行我们将要使用Redis分类集合。一个分类集合是一个非重复的字符串采集，其中每个成员和一个分数关联。其中的项根据它们的分数进行排序。</p>
<p>编辑<em>images</em>引用下的<em>views.py</em>文件，使<em>image_detail</em>视图（view）看上去如下所示：</p>
<pre class="hljs python"><code class="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_detail</span><span class="hljs-params">(request, id, slug)</span>:</span>
image = get_object_or_404(Image, id=id, slug=slug)
<span class="hljs-comment"># increment total image views by 1</span>
total_views = r.incr(<span class="hljs-string">'image:{}:views'</span>.format(image.id)) <span class="hljs-comment"># increment image ranking by 1 </span>
r.zincrby(<span class="hljs-string">'image_ranking'</span>, image.id, <span class="hljs-number">1</span>)
<span class="hljs-keyword">return</span> render(request,
              <span class="hljs-string">'images/image/detail.html'</span>,
              {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>,
               <span class="hljs-string">'image'</span>: image,
               <span class="hljs-string">'total_views'</span>: total_views})</code></pre>
<p>我们使用<code>zincrby()</code>命令存储图片视图（views）到一个分类集合中通过键<code>image:ranking</code>。我们存储图片<em>id</em>，和一个分数1，它们将会被加到分类集合中这个元素的总分上。这将允许我们在全局上持续跟踪所有的图片视图（views），并且有一个分类集合，该分类集合通过图片的浏览次数进行排序。</p>
<p>现在创建一个新的视图（view）用来展示最多浏览次数图片的排行。在<em>views.py</em>文件中添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_ranking</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-comment"># get image ranking dictionary</span>
    image_ranking = r.zrange(<span class="hljs-string">'image_ranking'</span>, <span class="hljs-number">0</span>, <span class="hljs-number">-1</span>,
                             desc=<span class="hljs-keyword">True</span>)[:<span class="hljs-number">10</span>]
    image_ranking_ids = [int(id) <span class="hljs-keyword">for</span> id <span class="hljs-keyword">in</span> image_ranking]
    <span class="hljs-comment"># get most viewed images</span>
    most_viewed = list(Image.objects.filter(
                       id__in=image_ranking_ids))
    most_viewed.sort(key=<span class="hljs-keyword">lambda</span> x: image_ranking_ids.index(x.id))
    <span class="hljs-keyword">return</span> render(request,
                  <span class="hljs-string">'images/image/ranking.html'</span>,
                  {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>,
                   <span class="hljs-string">'most_viewed'</span>: most_viewed})</code></pre>
<p>以上就是<em>image_ranking</em>视图。我们使用<code>zrange()</code>命令获得分类集合中的元素。这个命令期望一个自定义的范围，最低分和最高分。通过将 0 定为最低分， -1 为最高分，我们告诉Redis返回分类集合中的所有元素。最终，我们使用<code>[:10]</code>对结果进行切片获取最前面十个最高分的元素。我们构建一个返回的图片IDs的列，然后我们将该列存储在<em>image_ranking_ids</em>变量中，这是一个整数列。我们通过这些IDs取回对应的<em>Image</em>对象，并将它们强制转化为列通过使用<code>list()</code>函数。强制转化查询集（QuerySets）的执行是非常重要的，因为接下来我们要在该列上使用列的<code>sort()</code>方法（就是因为这点所以我们需要的是一个对象列而不是一个查询集（QuerySets））。我们排序这些<em>Image</em>对象通过它们在图片排行中的索引。现在我们可以在我们的模板（template）中使用<em>most_viewed</em>列来显示10个最多浏览次数的图片。</p>
<p>创建一个新的<em>image/ranking.html</em>模板（template）文件，添加如下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Images ranking</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Images ranking<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
     <span class="hljs-tag">&lt;<span class="hljs-name">ol</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> image <span class="hljs-keyword">in</span> most_viewed <span>%</span><span>}</span></span><span class="xml">
         <span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span>
           <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
             </span><span class="hljs-template-variable"><span>{</span><span>{</span> image.title <span>}</span><span>}</span></span><span class="xml">
           <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span> 
         <span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span>
       </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
     <span class="hljs-tag">&lt;/<span class="hljs-name">ol</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板（template）非常简单明了，我们只是对包含在<em>most_viewed</em>中的<em>Image</em>对象进行迭代。</p>
<p>最后为新的视图（view）创建一个URL模式。编辑<em>images</em>应用下的<em>urls.py</em>文件，添加如下内容：</p>
<pre class="hljs awk"><code class="awk">url(<span class="hljs-string">r'^ranking/$'</span>, views.image_ranking, name=<span class="hljs-string">'create'</span>),</code></pre>
<p>在浏览器中打开 <a href="http://127.0.0.1:8000/images/ranking/" target="_blank">http://127.0.0.1:8000/images/ranking/</a> 。你会看到如下图片排行：</p>
<div class="image-package">
<img src="http://upload-images.jianshu.io/upload_images/3966530-e3e6a0ac2b862a51.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240" data-original-src="http://upload-images.jianshu.io/upload_images/3966530-e3e6a0ac2b862a51.png?imageMogr2/auto-orient/strip%7CimageView2/2" style="cursor: zoom-in;"><br><div class="image-caption">django-6-7</div>
</div>
<h3>Redis的下一步</h3>
<p>Redis并不能替代你的SQL数据库，但是它是一个内存中的快速存储，更适合某些特定任务。将它添加到你的栈中使用当你真的感觉它很需要。以下是一些适合Redis的场景：</p>
<ul>
<li>Counting：如你之前看到的，通过Redis管理计数器非常容易。你可以使用<code>incr()</code>和`incrby()。</li>
<li>Storing latest items：你可以添加项到一个列的开头和结尾通过使用<code>lpush()</code>和<code>rpush()</code>。移除和返回开头和结尾的元素通过使用<code>lpop()</code>以及<code>rpop()</code>。你可以削减列的长度通过使用<code>ltrim()</code>来维持它的长度。</li>
<li>Queues：除了push和pop命令，Redis还提供堵塞的队列命令。</li>
<li>Caching：使用<code>expire()</code>和<code>expireat()</code>允许你将Redis当成缓存使用。你还可以找到第三方的Reids缓存后台给Django使用。</li>
<li>Pub/Sub：Redis提供命令给订阅或不订阅，并且给渠道发送消息。</li>
<li>Rankings and leaderboards：Redis使用分数的分类集合使创建排行榜非常的简单。</li>
<li>Real-time tracking：Redis快速的I/O(输入/输出)使它能完美支持实时场景。</li>
</ul>
<h3>总结</h3>
<p>在本章中，你构建了一个粉丝系统和一个用户活动流（activity stream）。你学习了Django信号是如何进行工作并且在你的项目中集成了Redis。</p>
<p>在下一章中，你会学习到如何构建一个在线商店。你会创建一个产品目录并且通过会话（sessions）创建一个购物车。你还会学习如何通过Celery执行异步任务。</p>
        </div>