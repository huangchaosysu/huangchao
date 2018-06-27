---
layout: single
permalink: /django/example5/
title: "Django By Example 第五章"
sidebar:
  nav: "django"
# toc: true
---

<div>
<h2><strong>第五章</strong></h2>
<h2><strong>在你的网站中分享内容</strong></h2>
<p>在上一章中，你为你的网站建立了用户注册和认证系统。你学习了如何为用户创建定制化的个人资料模型以及如何将主流的社交网络的认证添加进你的网站。<br>在这一章中，你将学习如何通过创建一个  JavaScript 书签来从其他的站点分享内容到你的网站，你也将通过使用 jQuery 在你的项目中实现一些 AJAX 特性。<br>这一章涵盖了以下几点：</p>
<ul>
<li>创建一个<code>many-to-many</code>（多对多）关系</li>
<li>定制表单（form）的行为</li>
<li>在 Django 中使用 jQuery</li>
<li>创建一个 jQuery 书签</li>
<li>通过使用 sorl-thumbnail 来生成缩略图</li>
<li>实现 AJAX 视图（views）并且使这些视图（views）和 jQuery 融合</li>
<li>为视图（views）创建定制化的装饰器 （decorators）</li>
<li>
<p>创建 AJAX 分页</p>
<h2><strong>建立一个能为图片打标签的网站</strong></h2>
<p>我们将允许用户可以在我们网站中分享他们在其他网站发现的图片，并且他们还可以为这些图片打上标签。为了达到这个目的，我们将要做以下几个任务：</p>
</li>
<li>
<p>定义一个模型来储存图片以及图片的信息</p>
</li>
<li>新建一个表单（form）和视图（view）来控制图片的上传</li>
<li>为用户创建一个可以上传他们在其他网站发现的图片的系统</li>
</ul>
<p>首先，通过以下命令在你的 bookmarks 项目中新建一个应用：</p>
<pre class="hljs ebnf"><code class="ebnf"><span class="hljs-attribute">django-admin startapp images</span></code></pre>
<p>像如下所示一样在你的 <code>settings.py</code> 文件中 <code>INSTALED_APPS</code> 设置项下添加 'images' :</p>
<pre class="hljs clean"><code class="clean">INSTALLED_APPS = [
    # ... 
    <span class="hljs-string">'images'</span>,
]</code></pre>
<p>现在Django知道我们的新应用已经被激活了。</p>
<h2><strong>创建图像模型</strong></h2>
<p>编辑 images 应用中的 <code>models.py</code>  文件，将以下代码添加进去：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.db <span class="hljs-keyword">import</span> models
<span class="hljs-keyword">from</span> django.conf <span class="hljs-keyword">import</span> settings
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Image</span><span class="hljs-params">(models.Model)</span>:</span>
    user = models.ForeignKey(settings.AUTH_USER_MODEL,
    related_name=<span class="hljs-string">'images_created'</span>)
    title = models.CharField(max_length=<span class="hljs-number">200</span>)
    slug = models.SlugField(max_length=<span class="hljs-number">200</span>,blank=<span class="hljs-keyword">True</span>)
    url = models.URLField()
    image = models.ImageField(upload_to=<span class="hljs-string">'images/%Y/%m/%d'</span>)
    description = models.TextField(blank=<span class="hljs-keyword">True</span>)
    created = models.DateField(auto_now_add=<span class="hljs-keyword">True</span>,
                               db_index=<span class="hljs-keyword">True</span>)
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">__str__</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> self.title</code></pre>
<p>我们将要使用这个模型来储存来自各个不同网站中被标记的图片。让我们来看看在这个模型中的字段：</p>
<ul>
<li>
<code>user</code>： 标记了这张图片 <code>User</code> 对象。这是一个 <code>ForeignKey</code>字段 (译者注：外键，即一对多字段)，因为它指定了一个一对多关系： 一个用户可以 post 多张图片， 但是每张图片只能由一个用户上传</li>
<li>
<code>title</code>： 图片的标题</li>
<li>
<code>slug</code>： 一个只包含字母、数字、下划线、和连字符的标签， 用于创建优美的 搜索引擎友好（SEO-friendly）的 URL（译者注：slug 这个词在中文没有很好的对应翻译，所以就请大家记住“slug 表示的是只有字母、数字、下划线和连字符的标签”。如果有仔细看过 Django 官方文档的读者就会知道： slug 是一个新闻术语， 而 Django 的开发目的也是为了更好的编辑新闻， 所以这里就不难理解为什么 Django 中会出现 slug 字段了）</li>
<li>
<code>url</code>： 这张图片的源 URL</li>
<li>
<code>image</code>： 图片文件</li>
<li>
<code>description</code>： 一个可选的图片描述字段</li>
<li>
<code>created</code>： 用于表明一个对象在数据库中创建时的时间和日期。由于我们使用了<code>auto_now_add</code> ,当对象被创建时候时间和日期将会被自动设置，我们使用了 <code>db_index=True</code> ，所以 Django 将会在数据库中为这个字段创建索引</li>
</ul>
<blockquote><p>数据库索引改善了查询的执行。考虑为这个字段设置 <code>db_index=True</code> 是因为你将要很频繁地使用 <code>filter()</code>，<code>exclude()</code>,<code>order_by()</code> 来执行查询。<code>ForeignKey</code> 字段或者带有<code>unique=True</code>的字段表明了一个索引的创建。你也可以使用<code>Meta.index_together</code>来为多个字段创建索引。</p></blockquote>
<p>我们将要重写 Image 模型的 <code>save()</code>方法来自动的生成<code>slug</code>字段。这个 <code>slug</code>字段基于<code>title</code>字段的值。像下面这样导入<code>slugify()</code>函数， 然后在 Image 模型中添加一个 <code>save()</code> 方法：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.utils.text <span class="hljs-keyword">import</span> slugify
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Image</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="hljs-comment"># ...</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">save</span><span class="hljs-params">(self, *args, **kwargs)</span>:</span>
        <span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> self.slug:
            self.slug = slugify(self.title)
            super(Image, self).save(*args, **kwargs)</code></pre>
<p>在这段代码中，我们使用了 Django 提供的<code>slugify()</code>函数在没有提供<code>slug</code>字段时根据给定的图片标题自动生slug,然后，我们保存了这个对象。我们自动生成slug，这样的话用户就不用自己输入<code>slug</code>字段了。</p>
<h2><strong>建立多对多关系</strong></h2>
<p>我们将要在 Image 模型中再添加一个字段来保存喜欢这张图片的用户。因此，我们需要一个多对多关系。因为一个用户可能喜欢很多张图片，一张图片也可能被很多用户喜欢。<br>在 Image 模型中添加以下字段：</p>
<pre class="hljs python"><code class="python">user_like = models.ManyToManyField(settings.AUTH_USER_MODEL,
                                   related_name=<span class="hljs-string">'images_liked'</span>,
                                   blank=<span class="hljs-keyword">True</span>)</code></pre>
<p>当你定义一个<code>ManyToMany</code>字段时，Django 会用两张表主键（primary key）创建一个中介联接表（译者注：就是新建一张普通的表，只是这张表的内容是由多对多关系双方的主键构成的）。<code>ManyToMany</code>字段可以在任意两个相关联的表中创建。<br>同<code>ForeignKey</code>字段一样，<code>ManyToMany</code>字段的<code>related_name</code>属性使我们可以命名另模型回溯（或者是反查）到本模型对象的关系。<code>ManyToMany</code>字段提供了一个多对多管理器（manager），这个管理器使我们可以回溯相关联的对象比如：<code>image.users_like.all()</code>或者从一个<code>user</code>中回溯，比如：<code>user.images_liked.all()</code>。<br>打开命令行，执行下面的命令以创建首次迁移：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> makemigrations images</code></pre>
<p>你能看见以下输出：</p>
<pre class="hljs stylus"><code class="stylus">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'images'</span>:
    <span class="hljs-number">0001</span>_initial<span class="hljs-selector-class">.py</span>:
        - Create model Image</code></pre>
<p>现在执行这条命令来应用你的迁移：</p>
<pre class="hljs stylus"><code class="stylus">python manage<span class="hljs-selector-class">.py</span> migrate images</code></pre>
<p>你将会看到包含这一行输出：</p>
<pre class="hljs clean"><code class="clean">Applying images<span class="hljs-number">.0001</span>_initial... OK</code></pre>
<p>现在 Image 模型已经在数据库中同步了。</p>
<h2><strong>注册 Image 模型到管理站点中</strong></h2>
<p>编辑 <code>images</code> 应用的 <code>admin.py</code> 文件，然后像下面这样将 <code>Image</code> 模型注册到管理站点中：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> admin
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Image
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ImageAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display = [<span class="hljs-string">'title'</span>, <span class="hljs-string">'slug'</span>, <span class="hljs-string">'image'</span>, <span class="hljs-string">'created'</span>]
    list_filter = [<span class="hljs-string">'created'</span>]

admin.site.register(Image, ImageAdmin)</code></pre>
<p>使用命令<code>python manage.py runserver</code>打开开发服务器，在浏览器中打开<code>http://127.0.0.1:8000/admin/</code>,可以看到<code>Image</code>模型已经注册到了管理站点中：</p>
<div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-1.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-1.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-1</div>
</div>
<h2><strong>从其他网站上传内容</strong></h2>
<p>我们将使用户可以给从他们在其他网站发现的图片打上标签。用户将要提供图片的 URL ，标题，和一个可选的描述。我们的应用将要下载这幅图片，并且在数据库中创建一个新的 <code>Image</code> 对象。<br>我们从新建一个用于提交图片的表单开始。在images应用的路径下创建一个 <code>forms.py</code> 文件，在这个文件中添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django <span class="hljs-keyword">import</span> forms
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Image
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">ImageCreateForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    <span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Meta</span>:</span>
        model = Image
        fields = (<span class="hljs-string">'title'</span>, <span class="hljs-string">'url'</span>, <span class="hljs-string">'description'</span>)
        widgets = {
            <span class="hljs-string">'url'</span>: forms.HiddenInput,
        }</code></pre>
<p>如你所见，这是一个通过<code>Image</code>模型创建的<code>ModelForm</code>（模型表单），但是这个表单只包含了 <code>title</code>,<code>url</code>,<code>description</code>字段。我们的用户不会在表单中直接为图片添加 URL。相反的，他们将会使用一个 JavaScropt 工具来从其他网站中选择一张图片然后我们的表单将会以参数的形式接收这张图片的 URL。我们覆写 <code>url</code> 字段的默认控件（widget）为一个<code>HiddenInput</code>控件，这个控件将会被渲染为属性是 <code>type="hidden"</code>的 HTML 元素。使用这个控件是因为我们不想让用户看见这个字段。</p>
<h2><strong>清洁表单字段</strong></h2>
<p>（译者注：原文标题是：cleaning form fields,在数据处理中有个术语是“清洗数据”，但是这里的清洁还有“使其整洁”的含义，感觉更加符合<code>clean_url</code>这个方法的定位。）<br>为了验证提供的图片 URL 是否合法，我们将检查以<code>.jpg</code>或<code>.jpeg</code>结尾的文件名，来只允许JPG文件的上传。Django允许你自定义表单方法来清洁特定的字段，通过使用以<code>clean_&lt;fieldname&gt;</code>形式命名的方法来实现。这个方法会在你为一个表单实例执行<code>is_valid()</code>时执行。在清洁方法中，你可以改变字段的值或者为某个特定的字段抛出错误当需要的时候，将下面这个方法添加进<code>ImageCreateForm</code>:</p>
<pre class="hljs python"><code class="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">clean_url</span><span class="hljs-params">(self)</span>:</span>
    url = self.cleaned_data[<span class="hljs-string">'url'</span>]
    valid_extensions = [<span class="hljs-string">'jpg'</span>, <span class="hljs-string">'jpeg'</span>]
    extension = url.rsplit(<span class="hljs-string">'.'</span>, <span class="hljs-number">1</span>)[<span class="hljs-number">1</span>].lower()
    <span class="hljs-keyword">if</span> extension <span class="hljs-keyword">not</span> <span class="hljs-keyword">in</span> valid_extensions:
        <span class="hljs-keyword">raise</span> forms.ValidationError(<span class="hljs-string">'The given URL does not '</span> \
                                   <span class="hljs-string">'match valid image extensions.'</span>)
    <span class="hljs-keyword">return</span> url</code></pre>
<p>在这段代码中，我们定义了一个<code>clean_url</code>方法来清洁<code>url</code>字段，这段代码的工作流程是：</p>
<ul>
<li>
<ol>
<li>我们从表单实例的<code>cleaned_data</code>字典中获取了<code>url</code>字段的值</li>
</ol>
</li>
<li>
<ol>
<li>我们分离了 URL 来获取文件扩展名，然后检查它是否为合法扩展名之一。如果它不是一个合法的扩展名，我们就会抛出<code>ValidationError</code>,并且表单也不会被认证。我们执行的是一个非常简单的认证。你可以使用更好的方法来验证所给的 URL 是否是一个合法的图片。<br>除了验证所给的 URL， 我们还需要下载并保存图片文件。比如，我们可以使用操作表单的视图来下载图片。不过，我们将采用一个更加通用的方法 ———— 通过覆写我们模型表单中<code>save()</code>方法来完成这个任务。</li>
</ol>
</li>
</ul>
<h2><strong>覆写模型表单中的save()方法</strong></h2>
<p>如你所知，<code>ModelForm</code>提供了一个<code>save()</code>方法来保存目前的模型实例到数据库中，并且返回一个对象。这个方法接受一个布尔参数<code>commit</code>，这个参数允许你指定这个对象是否要被储存到数据库中。如果<code>commit</code>是<code>False</code>，<code>save()</code>方法将会返回一个模型实例但是并不会把这个对象保存到数据库中。我们将覆写表单中的<code>save()</code>方法，来下载图片然后保存它。<br>将以下的包在<code>foroms.py</code>中的顶部导入：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> urllib <span class="hljs-keyword">import</span> request
<span class="hljs-keyword">from</span> django.core.files.base <span class="hljs-keyword">import</span> ContentFile
<span class="hljs-keyword">from</span> django.utils.text <span class="hljs-keyword">import</span> slugify</code></pre>
<p>把<code>save()</code>方法加入<code>ImageCreateForm</code>中：</p>
<pre class="hljs python"><code class="python"><span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">save</span><span class="hljs-params">(self, force_insert=False,
         force_update=False,
         commit=True)</span>:</span>
    image = super(ImageCreateForm, self).save(commit=<span class="hljs-keyword">False</span>)
    image_url = self.cleaned_data[<span class="hljs-string">'url'</span>]
    image_name = <span class="hljs-string">'{}.{}'</span>.format(slugify(image.title),
    image_url.rsplit(<span class="hljs-string">'.'</span>, <span class="hljs-number">1</span>)[<span class="hljs-number">1</span>].lower())
<span class="hljs-comment"># 从给定的 URL 中下载图片</span>
    response = request.urlopen(image_url)
    image.image.save(image_name,
                    ContentFile(response.read()),
                    save=<span class="hljs-keyword">False</span>)
    <span class="hljs-keyword">if</span> commit:
        image.save()
    <span class="hljs-keyword">return</span> image</code></pre>
<p>我们覆写的<code>save()</code>方法保持了<code>ModelForm</code>中需要的参数、<br>这段代码：</p>
<ol>
<li>我们通过调用<code>save()</code>方法从表单中新建了一个<code>image</code>对象，并且<code>commit=False</code>
</li>
<li>我们从表单的<code>cleaned_data</code>字典中获取了 URL </li>
<li>我们通过结合<code>image</code>的标题 slug 和源文件的扩展名生成了图片的名字</li>
<li>我们使用 Python 的 <code>urllib</code> 模块来下载图片，然后我们调用<code>save()</code>方法把图片传递给一个<code>ContentFiel</code>对象，这个对象被下载的文件所实例化。这样，我们就可以将我们的文件保存到项目中的 media 路径下。我们传递了参数<code>comiit=False</code>来避免对象被保存到数据库中。</li>
<li>为了保持和我们覆写的<code>save()</code>方法一样的行为，我们将在<code>commit</code>参数为<code>Ture</code>时保存表单到数据库中。</li>
</ol>
<p>现在我们需要一个新的视图来控制我们的表单。编辑 iamges 应用的<code>views.py</code>文件，然后将以下代码添加进去:</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.shortcuts <span class="hljs-keyword">import</span> render, redirect
<span class="hljs-keyword">from</span> django.contrib.auth.decorators <span class="hljs-keyword">import</span> login_required
<span class="hljs-keyword">from</span> django.contrib <span class="hljs-keyword">import</span> messages
<span class="hljs-keyword">from</span> .forms <span class="hljs-keyword">import</span> ImageCreateForm

<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_create</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-string">"""
    View for creating an Image using the JavaScript Bookmarklet.
    """</span>
    <span class="hljs-keyword">if</span> request.method == <span class="hljs-string">'POST'</span>:
        <span class="hljs-comment"># form is sent</span>
        form = ImageCreateForm(data=request.POST)
        <span class="hljs-keyword">if</span> form.is_valid():
            <span class="hljs-comment"># form data is valid</span>
            cd = form.cleaned_data
            new_item = form.save(commit=<span class="hljs-keyword">False</span>)
            <span class="hljs-comment"># assign current user to the item</span>
            new_item.user = request.user
            new_item.save()
            messages.success(request, <span class="hljs-string">'Image added successfully'</span>)
            <span class="hljs-comment"># redirect to new created item detail view</span>
            <span class="hljs-keyword">return</span> redirect(new_item.get_absolute_url())
    <span class="hljs-keyword">else</span>:
        <span class="hljs-comment"># build form with data provided by the bookmarklet via GET</span>
        form = ImageCreateForm(data=request.GET)

    <span class="hljs-keyword">return</span> render(request, <span class="hljs-string">'images/image/create.html'</span>, {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>,
                                                        <span class="hljs-string">'form'</span>: form})</code></pre>
<p>我们给<code>image_create</code>视图添加了一个<code>login_required</code>装饰器，来阻止未认证的用户的连接。这段代码完成下面的工作：</p>
<ol>
<li>我们先从 GET 中获取初始数据来创建一个表单实例。这个数据由来自外部网站图片的<code>url</code>和<code>title</code>属性构成，并且将由我们等会儿要创建的 JavaScript 工具提供。现在我们只是假设这里有初始数据。</li>
<li>如果表单被提交我们将检查它是否合法。如果这个表单是合法的，我们将新建一个<code>Image</code>实例，但是我们通过传递<code>commit=False</code>来保证这个对象将不会保存到数据库中。</li>
<li>我们将绑定当前用户（user）到一个新的<code>iamge</code>对象。这样我们就可以知道是谁上传了每一张图片。</li>
<li>我们把 iamge 对象保存到了数据库中</li>
<li>最后，我们使用 Django 的信息框架创建了一条上传成功的消息然后重定向用户到新图像的规范 URL 。我们没有在 Image 模型中实现<code>get_absolute_url()</code>方法，我们等会儿将编写它。</li>
</ol>
<p>在你的 images 应用中创建一个叫做<code>urls.py</code>的新文件，然后添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.conf.urls <span class="hljs-keyword">import</span> url
<span class="hljs-keyword">from</span> . <span class="hljs-keyword">import</span> views

urlpatterns = [
    url(<span class="hljs-string">r'^create/$'</span>, views.image_create, name=<span class="hljs-string">'create'</span>),
]</code></pre>
<p>像下面这样编辑在你项目文件夹中的主<code>urls.py</code>文件，将我们刚才为 images 应用创建的 url 模式添加进去：</p>
<pre class="hljs python"><code class="python">urlpatterns = [
    url(<span class="hljs-string">r'^admin/'</span>, include(admin.site.urls)),
    url(<span class="hljs-string">r'^account/'</span>, include(<span class="hljs-string">'account.urls'</span>)),
    url(<span class="hljs-string">r'^images/'</span>, include(<span class="hljs-string">'images.urls'</span>, namespace=<span class="hljs-string">'images'</span>)),
]</code></pre>
<p>最后，你需要创建一个模板来渲染你的表单。在你的 images 应用路径下创建如下路径结构：</p>
<pre class="hljs arduino"><code class="arduino">templates/
   images/
       <span class="hljs-built_in">image</span>/
          create.html</code></pre>
<p>编辑新的 <code>create.html</code> 模板然后添加以下代码进去：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Bookmark an image</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Bookmark an image<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> request.GET.url <span>}</span><span>}</span></span><span class="xml">)
    <span class="hljs-tag">&lt;<span class="hljs-name">form</span> <span class="hljs-attr">action</span>=<span class="hljs-string">"."</span> <span class="hljs-attr">method</span>=<span class="hljs-string">"post"</span>&gt;</span>
        </span><span class="hljs-template-variable"><span>{</span><span>{</span> form.as_p <span>}</span><span>}</span></span><span class="xml">
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">csrf_token</span></span> <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"Bookmark it!"</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>现在在你的浏览器中打开<code>http://127.0.0.1:8000/images/create/?title=...&amp;url=...</code>，记得在 后面传递 GET 参数<code>title</code>和<code>url</code>来提供一个已存在的JPG图像的 URL 。<br>举个例子，你可以使用像下面这样的 URL：</p>
<pre class="hljs awk"><code class="awk">http:<span class="hljs-regexp">//</span><span class="hljs-number">127.0</span>.<span class="hljs-number">0.1</span>:<span class="hljs-number">8000</span><span class="hljs-regexp">/images/</span>create<span class="hljs-regexp">/?title=%20Django%20and%20Duke&amp;url=http:/</span><span class="hljs-regexp">/upload.wikimedia.org/</span>wikipedia<span class="hljs-regexp">/commons/</span><span class="hljs-number">8</span><span class="hljs-regexp">/85/</span>Django_Reinhardt_and_Duke_Ellington_%<span class="hljs-number">28</span>Gottlieb%<span class="hljs-number">29</span>.jpg</code></pre>
<p>你可以看到一个带有图片预览的表单，就像下面这样：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-2.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-2.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-2</div>
</div><p><br>添加描述然后点击 <strong>Bookmark it！</strong>按钮。一个新的 <code>Image</code>对象将会被保存在你的数据库中。你将会得到一个错误，这个错误指示说<code>Image</code>模型没有<code>get_absolute_url()</code>方法。现在先不要担心这个，我们待会儿将添加这个方法、在你的浏览器中打开<code>http://127.0.0.1:8000/admin/images/image/</code>，确定新的图像对象已经被保存了。</p>
<h2><strong>用 jQuery 创建一个书签</strong></h2>
<p>书签是一个保存在浏览器中包含 JavaScript 代码的标签，用来拓展浏览器功能。当你点击书签的时候， JavaScript 代码会在浏览器显示的网站中被执行。这是一个在和其它网站交互时非常有用的工具。</p>
<p>一些在线服务，比如 Pinterest 实现了他们自己的书签来让用户可以在他们的平台中分享来自其他网站的内容，我们将以同样的方式创建一个书签，让用户可以在我们的网站中分享来自其他网站的图片。<br>我们将使用 jQuery 来创建我们的书签。 jQuery 是一个流行的 JavaScript 框架， 这个框架允许你快速开发客户端的功能。你可以在官网中更多的了解 jQuery: <a href="http://jquery.com/" target="_blank">http://jquery.com/</a> </p>
<p>你的用户将会像下面这样在他们的浏览器中添加书签然后使用它：</p>
<ol>
<li>用户从你的网站中拖拽一个链接到他的浏览器。这个链接在它的<code>href</code>属性中包含了 JavaScript 代码。这段代码将会被储存到书签当中。</li>
<li>用户访问任意一个网站，然后点击这个书签， 这个书签的 JavaScript 代码就被执行了。</li>
</ol>
<p>由于 JavaScript 代码将会以书签的形式被储存，之后你将不能更新它。这是个很显著的缺点，但是你可以通过实现一个简单的激活脚本来解决这个问题，这个脚本从一个 URL 中加载 JavaScript。你的用户将会以书签的形式来保存这个激活脚本，这样你就能在任何时候更新书签代码的内容了。我们将会采用这个方法来创建我们的书签。我们开始吧！<br>（译者注：上面这一段似乎有一点难以理解，其实很简单，就是把 JavaScript 保存在后端，只让用户保存一个能获取这段 JavaScript 的 url，url 是由书签来获取的。用户保存的就是这个含有获取 url 的 JavaScript 书签。）</p>
<p>在 image/templates/ 下创建一个新的模板，把它命名为 <code>bookmarklet_launcher.js</code>。这个就是我们的激活脚本了。将以下 JavaScript 代码添加进这个文件</p>
<pre class="hljs clojure"><code class="clojure">(<span class="hljs-name">function</span>(){
    if(<span class="hljs-name">window.myBookmarklet!==undefined</span>){
        myBookmarklet()<span class="hljs-comment">;</span>
    }
    else{
        document.body.appendChild(<span class="hljs-name">document.createElement</span>(<span class="hljs-name">'script'</span>)).src='http://127.0.0.1:8000/static/js/bookmarklet.js?r='+Math.floor(<span class="hljs-name">Math.random</span>()*99999999999999999999)<span class="hljs-comment">;</span>
    }
})()<span class="hljs-comment">;</span></code></pre>
<p>这段脚本通过检查 <code>myBookmarklet</code>变量是否被定义来检测书签是否被加载。这样，我们就可以避免在用户重复点击书签时重复加载。如果 <code>myBookmarklet</code> 没有被定义，我们就再加载一个 JavaScript 文件来在文档中添加一个<code>&lt;script&gt;</code>元素。 这个 <code>script</code> 标签加载 <code>bookmarklet_launcher.js</code>脚本，将一个随机数作为参数来防止加载浏览器缓存中的文件。</p>
<p>我们当前的 bookmarklet 代码位于 <code>bookmarklet.js</code> 静态文件中。这使我们在不要求用户更新书签的情况下更新我们代码。让我们把书签添加进 dashboard 页，我们的用户就可以将它拷贝到他们的书签中。</p>
<p>编辑  account/dashboard.html 模板，像如下一样更改它：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Dashboard</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Dashboard<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>

    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">with</span></span> total_images_created=request.user.images_created.count <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Welcome to your dashboard. You have bookmarked </span><span class="hljs-template-variable"><span>{</span><span>{</span> total_images_created <span>}</span><span>}</span></span><span class="xml"> image</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_images_created|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml">.<span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endwith</span></span> <span>%</span><span>}</span></span><span class="xml">

    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>Drag the following button to your bookmarks toolbar to bookmark images from other websites → <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"javascript:</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">include</span></span> "bookmarklet_launcher.js" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"button"</span>&gt;</span>Bookmark it!<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>

    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>You can also <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "edit" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>edit your profile<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span> or <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "password_change" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>change your password<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>.<span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个 danshboard 展示了用户所标记的图片总数。我们使用<code><span>{</span><span>%</span> with <span>%</span><span>}</span></code>模板标签来设置一个带有用户标记图片总数的参数。我们也引入了一个带有<code>href</code>属性的链接，这个链接含有我们的书签激活脚本。我们从<code>bookmarklet_launcher.js</code>模板中引入 JavaScript 脚本。</p>
<p>在你的浏览器中打开<code>http://127.0.0.1:8000/account/</code>,你可以看到如下页面：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-3.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-3.png" style="cursor: zoom-in;"><br><div class="image-caption">此处输入图片的描述</div>
</div>
<p>拖拽<code>Bookmark it!</code>链接到你的浏览器的书签工具栏中。</p>
<p>现在创建下面几个路径和文件在 images 应用路径中：</p>
<ul>
<li>static/</li>
<li>js/</li>
<li>bookmarklet.js</li>
</ul>
<p>你会在本章示例代码文件夹中的images 应用路径下找到 <code>static/css/</code> 路径。复制 <code>css/</code> 路径到你的代码文件夹下的<code>static/</code>中。<code>css/bookmarklet.css</code>文件为我们的 JavaScript 书签提供了样式。<br>编辑<code>bookmarklet.js</code>静态文件，然后添加以下 JavaScript 代码：</p>
<pre class="hljs javascript"><code class="javascript">(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
  <span class="hljs-keyword">var</span> jquery_version = <span class="hljs-string">'2.1.4'</span>;
  <span class="hljs-keyword">var</span> site_url = <span class="hljs-string">'http://127.0.0.1:8000/'</span>;
  <span class="hljs-keyword">var</span> static_url = site_url + <span class="hljs-string">'static/'</span>;
  <span class="hljs-keyword">var</span> min_width = <span class="hljs-number">100</span>;
  <span class="hljs-keyword">var</span> min_height = <span class="hljs-number">100</span>;

  <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">bookmarklet</span>(<span class="hljs-params">msg</span>) </span>{
      <span class="hljs-comment">// Here goes our bookmarklet code</span>
);
 <span class="hljs-comment">// Check if jQuery is loaded</span>
  <span class="hljs-keyword">if</span>(<span class="hljs-keyword">typeof</span> <span class="hljs-built_in">window</span>.jQuery != <span class="hljs-string">'undefined'</span>) {
    bookmarklet();
  } <span class="hljs-keyword">else</span> {
    <span class="hljs-comment">// Check for conflicts</span>
    <span class="hljs-keyword">var</span> conflict = <span class="hljs-keyword">typeof</span> <span class="hljs-built_in">window</span>.$ != <span class="hljs-string">'undefined'</span>;
    <span class="hljs-comment">// Create the script and point to Google API</span>
    <span class="hljs-keyword">var</span> script = <span class="hljs-built_in">document</span>.createElement(<span class="hljs-string">'script'</span>);
    script.setAttribute(<span class="hljs-string">'src'</span>,<span class="hljs-string">'http://ajax.googleapis.com/ajax/libs/jquery/'</span> + jquery_version + <span class="hljs-string">'/jquery.min.js'</span>);
    <span class="hljs-comment">// Add the script to the 'head' for processing</span>
    <span class="hljs-built_in">document</span>.getElementsByTagName(<span class="hljs-string">'head'</span>)[<span class="hljs-number">0</span>].appendChild(script);
    <span class="hljs-comment">// Create a way to wait until script loading</span>
    <span class="hljs-keyword">var</span> attempts = <span class="hljs-number">15</span>;
    (<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
      <span class="hljs-comment">// Check again if jQuery is undefined</span>
      <span class="hljs-keyword">if</span>(<span class="hljs-keyword">typeof</span> <span class="hljs-built_in">window</span>.jQuery == <span class="hljs-string">'undefined'</span>) {
        <span class="hljs-keyword">if</span>(--attempts &gt; <span class="hljs-number">0</span>) {
          <span class="hljs-comment">// Calls himself in a few milliseconds</span>
          <span class="hljs-built_in">window</span>.setTimeout(<span class="hljs-built_in">arguments</span>.callee, <span class="hljs-number">250</span>)
        } <span class="hljs-keyword">else</span> {
          <span class="hljs-comment">// Too much attempts to load, send error</span>
          alert(<span class="hljs-string">'An error ocurred while loading jQuery'</span>)
        }
      } <span class="hljs-keyword">else</span> {
          bookmarklet();
      }
    })();
  }

})()</code></pre>
<p>这是主要的 jQuery 加载脚本，当脚本已经加载到当前网站中时，它负责调用 JQuery 或者是从 Google 的 CDN 中加载 jQuery。当 jQuery 被加载，它会执行<code>bookmarklet()</code>函数，该函数包含我们的bookmarklet代码。我们还在这个文件顶部设置几个变量：</p>
<ul>
<li>
<code>jquery_version</code>: 加载的 jQuery 版本 </li>
<li>
<code>site_url</code>和<code>static_url</code>：我们网站的主URL 和各自静态文件的主URL</li>
<li>
<code>min_width</code>和<code>min_height</code>：我们的书签在网站中将要寻找的图像支持的最小宽度和最小高度，</li>
</ul>
<p>现在让我们来实现 <code>bookmarklet</code>函数，编辑<code>bookmarklet()</code>，让它看起来像这样：</p>
<pre class="hljs javascript"><code class="javascript">  <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">bookmarklet</span>(<span class="hljs-params">msg</span>) </span>{
    <span class="hljs-comment">// load CSS</span>
    <span class="hljs-keyword">var</span> css = jQuery(<span class="hljs-string">'&lt;link&gt;'</span>);
    css.attr({
      <span class="hljs-attr">rel</span>: <span class="hljs-string">'stylesheet'</span>,
      <span class="hljs-attr">type</span>: <span class="hljs-string">'text/css'</span>,
      <span class="hljs-attr">href</span>: static_url + <span class="hljs-string">'css/bookmarklet.css?r='</span> + <span class="hljs-built_in">Math</span>.floor(<span class="hljs-built_in">Math</span>.random()*<span class="hljs-number">99999999999999999999</span>)
    });
    jQuery(<span class="hljs-string">'head'</span>).append(css);

    <span class="hljs-comment">// load HTML</span>
    box_html = <span class="hljs-string">'&lt;div id="bookmarklet"&gt;&lt;a href="#" id="close"&gt;×&lt;/a&gt;&lt;h1&gt;Select an image to bookmark:&lt;/h1&gt;&lt;div class="images"&gt;&lt;/div&gt;&lt;/div&gt;'</span>;
    jQuery(<span class="hljs-string">'body'</span>).append(box_html);

      <span class="hljs-comment">// close event</span>
      jQuery(<span class="hljs-string">'#bookmarklet #close'</span>).click(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
      jQuery(<span class="hljs-string">'#bookmarklet'</span>).remove();
      });
      };</code></pre>
<p>这段代码运行如下：</p>
<ol>
<li>我们加载了<code>bookmarklet.css</code>样式表，使用一个随机的数字作为参数来避免浏览器的缓存</li>
<li>我们添加了定制的 HTML 到当前网站的<code>&lt;body&gt;</code>元素中。这个HTML由包含在当前网站寻找到的图片的<code>&lt;div&gt;</code>元素构成的。</li>
<li>我们添加了一个事件，当用户点击我们的 HTML 块中的关闭链接时，我们将移除我们添加进去的 HTML。我们使用 <code>#bookmarklet``#close</code>选择器来找到带有一个 ID 为<code>close</code>的 HTML 元素，这个 HTML 元素的父ID是 <code>bookmarklet</code>。jQuery 选择器允许你寻找 HTML 元素。jQuery 选择器返回所有给定的 CSS 选择器找到的元素，你可以在这个链接中找到一组 jQuery 选择器：<a href="http://api.jquery.com/category/selectors/" target="_blank">http://api.jquery.com/category/selectors/</a>
</li>
</ol>
<p>在加载了 CSS 样式表和 HTML 后，我们需要在网站中找到图片。在<code>bookmarklet()</code>函数的底部添加如下代码：</p>
<pre class="hljs processing"><code class="processing">    <span class="hljs-comment">// find images and display them</span>
    jQuery.each(jQuery(<span class="hljs-string">'img[src$="jpg"]'</span>), function(index, <span class="hljs-built_in">image</span>) {
      <span class="hljs-keyword">if</span> (jQuery(<span class="hljs-built_in">image</span>).<span class="hljs-built_in">width</span>() &gt;= min_width &amp;&amp; jQuery(<span class="hljs-built_in">image</span>).<span class="hljs-built_in">height</span>() &gt;= min_height)
      {
        image_url = jQuery(<span class="hljs-built_in">image</span>).attr(<span class="hljs-string">'src'</span>);
        jQuery(<span class="hljs-string">'#bookmarklet .images'</span>).<span class="hljs-built_in">append</span>(<span class="hljs-string">'&lt;a href="#"&gt;![]('</span>+ image_url +<span class="hljs-string">')&lt;/a&gt;'</span>);
      }
    });</code></pre>
<p>这段代码使用了<code>img[src$="jpg"]</code>选择器来找到所有的<code>&lt;img&gt;</code> HTML 元素，并且这些元素的<code>src</code>属性以<code>jpg</code>结尾。这意味着我们会找到当前网页中所有的 JPG 图片。我们通过<code>each()</code>方法来遍历所有的结果。我们添加了<code>&lt;div class="images"&gt;</code> HTML 容器用以放置图片，容器的的尺寸刚好比<code>min_width</code>和<code>min_width</code>大一点。</p>
<p>这个 HTML 容器现在包含了可以被打上标签的图片，我们想要用户点击他们需要的图片然后给他们打上标签。在<code>bookmarklet()</code>函数中添加以下代码：</p>
<pre class="hljs javascript"><code class="javascript">    <span class="hljs-comment">// when an image is selected open URL with it</span>
    jQuery(<span class="hljs-string">'#bookmarklet .images a'</span>).click(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">e</span>)</span>{
      selected_image = jQuery(<span class="hljs-keyword">this</span>).children(<span class="hljs-string">'img'</span>).attr(<span class="hljs-string">'src'</span>);
      <span class="hljs-comment">// hide bookmarklet</span>
      jQuery(<span class="hljs-string">'#bookmarklet'</span>).hide();
      <span class="hljs-comment">// open new window to submit the image</span>
      <span class="hljs-built_in">window</span>.open(site_url +<span class="hljs-string">'images/create/?url='</span>
                  + <span class="hljs-built_in">encodeURIComponent</span>(selected_image)
                  + <span class="hljs-string">'&amp;title='</span> + <span class="hljs-built_in">encodeURIComponent</span>(jQuery(<span class="hljs-string">'title'</span>).text()),
                  <span class="hljs-string">'_blank'</span>);
    });</code></pre>
<p>这段代码按照如下流程运行：</p>
<ol>
<li>我们把一个<code>clck()</code>事件绑定到了图片的链接元素上</li>
<li>当一个用户点击一个图片时我们新建了一个变量<code>selected_image</code>，这个变量包含了被选择的图片的 URL。</li>
<li>我们隐藏了书签然后在浏览器中打开一个新的窗口，这个窗口访问了我们的网站中为一个新的图片打标签的 URL 。我们传递了网站的<code>title</code>元素和被选中图片的 URL 作为 GET 参数。</li>
</ol>
<p>在你的浏览器中随便选择一个网址打开，然后点击你的书签。你将会看到一个白色的新窗口出现在当前网页上，它展示了所有尺寸大于 100*100px 的 JPG 图片，它看起来就像下面的例子一样：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-4.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-4.png" style="cursor: zoom-in;"><br><div class="image-caption">django-5-4</div>
</div><p><br>因为我们已经开启了 Django 的开发服务器，使用 HTTP 来提供页面， 由于安全限制，书签将不能在 HTTPS 上工作。</p>
<p>如果你点击一幅图片，你将会被重定向到创建图片的页面，请求地址传递了网站的标题和被选中图片的 URL 作为 GET 参数。</p>
<div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-5.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-5.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-5</div>
</div><p><br>恭喜！这是你的第一个 JavaScript 书签！现在它已经和你的 Django 项目成为一体！</p>
<h2><strong>为你的图片创建一个详情视图</strong></h2>
<p>我们将创建一个简单的详情视图，用于展示一张已经保存在我们的网站中的图片。打开 images 应用的<code>views.py</code>，将以下代码添加进去：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.shortcuts <span class="hljs-keyword">import</span> get_object_or_404
<span class="hljs-keyword">from</span> .models <span class="hljs-keyword">import</span> Image
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_detail</span><span class="hljs-params">(request, id, slug)</span>:</span>
    image = get_object_or_404(Image, id=id, slug=slug)
    <span class="hljs-keyword">return</span> render(request, <span class="hljs-string">'images/image/detail.html'</span>, {<span class="hljs-string">'section'</span>:                                                                 <span class="hljs-string">'images'</span>,<span class="hljs-string">'image'</span>: image})</code></pre>
<p>这是一个用于展示图片的简单视图。编辑 iamges 应用的 <code>urls.py</code>，添加以下 URL 模式：</p>
<pre class="hljs python"><code class="python">url(<span class="hljs-string">r'^detail/(?P&lt;id&gt;\d+)/(?P&lt;slug&gt;[-\w]+)/$'</span>,
              views.image_detail, name=<span class="hljs-string">'detail'</span>),</code></pre>
<p>编辑 images 应用的<code>models.py</code>,并且将<code>get_absolute_url()</code>方法添加进 <code>Image</code> 模型:</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.core.urlresolvers <span class="hljs-keyword">import</span> reverse
<span class="hljs-class"><span class="hljs-keyword">class</span> <span class="hljs-title">Image</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="hljs-comment"># ...</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">get_absolute_url</span><span class="hljs-params">(self)</span>:</span>
        <span class="hljs-keyword">return</span> reverse(<span class="hljs-string">'images:detail'</span>,args=(self.id,self.slug))</code></pre>
<p>记住，为对象提供精确 URL 的通用模式是在模型中定义<code>get_absolute_url()</code>方法。</p>
<p>最后，在 images 应用的 模版路径<code>/images/image/</code>中新建一个模板，命名为<code>detail.html</code>，添加以下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml"></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.title <span>}</span><span>}</span></span><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.title <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> image.image.url <span>}</span><span>}</span></span><span class="xml">)
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">with</span></span> total_likes=image.users_like.count <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"image-info"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
                    <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"count"</span>&gt;</span>
                        </span><span class="hljs-template-variable"><span>{</span><span>{</span> total_likes <span>}</span><span>}</span></span><span class="xml">like</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_likes|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml">
                    <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
                 <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
                 </span><span class="hljs-template-variable"><span>{</span><span>{</span> image.description|<span class="hljs-name">linebreaks</span> <span>}</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"image-likes"</span>&gt;</span>
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> user <span class="hljs-keyword">in</span> image.users_like.all <span>%</span><span>}</span></span><span class="xml">
                <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
                    ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> user.profile.photo.url <span>}</span><span>}</span></span><span class="xml">)
                    <span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> user.first_name <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span>
                <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">empty</span></span> <span>%</span><span>}</span></span><span class="xml">
                Nobody likes this image yet.
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endwith</span></span> <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模版用来展示一张被打标签图片。我们使用<code><span>{</span><span>%</span> with <span>%</span><span>}</span></code>标签来保存所有统计<code>user likes</code>查询集（QuerySet）的结果，并将这个结果保存在一个新的变量<code>total_likes</code>中。这样我们就可以避免计算两次查询集（QuerySet）的结果。我们也引入了图片的描述，迭代了<code>image.users_like.all</code>来展示所有喜欢这张图片的用户。</p>
<blockquote><p>使用<code><span>{</span><span>%</span> with <span>%</span><span>}</span></code>模版标签来防止 Django 做多次查询是很有用的</p></blockquote>
<p>现在使用书签来为一张图片打上标签。在你提交图片之后你将会被重定向图片详情页面。这张图片将会包含一条提交成功的消息，效果如下：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-6.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-6.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-6</div>
</div>
<h2><strong>使用 sorl-thumbnail 创建缩略图</strong></h2>
<p>我们在详情页展示原图片，但是不同的图片的尺寸是不同的。一些图片源文件或许会非常大，加载他们会耗费很长时间。展示规范图片的最好方法是生成缩略图。我们将使用一个 Django 应用，叫做<code>sorl-thumbnail</code>。</p>
<p>打开终端，用下面的命令来安装<code>sorl-thumbnail</code>：</p>
<pre class="hljs python"><code class="python">pip install sorl-thumbnail==<span class="hljs-number">12.3</span></code></pre>
<p>编辑 bookmarklet 项目文件的<code>settings.py</code>,将<code>sorl-thumbnail</code>添加进<code>INSTALLED_APPS</code>.</p>
<p>运行下面的命令来同步你的数据库：</p>
<pre class="hljs python"><code class="python">python manage.py migrate</code></pre>
<p>你看到的输出中应该包含下面这一行：</p>
<pre class="hljs gams"><code class="gams">Creating <span class="hljs-keyword">table</span> thumbnail_kvstore</code></pre>
<p><code>sorl-thumbnail</code>应用提供了不同的方法来定义一张图片的缩略图。它提供了<code><span>{</span><span>%</span> thumbnail <span>%</span><span>}</span></code>模版标签来在模版中生成缩略图，同时还有一个定制的<code>ImageField</code>字段，如果你想要在你的模型中定制缩略图的话。我们将要使用这个模版标签。编辑 <code>images/image/detail.html</code>模版，删除这一行：</p>
<pre class="hljs stylus"><code class="stylus">![](<span>{</span><span>{</span> image<span class="hljs-selector-class">.image</span><span class="hljs-selector-class">.url</span> <span>}</span><span>}</span>)</code></pre>
<p>替换成：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> thumbnail <span>%</span><span>}</span></span><span class="xml">
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">thumbnail</span> image.image "300" <span class="hljs-keyword">as</span> im <span>%</span><span>}</span></span><span class="xml">
<span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.image.url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> im.url <span>}</span><span>}</span></span><span class="xml">)
<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">endthumbnail</span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这里，我们定义了一个固定宽度为 300px 的缩略图。当用户第一次加载这页面时，缩略图将会被创建。生成的缩略图将会在接下来的请求中被使用。运行<code>python manage.py runserver</code>开启开发服务器，连接到一张已有图片的详情页。缩略图将会生成并展示在网站中。</p>
<p><code>sorl-thumbnail</code>应用提供了几个选择来定制你的缩略图，包括裁减算法和能被应用的不同效果。如果你有任何生成缩略图的疑难点，你可以在你的设置中添加<code>THUMBNAIL_DEBUG = TRUE</code>来获得 debug 信息。你可以阅读<code>sorl-thumbnail</code>的完整文档：<a href="http://sorl-thumbnail.readthedocs.org/" target="_blank">http://sorl-thumbnail.readthedocs.org/</a></p>
<h2><strong>用 jQuery 添加 AJAX 动作</strong></h2>
<p>现在我们将在你的应用中添加 AJAX 动作。AJAX 源于 Asynchronous JavaScript and XML（异步 JavaScript 和 XML）。这个术语包含一组可以制造异步 HTTP 请求的技术，它包含从服务器异步发送和接收数据，不需要重载整个页面，虽然它的名字里有 XML， 但是 XML 不是必需的。你可以以其他的格式发送或者接收数据，如 JSON， HTML，或者是纯文本。</p>
<p>我们将在图片详情页添加一个供用户点击的链接，表示他们喜欢这张图片。我们将会用 AJAX 来避免重载整个页面。首先，在 <code>views.py</code> 中创建一个可供用户点击“喜欢”或“不喜欢”的视图。编辑 images 应用的<code>views.py</code>，将以下代码添加进去：</p>
<pre class="hljs python"><code class="python"><span class="hljs-meta">@login_required</span>
<span class="hljs-meta">@require_POST</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_like</span><span class="hljs-params">(request)</span>:</span>
    image_id = request.POST.get(<span class="hljs-string">'id'</span>)
    action = request.POST.get(<span class="hljs-string">'action'</span>)
    <span class="hljs-keyword">if</span> image_id <span class="hljs-keyword">and</span> action:
        <span class="hljs-keyword">try</span>:
            image = Image.objects.get(id=image_id)
            <span class="hljs-keyword">if</span> action == <span class="hljs-string">'like'</span>:
                image.users_like.add(request.user)
            <span class="hljs-keyword">else</span>:
                image.users_like.remove(request.user)
            <span class="hljs-keyword">return</span> JsonResponse({<span class="hljs-string">'status'</span>:<span class="hljs-string">'ok'</span>})
        <span class="hljs-keyword">except</span>:
            <span class="hljs-keyword">pass</span>
    <span class="hljs-keyword">return</span> JsonResponse({<span class="hljs-string">'status'</span>:<span class="hljs-string">'ko'</span>})</code></pre>
<p>我们在这个视图中使用了两个装饰器。 <code>login_required</code> 装饰器阻止未登录的用户连接到这个视图。<code>require_GET</code> 装饰器返回一个<code>HttpResponseNotAlloed</code>对象（状态吗：405）如果 HTTP 请求不是 GET 。这样就可以只允许 GET 请求来访问这个视图。 Django 同样也提供了<code>require_POST</code>装饰器来只允许 POST 请求，以及一个可让你传递一组请求方法作为参数的 <code>require_http_methods</code>装饰器。</p>
<p>在这个视图中我们使用了两个 GET 参数：</p>
<ol>
<li>
<code>image_id</code>:用户操作的 image 对象的 ID </li>
<li>
<code>action</code>: 用户想要执行的动作。我们把它的值设定为<code>like</code>或者'<code>dislike</code>
</li>
</ol>
<p>我们在<code>Image</code>模型的多对多字段<code>users_like</code>上使用 Django 提供的管理器来添加或者删除对象关系通过调用<code>add()</code>或者<code>remove()</code>方法来执行这些动作。调用<code>add()</code>时传递一个存在于关联模型中的对象集不会重复添加这个对象，同样，调用<code>remove()</code>时传递一个不存在于关联模型中的对象集什操作也不会执行。另一个有用的多对多管理器是<code>clear()</code>，它将删除所有的关联对象集。</p>
<p>最后，我们使用 Django 提供的<code>JsonResponse</code>类来将给你定的对象转换为一个 JSON 输出，这个类返回一个带有<code>application/json</code>内容类型的 HTTP 响应。</p>
<p>编辑 <code>images</code> 应用中的 <code>urls.py</code>,添加以下 URL 模式：</p>
<pre class="hljs awk"><code class="awk">url(<span class="hljs-string">r'^like/$'</span>, views.image_like, name=<span class="hljs-string">'like'</span>),</code></pre>
<h2><strong>加载 jQuery</strong></h2>
<p>我们需要在我们的图片详情页中添加 AJAX 功能。我们首先将在 <code>base.html</code>模版中引入 AJAX。编辑 account 应用的 <code>base.html</code>模版，然后将以下代码在<code>&lt;/body&gt;</code>标签前添加以下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/
jquery.min.js"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  $(<span class="hljs-built_in">document</span>).ready(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
    </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> domready <span>%</span><span>}</span></span><span class="xml"><span class="undefined">
    </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="undefined">
    });
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span></code></pre>
<p>我们从 Google 加载 jQuery 框架，Google提供了一个在高速内容分发网络中的流行 JavaScript 框架。你也可以自己下载 jQuery, 地址：<a href="http://jquery.com/" target="_blank">http://jquery.com/</a> 。然后将下载的文件添加进应用的<code>static</code>路径下。</p>
<p>我们添加<code>&lt;script&gt;</code>标签来引入 JavaScript 代码。<code>$(dovument).ready()</code>是一个 jQuery 函数，这个函数会在 DOM 层加载完毕后执行。 DON 源于 Document Object Model。当一个页面被载入时，DOM 会由浏览器创建， DOM 被创建为一个树对象。通过在这个函数中包含我们的代码来确保我们可以与DOM中加载的所有HTML元素都能进行交互操作。我们的代码仅仅会在 DOM 对象被加载完毕之后执行。</p>
<p>在文档预处理函数中，我们会在模板中引入一个 Django 模板块叫做 <strong>domready</strong>, 在扩展了基础模版之后将会引入特定的 JavaScript 。</p>
<p>不要将 JavaScript 代码和 Django 模板标签搞混了。 Django 模板语言是在服务端被渲染并输出最终的 HTML 文档，JavaScript 是在客户端被执行的。在某些情况下，使用 Django 动态生成 JavaScript 很有用。</p>
<blockquote><p>在这一章中，我们在 Django 模板中引入（include）了 JavaSript 代码。更好的引入方法是加载（load） JavaSript. <code>js</code>文件是作为静态文件被提供的，特别在有大量脚本时尤其如此。</p></blockquote>
<h2><strong>AJAX 请求中的跨站请求攻击（CSRF）</strong></h2>
<p>你已经在第二章了解到了跨站请求攻击，在CSRF保护激活的情况下， Django 会检查所有 POST 请求中的 CSRF token。当你提交表但时，你可以使用<code><span>{</span><span>%</span> csrf_token <span>%</span><span>}</span></code>模板标签来发送带有 token 的表单。无论如何，像 POST 请求一样对 AJAX 请求传递CDRF token 有一点点不方便。因此，Django 允许你在你的 AJAX 请求中设置一个定制的 <code>X-CSRFToken</code> token 头（header）。这允许你安装一个 jQuery 或者任意 JavaScript 库来自动设置<code>X-CSRFToken</code>头在每一次请求中。</p>
<p>为了在所有的请求中加入 token ，你需要：</p>
<ol>
<li>从 <code>csrftoken</code> cookie 中检索 CSRF token，它在CSRF保护激活的情况下会被设置</li>
<li>使用 <code>X-CSRFToken</code>头发送 token 到 AJAX 中</li>
</ol>
<p>你可以找到更多关于 CSRF 保护 和 AJAX 的信息：<a href="http://docs.djangoproject.com/en/1.8/ref/csrf/#ajax" target="_blank">http://docs.djangoproject.com/en/1.8/ref/csrf/#ajax</a></p>
<p>在你的<code>base.html</code>模板中添加最后一段代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">"https://ajax.googleapis.com/ajax/libs/jquery/2.1.4/
jquery.min.js"</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">script</span> <span class="hljs-attr">src</span>=<span class="hljs-string">" http://cdn.jsdelivr.net/jquery.cookie/1.4.1/jquery.
cookie.min.js "</span>&gt;</span><span class="undefined"></span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span>
<span class="hljs-tag">&lt;<span class="hljs-name">script</span>&gt;</span><span class="javascript">
  <span class="hljs-keyword">var</span> csrftoken = $.cookie(<span class="hljs-string">'csrftoken'</span>);
  <span class="hljs-function"><span class="hljs-keyword">function</span> <span class="hljs-title">csrfSafeMethod</span>(<span class="hljs-params">method</span>) </span>{
    <span class="hljs-comment">// these HTTP methods do not require CSRF protection</span>
    <span class="hljs-keyword">return</span> (<span class="hljs-regexp">/^(GET|HEAD|OPTIONS|TRACE)$/</span>.test(method));
}
$.ajaxSetup({
  <span class="hljs-attr">beforeSend</span>: <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">xhr, settings</span>) </span>{
    <span class="hljs-keyword">if</span> (!csrfSafeMethod(settings.type) &amp;&amp; !<span class="hljs-keyword">this</span>.crossDomain) {
      xhr.setRequestHeader(<span class="hljs-string">"X-CSRFToken"</span>, csrftoken);
  }
}
});
$(<span class="hljs-built_in">document</span>).ready(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>)</span>{
    </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> domready <span>%</span><span>}</span></span><span class="xml"><span class="undefined">
    </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="undefined">
  });
</span><span class="hljs-tag">&lt;/<span class="hljs-name">script</span>&gt;</span></span></code></pre>
<p>上面这段代码解释如下：</p>
<ol>
<li>我们从一个公共的 CDN 中载入了一个 jQuery Cookie 插件，这样我们就可以和 cookies 交互。</li>
<li>读取 <code>csrftoken</code> cookie</li>
<li>我们将定义<code>csrfSafeMethod</code>函数来检查一个 HTTP 方法是否安全。安全方法不要求 CSRF 保护，他们分别是 GET, HEAD, OPTIONS, TRACE。</li>
<li>我们用<code>$.ajaxSetup()</code>设置了 jQuery AJAX 请求,在每个 AJAX 请求执行前，我们会检查请求方法是否安全和当前请求是否跨域名。如果请求是不安全的，我们将用从 cookie 中获得的值来设置 <code>X-CSRFToken</code>头。这个设置将会应用到所有由 jQuery 执行的 AJAX 请求中</li>
</ol>
<p>CSRF token将会在所有的不安全 HTTP 方法的 AJAX 请求中引入，比如 POST, PUT</p>
<h2><strong>用 JQuery 执行 AJAX请求</strong></h2>
<p>编辑 images 应用中的 <code>images/image/detailmhtml</code>模板，删除这一行：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">with</span></span> total_likes=image.users_like.count <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>替换为：</p>
<pre class="hljs sqf"><code class="sqf"><span>{</span><span>%</span> <span class="hljs-keyword">with</span> total_likes=<span class="hljs-built_in">image</span>.users_like.<span class="hljs-built_in">count</span> users_like=<span class="hljs-built_in">image</span>.users_like.all <span>%</span><span>}</span></code></pre>
<p>用<code>image-info</code>类属性修改<code>&lt;div</code>元素：</p>
<pre class="hljs django"><code class="django"><span class="xml">        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"image-info"</span>&gt;</span>
                <span class="hljs-tag">&lt;<span class="hljs-name">div</span>&gt;</span>
                    <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"count"</span>&gt;</span>
                        <span class="hljs-tag">&lt;<span class="hljs-name">span</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"total"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> total_likes <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
                        like</span><span class="hljs-template-variable"><span>{</span><span>{</span> total_likes|<span class="hljs-name">pluralize</span> <span>}</span><span>}</span></span><span class="xml">
                    <span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span>
                    <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"#"</span> <span class="hljs-attr">data-id</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.id <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">data-action</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user <span class="hljs-keyword">in</span> users_like <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">un</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">like"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"like button"</span>&gt;</span>
                        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> request.user not <span class="hljs-keyword">in</span> users_like <span>%</span><span>}</span></span><span class="xml">
                            Like
                        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">else</span></span> <span>%</span><span>}</span></span><span class="xml">
                            Unlike
                        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml">
                    <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
                <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
            </span><span class="hljs-template-variable"><span>{</span><span>{</span> image.description|<span class="hljs-name">linebreaks</span> <span>}</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre>
<p>首先，我们添加了一个变量到<code><span>{</span><span>%</span> with <span>%</span><span>}</span></code>模板标签中来保存<code>image.uers_like.all</code>查询接的结果来避免执行两次查询。展示喜欢这张图片用户的总数，包含一个“like/unlike”链接。我们检查用户是否在关联对象<code>`user_likes</code>中，基于当前的用户和图片的关系展示 like 或者 unlike。。我们将以下属性添加进了<code>&lt;a&gt;</code> HTML 元素中：</p>
<ul>
<li>
<code>data-id</code>:被展示图片的 ID</li>
<li>
<code>data-action</code>：当用户点击这个链接时执行这个动作。这个动作可以是 like 或者是 unlike<br>我们将会在向 <code>iamge_like</code>视图的 AJAX 请求中添加这两个属性值。当一个用户点击<code>like/unlike</code>链接时，我们需要在客户端执行以下几个动作：</li>
<li>调用 AJAX 视图，并把图片的 ID 和动作参数传递进去</li>
<li>如果 AJAX 请求成功， 更新<code>&lt;a&gt;</code> HTML元素的<code>data-action</code>属性（like / unlike），根据此来修改它展示的文本</li>
<li>更新展示的 likes 的总数</li>
</ul>
<p>在<code>images/image/detail.html</code>模板中添加<code>domready</code>块，使用如下 JavaScript 代码：</p>
<pre class="hljs javascript"><code class="javascript"><span>{</span><span>%</span> block domready <span>%</span><span>}</span>
    $(<span class="hljs-string">'a.like'</span>).click(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">e</span>)</span>{
        e.preventDefault();
        $.post(<span class="hljs-string">'<span>{</span><span>%</span> url "images:like" <span>%</span><span>}</span>'</span>,
            {
                <span class="hljs-attr">id</span>: $(<span class="hljs-keyword">this</span>).data(<span class="hljs-string">'id'</span>),
                <span class="hljs-attr">action</span>: $(<span class="hljs-keyword">this</span>).data(<span class="hljs-string">'action'</span>)
            },
            <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">data</span>)</span>{
                <span class="hljs-keyword">if</span> (data[<span class="hljs-string">'status'</span>] == <span class="hljs-string">'ok'</span>)
                {
                    <span class="hljs-keyword">var</span> previous_action = $(<span class="hljs-string">'a.like'</span>).data(<span class="hljs-string">'action'</span>);

                    <span class="hljs-comment">// toggle data-action</span>
                    $(<span class="hljs-string">'a.like'</span>).data(<span class="hljs-string">'action'</span>, previous_action == <span class="hljs-string">'like'</span> ? <span class="hljs-string">'unlike'</span> : <span class="hljs-string">'like'</span>);
                    <span class="hljs-comment">// toggle link text</span>
                    $(<span class="hljs-string">'a.like'</span>).text(previous_action == <span class="hljs-string">'like'</span> ? <span class="hljs-string">'Unlike'</span> : <span class="hljs-string">'Like'</span>);

                    <span class="hljs-comment">// update total likes</span>
                    <span class="hljs-keyword">var</span> previous_likes = <span class="hljs-built_in">parseInt</span>($(<span class="hljs-string">'span.count .total'</span>).text());
                    $(<span class="hljs-string">'span.count .total'</span>).text(previous_action == <span class="hljs-string">'like'</span> ? previous_likes + <span class="hljs-number">1</span> : previous_likes - <span class="hljs-number">1</span>);
                }
        });

    });
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre>
<p>这段代码工作流程如下：</p>
<ol>
<li>我们使用<code>$.('a.like')</code> jQuery 选择器来找到所有的 class 属性是 like 的<code>&lt;a&gt;</code>标签</li>
<li>我们为点击事件定义了一个控制器函数。这个函数会在用户每次点击<code>like/unlike</code>时执行<br>3.在控制器函数中，我们使用<code>e.preventDefault()</code>来避免<code>&lt;a&gt;</code>标签的默认行为。这会阻止链接把我们带到其他地方。</li>
<li>我们使用<code>$.post()</code>向服务器执行一个异步 POST 请求。 jQuery 也会提供一个<code>$.get()</code>方法来执行 GET 请求和一个低级别的 <code>$.ajax()</code>方法。</li>
<li>我们使用 Django 的<code><span>{</span><span>%</span> url <span>%</span><span>}</span></code>模板标签来构建为 AJAX 请求需要的URL</li>
<li>我们在请求中建立要发送的 POST 参数字典。他们是 Django 视图中期望的 <code>ID</code>和<code>action</code>参数。我们从<code>&lt;a&gt;</code>元素的<code>data-id</code>和<code>data-action</code>中获取两个参数的值。</li>
<li>我们定义了一个当 HTTP 应答被接收时的回调函数。它接收一个含有应答内容的数据属性。</li>
<li>我们获取接收数据的<code>status</code>属性然后检查它的值是否是<code>ok</code>。如果返回的<code>data</code>是期望中的那样，我们将切换<code>data-action</code>属性的链接和它的文本内容。这可以让用户取消这个动作。</li>
<li>我们基于执行的动作来增加或者减少 likes 的总数</li>
</ol>
<p>在你的浏览器中打开一张你上传的图片的详情页，你可以看到初始的 like 统计和一个 <code>LIKE</code> 按钮：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-7.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-7.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-7</div>
</div><p><br>点击<strong><code>LIKE</code></strong>按钮，你将会看见 likes 的总数上升了，按钮的文本也变成了<strong><code>UNLIKE</code></strong>：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-8.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-8.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-8</div>
</div><p><br>当你点击<strong><code>UNLIKE</code></strong>按钮时动作被执行，按钮的文本也会变成<strong><code>LIKE</code></strong>，统计的总数也会据此下降。</p>
<p>在编写 JavaScript 时，特别是在写 AJAX 请求时， 我们建议应该使用一个类似于 Firebug 的工具来调试你的 JavaScript 脚本以及监视 CSS 和 HTML 的变化，你可以下载 Firebug ： <a href="http://getfirebug.com/%E3%80%82%E4%B8%80%E4%BA%9B%E6%B5%8F%E8%A7%88%E5%99%A8%E6%AF%94%E5%A6%82*Chrome*%E6%88%96%E8%80%85*Safari*%E4%B9%9F%E5%8C%85%E5%90%AB%E4%B8%80%E4%BA%9B%E8%B0%83%E8%AF%95" target="_blank">http://getfirebug.com/。一些浏览器比如*Chrome*或者*Safari*也包含一些调试</a> JavaScript 的开发者工具。在那些浏览器中，你可以在网页的任何地方右键然后点击<strong>Inspect element</strong>来使用网页开发者工具。</p>
<h2><strong>为你的视图创建定制化的装饰器</strong></h2>
<p>我们将会限制我们的 AJAX 视图只接收由 AJAX 发起的请求。Django Request 对象提供了一个 <code>is_ajax()</code>方法， 这个方法会检查请求是否带有<code>XMLHttpRequest</code>,也就是说，会检查这个请求是否是一个 AJAX 请求。这个值被设置在<code>HTTP_X_REQUESTED_WITH HTTP</code>头中， 这个头被大多数的由JavaScript库发起的 AJAX 请求包含。</p>
<p>我们将在我们的视图中创建一个装饰器，来检测<code>HTTP_X_RQUESTED_WITH</code>头。装饰器是一个可以接收一个函数为参数的函数，并且它可以在不改变作为参数的函数的情况下，拓展此函数的功能。如果装饰器的概念对你来说还很陌生，在继续阅读之前你或许可以看看这个：https:www.python.org/dev/peps/pep-0318/。</p>
<p>由于我们的装饰器将会是通用的，它将被应用到任何视图中，所以我们在我们的项目中将创建一个 <code>common</code>Python 包，在 bookmarklet 项目中创建如下路径：</p>
<ul>
<li>common/</li>
<li>_<em>init_</em>.py</li>
<li>decorators.py</li>
</ul>
<p>编辑<code>decorators.py</code>,添加如下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.http <span class="hljs-keyword">import</span> HttpResponseBadRequest

<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">ajax_required</span><span class="hljs-params">(f)</span>:</span>
    <span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">wrap</span><span class="hljs-params">(request, *args, **kwargs)</span>:</span>
            <span class="hljs-keyword">if</span> <span class="hljs-keyword">not</span> request.is_ajax():
                <span class="hljs-keyword">return</span> HttpResponseBadRequest()
            <span class="hljs-keyword">return</span> f(request, *args, **kwargs)
    wrap.__doc__=f.__doc__
    wrap.__name__=f.__name__
    <span class="hljs-keyword">return</span> wrap</code></pre>
<p>这是我们定制的<code>ajax_required</code>装饰器。它定义一个当请求不是 AJAX 时返回<code>HttpResponseBadRequest</code>（HTTP 400）对象的wrap 函数，否则它将返回一个被装饰了的对象。</p>
<p>现在你可以编辑 images 应用的<code>views.py</code>，为你的 <code>image_like</code> AJAX 视图添加这个装饰器：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> common.decorators <span class="hljs-keyword">import</span> ajax_required

<span class="hljs-meta">@ajax_required</span>
<span class="hljs-meta">@login_required</span>
<span class="hljs-meta">@require_POST</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_like</span><span class="hljs-params">(request)</span>:</span>
    <span class="hljs-comment"># ...</span></code></pre>
<p>如果你直接在你的浏览器中访问<code>http://127.0.0.1:8000/images/like/</code>,你将会得到一个 HTTP 400 的错误。</p>
<blockquote><p>如果你发现你正在视图中执行重复的检查，请为你的视图创建装饰器</p></blockquote>
<h2><strong>在你的列表视图中添加 AJAX 分页</strong></h2>
<p>我们需要在你的网站中列出所有的被标签的图片。我们将要使用 AJAX 分页来建立一个不受限的滚屏功能。不受限的滚屏是在用户滚动到底部时，自动加载下一页的结果来实现的。</p>
<p>我们将实现一个图片列表视图，这个视图既可以支持标准的浏览器请求，也支持包含分页的 AJAX 请求。当用户首次加载列表页时，我们展示第一页的图片。当用户滚动到底部时，我们用 AJAX 加载下一页的内容，然后将内容加入到页面的底部。</p>
<p>编辑 images 应用的<code>views.py</code>，添加以下代码：</p>
<pre class="hljs python"><code class="python"><span class="hljs-keyword">from</span> django.http <span class="hljs-keyword">import</span> HttpResponse
<span class="hljs-keyword">from</span> django.core.paginator <span class="hljs-keyword">import</span> Paginator, EmptyPage, PageNotAnInteger

<span class="hljs-meta">@login_required</span>
<span class="hljs-function"><span class="hljs-keyword">def</span> <span class="hljs-title">image_list</span><span class="hljs-params">(request)</span>:</span>
    images = Image.objects.all()
    paginator = Paginator(images, <span class="hljs-number">8</span>)
    page = request.GET.get(<span class="hljs-string">'page'</span>)
    <span class="hljs-keyword">try</span>:
        images = paginator.page(page)
    <span class="hljs-keyword">except</span> PageNotAnInteger:
        <span class="hljs-comment"># If page is not an integer deliver the first page</span>
        images = paginator.page(<span class="hljs-number">1</span>)
    <span class="hljs-keyword">except</span> EmptyPage:
        <span class="hljs-keyword">if</span> request.is_ajax():
            <span class="hljs-comment"># If the request is AJAX and the page is out of range return an empty page</span>
            <span class="hljs-keyword">return</span> HttpResponse(<span class="hljs-string">''</span>)
        <span class="hljs-comment"># If page is out of range deliver last page of results</span>
        images = paginator.page(paginator.num_pages)
    <span class="hljs-keyword">if</span> request.is_ajax():
        <span class="hljs-keyword">return</span> render(request,
                      <span class="hljs-string">'images/image/list_ajax.html'</span>,
                      {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>, <span class="hljs-string">'images'</span>: images})
    <span class="hljs-keyword">return</span> render(request,
                  <span class="hljs-string">'images/image/list.html'</span>,
                   {<span class="hljs-string">'section'</span>: <span class="hljs-string">'images'</span>, <span class="hljs-string">'images'</span>: images})</code></pre>
<p>在这个视图中，我们创建一个查询集（QuerySet）来从数据库中获得所有的图片。然后我们创建了一个<code>Paginator</code>对象来分页查询结果，每页有八张图片。如果请求的页面超出范围了，我们将会得到一个<code>EmptyPage</code>异常，在这种情况下并且请求又是由 AJAX 发起的话，我们将会返回一个空的<code>HttpResponse</code>，这将帮助我们在客户端停止 AJAX 分页，我们将会把结果渲染给两个不同的模板：</p>
<ol>
<li>对于 AJAX 请求，我们渲染<code>list_ajax.html</code>模板。这个模板将只会包含我们请求页面的图片</li>
<li>对于标准请求： 我们渲染<code>list.html</code>模板。这个模板将会继承<code>base.html</code>来展示整个页面，并且<code>list_ajax.html</code>页面也会被引入在其中。</li>
</ol>
<p>编辑 images 应用的<code>urls.py</code>,添加以下 URL 模式：</p>
<pre class="hljs awk"><code class="awk">url(<span class="hljs-string">r'^$'</span>, views.image_list, name=<span class="hljs-string">'list'</span>),</code></pre>
<p>最后，我们需要创建我们在上面提到的模板。在 images/image/ 下创建一个名为'list_ajax.html'的模板，添加以下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> thumbnail <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">for</span></span> image <span class="hljs-keyword">in</span> images <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"image"</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">thumbnail</span> image.image "300x300" crop="100%" <span class="hljs-keyword">as</span> im <span>%</span><span>}</span></span><span class="xml">
                <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>
                    ![](</span><span class="hljs-template-variable"><span>{</span><span>{</span> im.url <span>}</span><span>}</span></span><span class="xml">)
                <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
            </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name">endthumbnail</span> <span>%</span><span>}</span></span><span class="xml">
        <span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
        <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"info"</span>&gt;</span>
            <span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.get_absolute_url <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span> <span class="hljs-attr">class</span>=<span class="hljs-string">"title"</span>&gt;</span></span><span class="hljs-template-variable"><span>{</span><span>{</span> image.title <span>}</span><span>}</span></span><span class="xml"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span>
        <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endfor</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个模板将用于展示一组图片，它会作为结果返回给 AJAX 请求。之后，在相同路径下创建'list.html'模板，添加以下代码：</p>
<pre class="hljs django"><code class="django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">extends</span></span> "base.html" <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> title <span>%</span><span>}</span></span><span class="xml">Images bookmarked</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml">

</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">block</span></span> content <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span>Images bookmarked<span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span>
    <span class="hljs-tag">&lt;<span class="hljs-name">div</span> <span class="hljs-attr">id</span>=<span class="hljs-string">"image-list"</span>&gt;</span>
        </span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">include</span></span> "images/image/list_ajax.html" <span>%</span><span>}</span></span><span class="xml">
    <span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span>
</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endblock</span></span> <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>这个列表模板继承了'<code>base.html</code>模板。为了避免重复编码，我们引入了<code>list_ajax.html</code>模板。这个<code>listmhtml</code>模板将会有一段 JavaScript 代码来加载当滚动到底部时的额外页面。<br>将以下代码添加进<code>list.html</code>模板中：</p>
<pre class="hljs javascript"><code class="javascript"><span>{</span><span>%</span> block domready <span>%</span><span>}</span>
    <span class="hljs-keyword">var</span> page = <span class="hljs-number">1</span>;
    <span class="hljs-keyword">var</span> empty_page = <span class="hljs-literal">false</span>;
    <span class="hljs-keyword">var</span> block_request = <span class="hljs-literal">false</span>;

    $(<span class="hljs-built_in">window</span>).scroll(<span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params"></span>) </span>{
        <span class="hljs-keyword">var</span> margin = $(<span class="hljs-built_in">document</span>).height() - $(<span class="hljs-built_in">window</span>).height() - <span class="hljs-number">200</span>;
        <span class="hljs-keyword">if</span>  ($(<span class="hljs-built_in">window</span>).scrollTop() &gt; margin &amp;&amp; empty_page == <span class="hljs-literal">false</span> &amp;&amp; block_request == <span class="hljs-literal">false</span>) {
            block_request = <span class="hljs-literal">true</span>;
            page += <span class="hljs-number">1</span>;
            $.get(<span class="hljs-string">'?page='</span> + page, <span class="hljs-function"><span class="hljs-keyword">function</span>(<span class="hljs-params">data</span>) </span>{
                <span class="hljs-keyword">if</span>(data == <span class="hljs-string">''</span>)
                {
                    empty_page = <span class="hljs-literal">true</span>;
                }
                <span class="hljs-keyword">else</span> {
                    block_request = <span class="hljs-literal">false</span>;
                    $(<span class="hljs-string">'#image-list'</span>).append(data);
                }
            });
        }
    });
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre>
<p>这段代码实现了不受限的滚屏功能。我们在 <code>base.html</code>中定义的<code>domready</code>块中引入了 JavaScript 代码，这段代码的工作流程如下：</p>
<ol>
<li>
<p>我们定义了如下几个变量：</p>
<ul>
<li>
<code>page</code>：保存当前的页码</li>
<li>
<code>empt_page</code>:让我们知道用户是否到了最后一页，然后接收一个空页面。只要接收到了一个空页面，我们就会停止发送额外的 AJAX 请求，因为我们确定此时已经没有结果了。</li>
<li>
<code>block_requests</code>：当有进程中有 AJAX 请求时，阻止额外的请求。</li>
</ul>
</li>
<li>我们使用<code>$(window).scroll()</code>来捕获滚动事件，然后我们为此定义了一个控制器函数。</li>
<li>我们计算边框变量来得到文档高度和窗口高度的差值，因为这个差值是用户将要滚动的内容的高度。我们从结果当中减去 200，这样我们就可以在用户接近底部 200pixels 时加载下一页的内容。</li>
<li>我们只在以下两种条件满足时发送 AJAX 请求：没有其他 AJAX 请求被正在被执行时（译者注：就是同时只有一个 AJAX 请求）（<code>block_request</code>必须是<code>false</code>）,用户也没有到达页面底部（<code>empty_page</code>也必须是<code>false</code>）。</li>
<li>我们将<code>block_request</code>设为<code>True</code>来避免滚动时间触发额外的 AJAX 请求，然后我们会在请求下一页时增加一次<code>page</code>计数。</li>
<li>我们使用<code>$.get()</code>来执行一次 AJAX GET 请求，然后我们在一个叫做<code>data</code>的变量中接收 HTML 响应。这里有两种情况。<ul>
<li>响应没有内容：我们已经到了结果的末尾，所以这里没有更多的页面来供我们加载。我们把<code>empty_page</code>设为<code>True</code>来阻止加载更多的 AJAX 请求。</li>
<li>响应含有数据：我们将数据添加到id为 <code>image-list</code>的 HTML 元素中，当用户滚动到底部时页面将直接扩展添加的结果。</li>
</ul>
</li>
</ol>
<p>在浏览器中访问<code>http://127.0.0.1:8000/images/</code>,你会看到你之前添加的一组图片，看起来像这样：<br></p><div class="image-package">
<img src="http://ohqrvqrlb.bkt.clouddn.com/django-5-9.png" data-original-src="http://ohqrvqrlb.bkt.clouddn.com/django-5-9.png" style="cursor: zoom-in;"><br><div class="image-caption">Django-5-9</div>
</div>
<p>滚动到底部将会加载下一页。确定你已经使用书签添加了多于 8 张图片，因为我们每一页展示的是 8 张图片。记得使用 Firebug 或者类似的工具来跟踪 AJAX 请求和调试你的 JavaScript 代码。</p>
<p>最后，编辑 <code>account</code> 应用中的<code>base.html</code>模板，为主菜单添加图片项：</p>
<pre class="hljs django"><code class="django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">li</span> </span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">if</span></span> section == "images" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-attr">class</span>=<span class="hljs-string">"selected"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">endif</span></span> <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span> <span class="hljs-attr">href</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">url</span></span> "images:list" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span>Images<span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span></code></pre>
<p>现在你可以从主菜单连接到图片列表了。</p>
<h2><strong>总结</strong></h2>
<p>在这一章中，我们创建了一个 JavaScript 书签来从其他网站分享图片到我们的网站。你已经用 jQuery 实现了 AJAX 视图，还添加了 AJAX 分页。</p>
<p>在下一章中，将会教你如何创建一个粉丝系统和一个活动流。你将和通用关系、信号、与反规范化打交道。你也将学习如何在 Django  中使用 Redis。</p>
</div>
