---
layout: single
permalink: /django/example7/
title: "Django By Example 第七章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第七章"><strong>第七章</strong></h1>
<h2 id="建立一个在线商店"><strong>建立一个在线商店</strong></h2>
<p>在上一章，你创建了一个用户跟踪系统和建立了一个用户活跃流。你也学习了 Django 信号是如何工作的，并且把 Redis 融合进了项目中来为图像视图计数。在这一章中，你将学会如何建立一个最基本的在线商店。你将会为你的产品创建目录和使用 Django sessions 实现一个购物车。你也将学习怎样定制上下文处理器（ context processors ）以及用 Celery 来激活动态任务。</p>
<p>在这一章中，你将学会：</p>
<ul>
<li>创建一个产品目录</li>
<li>使用 Django sessions 建立购物车</li>
<li>管理顾客的订单</li>
<li>用 Celery 发送异步通知</li>
</ul>
<h2 id="创建一个在线商店项目project"><strong>创建一个在线商店项目（project）</strong></h2>
<p>我们将从新建一个在线商店项目开始。我们的用户可以浏览产品目录并且可以向购物车中添加商品。最后，他们将清点购物车然后下单。这一章涵盖了在线商店的以下几个功能：</p>
<ul>
<li>创建产品目录模型（模型），将它们添加到管理站点，创建基本的视图（view）来展示目录</li>
<li>使用 Django sessions 建立一个购物车系统，使用户可以在浏览网站的过程中保存他们选中的商品</li>
<li>创建下单表单和功能</li>
<li>发送一封异步的确认邮件在用户下单的时候</li>
</ul>
<p>首先，用以下命令来为你的新项目创建一个虚拟环境，然后激活它：</p>
<pre class="shell"><code class="hljs perl"><span class="hljs-keyword">mkdir</span> env
virtualenv env/myshop
source env/myshop/bin/activate</code></pre>
<p>用以下命令在你的虚拟环境中安装 Django :</p>
<pre class="shell"><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django==<span class="hljs-number">1.0</span>.<span class="hljs-number">6</span></code></pre>
<p>创建一个叫做 <code>myshop</code> 的新项目，再创建一个叫做 <code>shop</code> 的应用，命令如下：</p>
<pre class="shell"><code class="hljs dos">django-admin startproject myshop
<span class="hljs-built_in">cd</span> myshop
django-admin startapp shop</code></pre>
<p>编辑你项目中的 <code>settings.py</code> 文件，像下面这样将你的应用添加到 <code>INSTALLED_APPS</code> 中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'shop'</span></span>,
]</code></pre></div>
<p>现在你的应用已经在项目中激活。接下来让我们为产品目录定义模型（models）。</p>
<h2 id="创建产品目录模型models"><strong>创建产品目录模型（models）</strong></h2>
<p>我们商店中的目录将会由不同分类的产品组成。每一个产品会有一个名字，一段可选的描述，一张可选的图片，价格，以及库存。 编辑位于<code>shop</code>应用中的<code>models.py</code>文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Category</span><span class="hljs-params">(models.Model)</span>:</span>
     name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                                  db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
     slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                            db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>,
                                   unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
     <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
          ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'name'</span></span>,)
          verbose_name <span class="op">=</span> <span class="st"><span class="hljs-string">'category'</span></span>
          verbose_name_plural <span class="op">=</span> <span class="st"><span class="hljs-string">'categories'</span></span>
 
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.name
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Product</span><span class="hljs-params">(models.Model)</span>:</span>
    category <span class="op">=</span> models.ForeignKey(Category, 
                                 related_name<span class="op">=</span><span class="st"><span class="hljs-string">'products'</span></span>)
    name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    image <span class="op">=</span> models.ImageField(upload_to<span class="op">=</span><span class="st"><span class="hljs-string">'products/%Y/%m/</span></span><span class="sc"><span class="hljs-string">%d</span></span><span class="st"><span class="hljs-string">'</span></span>,
                              blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    price <span class="op">=</span> models.DecimalField(max_digits<span class="op">=</span><span class="dv"><span class="hljs-number">10</span></span>, decimal_places<span class="op">=</span><span class="dv"><span class="hljs-number">2</span></span>)
    stock <span class="op">=</span> models.PositiveIntegerField()
    available <span class="op">=</span> models.BooleanField(default<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)

    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'name'</span></span>,)
        index_together <span class="op">=</span> ((<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>),)

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.name</code></pre></div>
<p>这是我们的 <code>Category</code> 和 <code>Product</code> 模型（models）。<code>Category</code> 模型（models）由一个 <code>name</code> 字段和一个唯一的 <code>slug</code> 字段构成。<code>Product</code> 模型（model）：</p>
<ul>
<li><code>category</code>: 这是一个链接向 <code>Category</code> 的 <code>ForeignKey</code> 。这是个多对一（many-to-one）关系。一个产品可以属于一个分类，一个分类也可包含多个产品。</li>
<li><code>name</code>: 这是产品的名字</li>
<li><code>slug</code>: 用来为这个产品建立 URL 的 slug</li>
<li><code>image</code>: 可选的产品图片</li>
<li><code>description</code>: 可选的产品描述</li>
<li><code>price</code>: 这是个 <code>DecimalField</code><strong>（<a href="mailto:%E8%AF%91%E8%80%85@ucag注">译者@ucag注</a>：十进制字段）</strong>。这个字段使用 Python 的 <code>decimal.Decimal</code> 元类来保存一个固定精度的十进制数。<code>max_digits</code> 属性可用于设定数字的最大值， <code>decimal_places</code> 属性用于设置小数位数。</li>
<li><code>stock</code>: 这是个 <code>PositiveIntegerField</code><strong>（<a href="mailto:%E8%AF%91%E8%80%85@ucag注">译者@ucag注</a>：正整数字段）</strong> 来保存这个产品的库存。</li>
<li><code>available</code>: 这个布尔值用于展示产品是否可供购买。这使得我们可在目录中使产品废弃或生效。</li>
<li><code>created</code>: 当对象被创建时这个字段被保存。</li>
<li><code>update</code>: 当对象最后一次被更新时这个字段被保存。</li>
</ul>
<p>对于 <code>price</code> 字段，我们使用 <code>DecimalField</code> 而不是 <code>FloatField</code> 来避免精度问题。</p>
<blockquote>
<p>我们总是使用 <code>DecimalField</code> 来保存货币值。 <code>FloatField</code> 在内部使用 Python 的 <code>float</code> 类型。反之， <code>DecimalField</code> 使用的是 Python 中的 <code>Decimal</code> 类型，使用 <code>Decimal</code> 类型可以避免精度问题。</p>
</blockquote>
<p>在 <code>Product</code> 模型（model）中的 <code>Meta</code> 类中，我们使用 <code>index_together</code> 元选项来指定 <code>id</code> 和 <code>slug</code> 字段的共同索引。我们定义这个索引，因为我们准备使用这两个字段来查询产品，两个字段被索引在一起来提高使用双字段查询的效率。</p>
<p>由于我们会在模型（models）中和图片打交道，打开 shell ，用下面的命令安装 Pillow ：</p>
<pre class="shell"><code class="hljs nginx"><span class="hljs-attribute">pip</span> isntall Pillow==<span class="hljs-number">2</span>.<span class="hljs-number">9</span>.<span class="hljs-number">0</span></code></pre>
<p>现在，运行下面的命令来为你的项目创建初始迁移：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你将会看到以下输出：</p>
<pre class="shell"><code class="hljs sql">Migrations for 'shop':
    0001_initial.py:
      - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-keyword">Category</span>
      - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> Product
      - <span class="hljs-keyword">Alter</span> index_together <span class="hljs-keyword">for</span> product (<span class="hljs-number">1</span> <span class="hljs-keyword">constraint</span>(s))</code></pre>
<p>用下面的命令来同步你的数据库：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">mange</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你将会看到包含下面这一行的输出：</p>
<pre class="shell"><code class="hljs css"> <span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">shop</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>现在数据库已经和你的模型（models）同步了。</p>
<h2 id="注册目录模型models到管理站点"><strong>注册目录模型（models）到管理站点</strong></h2>
<p>让我们把模型（models）注册到管理站点，这样我们就可以轻松管理产品和产品分类了。编辑 <code>shop</code> 应用的 <code>admin.py</code> 文件，添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Category, Product

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CategoryAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>]
    prepopulated_fields <span class="op">=</span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'name'</span></span>,)}
admin.site.register(Category, CategoryAdmin)

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ProductAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'price'</span></span>, <span class="st"><span class="hljs-string">'stock'</span></span>, 
                    <span class="co"><span class="hljs-string">'available'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>]
    list_filter <span class="op">=</span> [<span class="st"><span class="hljs-string">'available'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>]
    list_editable <span class="op">=</span> [<span class="st"><span class="hljs-string">'price'</span></span>, <span class="st"><span class="hljs-string">'stock'</span></span>, <span class="st"><span class="hljs-string">'available'</span></span>]
    prepopulated_fields <span class="op">=</span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'name'</span></span>,)}
admin.site.register(Product, ProductAdmin)</code></pre></div>
<p>记住，我们使用 <code>prepopulated_fields</code> 属性来指定那些要使用其他字段来自动赋值的字段。正如你以前看到的那样，这样做可以很方便的生成 slugs 。我们在 <code>ProductAdmin</code> 类中使用 <code>list_editable</code> 属性来设置可被编辑的字段，并且这些字段都在管理站点的列表页被列出。这样可以让你一次编辑多行。任何在 <code>list_editable</code> 的字段也必须在 <code>list_display</code> 中，因为只有这样被展示的字段才可以被编辑。</p>
<p>现在，使用如下命令为你的站点创建一个超级用户：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">createsuperuser</span></code></pre>
<p>使用命令 <code>python manage.py runserver</code> 启动开发服务器。 访问 <a href="http://127.0.0.1:8000/admin/shop/product/add" class="uri">http://127.0.0.1:8000/admin/shop/product/add</a> ,登录你刚才创建的超级用户。在管理站点的交互界面添加一个新的品种和产品。 product 的更改页面如下所示：</p>
<p><iframe id="iframe_0.12310489122245527" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-c416ba5ed3c94b29.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.12310489122245527',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 622px; height: 175px;" frameborder="0" scrolling="no"></iframe></p>
<h2 id="创建目录视图views"><strong>创建目录视图（views）</strong></h2>
<p>为了展示产品目录， 我们需要创建一个视图（view）来列出所有产品或者是给出的筛选后的产品。编辑 <code>shop</code> 应用中的 <code>views.py</code> 文件，添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Category, Product

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">product_list</span><span class="hljs-params">(request, category_slug</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    category <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    categories <span class="op">=</span> Category.objects.<span class="bu">all</span>()
    products <span class="op">=</span> Product.objects.<span class="bu">filter</span>(available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="cf"><span class="hljs-keyword">if</span></span> category_slug:
        category <span class="op">=</span> get_object_or_404(Category, slug<span class="op">=</span>category_slug)
        products <span class="op">=</span> products.<span class="bu">filter</span>(category<span class="op">=</span>category)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, 
                  <span class="st"><span class="hljs-string">'shop/product/list.html'</span></span>, 
                  {<span class="st"><span class="hljs-string">'category'</span></span>: category,
                  <span class="co"><span class="hljs-string">'categories'</span></span>: categories,
                  <span class="co"><span class="hljs-string">'products'</span></span>: products})</code></pre></div>
<p>我们只筛选 <code>available=True</code> 的查询集来检索可用的产品。我们使用一个可选参数 <code>category_slug</code> 通过所给产品类别来有选择性的筛选产品。</p>
<p>我们也需要一个视图来检索和展示单一的产品。把下面的代码添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">product_detail</span><span class="hljs-params">(request, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="hljs-function"><span class="hljs-params">, slug)</span>:</span>
    product <span class="op">=</span> get_object_or_404(Product, 
                                <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>, 
                                slug<span class="op">=</span>slug, 
                                available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, 
                  <span class="st"><span class="hljs-string">'shop/product/detail.html'</span></span>, 
                  {<span class="st"><span class="hljs-string">'product'</span></span>: product})</code></pre></div>
<p><code>product_detail</code> 视图（view）接收 <code>id</code> 和 <code>slug</code> 参数来检索 <code>Product</code> 实例。我们可以只用 ID 就可以得到这个实例，因为它是一个独一无二的属性。尽管，我们在 URL 中引入了 slug 来建立搜索引擎友好（SEO-friendly）的 URL。</p>
<p>在创建了产品列表和明细视图（views）之后，我们该为它们定义 URL 模式了。在 <code>shop</code> 应用的路径下创建一个新的文件，命名为 <code>urls.py</code> ，然后添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, views.product_list, name<span class="op">=</span><span class="st"><span class="hljs-string">'product_list'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;category_slug&gt;[-\w]+)/$'</span></span>, 
        views.product_list, 
        name<span class="op">=</span><span class="st"><span class="hljs-string">'product_list_by_category'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^(?P&lt;id&gt;\d+)/(?P&lt;slug&gt;[-\w]+)/$'</span></span>, 
        views.product_detail, 
        name<span class="op">=</span><span class="st"><span class="hljs-string">'product_detail'</span></span>),</code></pre></div>
<p>这些是我们产品目录的URL模式。 我们为 <code>product_list</code> 视图（view）定义了两个不同的 URL 模式。 命名为<code>product_list</code> 的模式不带参数调用 <code>product_list</code> 视图（view）；命名为 <code>product_list_bu_category</code> 的模式向视图（view）函数传递一个 <code>category_slug</code> 参数，以便通过给定的产品种类来筛选产品。我们为 <code>product_detail</code> 视图（view）添加的模式传递了 <code>id</code> 和 <code>slug</code> 参数来检索特定的产品。</p>
<p>像这样编辑 <code>myshop</code> 项目中的 <code>urls.py</code> 文件：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url, include
<span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
    url(<span class="vs"><span class="hljs-string">r'^'</span></span>, include(<span class="st"><span class="hljs-string">'shop.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'shop'</span></span>)),
    ]</code></pre></div>
<p>在项目的主要 URL 模式中，我们引入了 <code>shop</code> 应用的 URL 模式，并指定了一个命名空间，叫做 <code>shop</code>。</p>
<p>现在，编辑 <code>shop</code> 应用中的 <code>models.py</code> 文件，导入 <code>reverse()</code> 函数，然后给 <code>Category</code> 模型和 <code>Product</code> 模型添加 <code>get_absolute_url()</code> 方法：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse
<span class="co"><span class="hljs-comment"># ...</span></span>
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Category</span><span class="hljs-params">(models.Model)</span>:</span>
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_absolute_url</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> reverse(<span class="st"><span class="hljs-string">'shop:product_list_by_category'</span></span>,
                        args<span class="op">=</span>[<span class="va">self</span>.slug])
                        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Product</span><span class="hljs-params">(models.Model)</span>:</span>
<span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_absolute_url</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> reverse(<span class="st"><span class="hljs-string">'shop:product_detail'</span></span>,
            args<span class="op">=</span>[<span class="va">self</span>.<span class="bu">id</span>, <span class="va">self</span>.slug])</code></pre></div>
<p>正如你已经知道的那样， <code>get_absolute_url()</code> 是检索一个对象的 URL 约定俗成的方法。这里，我们将使用我们刚刚在 <code>urls.py</code> 文件中定义的 URL 模式。</p>
<h1 id="创建目录模板templates"><strong>创建目录模板（templates）</strong></h1>
<p>现在，我们需要为产品列表和明细视图创建模板（templates）。在 <code>shop</code> 应用的路径下创建如下路径和文件：</p>
<pre class="shell"><code class="hljs cpp">templates/
    shop/
        base.html
        product/
            <span class="hljs-built_in">list</span>.html
            detail.html</code></pre>
<p>我们需要定义一个基础模板（template），然后在产品列表和明细模板（templates）中继承它。 编辑 <code>shop/base.html</code> 模板（template），添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load static <span>%</span><span>}</span>
<span class="dt"><span class="hljs-meta">&lt;!DOCTYPE </span></span><span class="hljs-meta">html</span><span class="dt"><span class="hljs-meta">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">charset</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"utf-8"</span></span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span><span>{</span><span>%</span> block title <span>%</span><span>}</span>My shop<span>{</span><span>%</span> endblock <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">link</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">css</span>/<span class="hljs-attr">base.css</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">rel</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"stylesheet"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"header"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"logo"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>My shop<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"subheader"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            Your cart is empty.
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"content"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span>{</span><span>%</span> block content <span>%</span><span>}</span>
        <span>{</span><span>%</span> endblock <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre></div>
<p>这就是我们将为我们的商店应用使用的基础模板（template）。为了引入模板使用的 CSS 和图像，你需要复制这一章示例代码中的静态文件，位于 <code>shop</code> 应用中的 <code>static/</code> 路径下。把它们复制到你的项目中相同的地方。</p>
<p>编辑 <code>shop/product/list.html</code> 模板（template），然后添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> load static <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    <span>{</span><span>%</span> if category <span>%</span><span>}</span><span>{</span><span>{</span> category.name <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span>Products<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"sidebar"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Categories<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">not</span> <span class="hljs-attr">category</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span class="hljs-attr">class</span></span></span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">shop:product_list"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>All<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
        <span>{</span><span>%</span> for c in categories <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">category.slug</span></span></span><span class="hljs-tag"> </span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">=</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">c.slug</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span class="hljs-attr">class</span></span></span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> c.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> c.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"main"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"product-list"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span>{</span><span>%</span> if category <span>%</span><span>}</span><span>{</span><span>{</span> category.name <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span>Products<span>{</span><span>%</span> endif <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
        <span>{</span><span>%</span> for product in products <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"item"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> product.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                    ![](<span>{</span><span>%</span> if product.image <span>%</span><span>}</span><span>{</span><span>{</span> product.image.url <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span><span>{</span><span>%</span> static )
                <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> product.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> product.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span></span>
                $<span>{</span><span>{</span> product.price <span>}</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span> </code></pre></div>
<p>这是产品列表模板（template）。它继承了 <code>shop/base.html</code> 并且使用了 <code>categories</code> 上下文变量来展示所有在侧边栏里的产品种类，以及 <code>products</code> 上下文变量来展示当前页面的产品。相同的模板用于展示所有的可用的产品以及经目录分类筛选后的产品。由于<code>Product</code> 模型的 <code>image</code> 字段可以为空，我们需要为没有图片的产品提供一个默认图像。这个图片位于我们的静态文件路径下，相对路径为 <code>img/no_image.png</code>。</p>
<p>因为我们在使用 <code>ImageField</code> 来保存产品图片，我们需要开发服务器来服务上传图片文件。编辑 <code>myshop</code> 项目的 <code>settings.py</code> 文件，添加以下设置：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">MEDIA_URL <span class="op">=</span> <span class="st"><span class="hljs-string">'/media/'</span></span>
MEDIA_ROOT <span class="op">=</span> os.path.join(BASE_DIR, <span class="st"><span class="hljs-string">'media/'</span></span>)</code></pre></div>
<p><code>MEDIA_URL</code> 是基础 URL，它为用户上传的媒体文件提供服务。<code>MEDIA_ROOT</code> 是一个本地路径，媒体文件就在这个路径下，并且是由我们动态的将 <code>BASE_DIR</code> 添加到它的前面而得到的。</p>
<p>为了让 Django 给通过开发服务器上传的媒体文件提供服务，编辑<code>myshop</code> 中的 <code>urls.py</code> 文件，添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls.static <span class="im"><span class="hljs-keyword">import</span></span> static

urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># ...</span></span>
]
<span class="cf"><span class="hljs-keyword">if</span></span> settings.DEBUG:
    urlpatterns <span class="op">+=</span> static(settings.MEDIA_URL,
                          document_root<span class="op">=</span>settings.MEDIA_ROOT)</code></pre></div>
<p>记住，我们仅仅在开发中像这样提供静态文件服务。在生产环境下，你不应该用 Django 来服务静态文件。</p>
<p>使用管理站点为你的商店添加几个产品，然后访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> 。你可以看到如下的产品列表页：</p>
<p><iframe id="iframe_0.13401176977127172" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-4efe4640dbcd062b.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.13401176977127172',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 615px; height: 251px;" frameborder="0" scrolling="no"></iframe></p>
<p>如果你用管理站点创建了几个产品，并且没有上传任何图片的话，就会显示默认的 <code>no_img.png</code> 。</p>
<p><iframe id="iframe_0.8436912788496373" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-0ca87c08d70489f9.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.8436912788496373',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 613px; height: 230px;" frameborder="0" scrolling="no"></iframe></p>
<p>让我们编辑产品明细模板（template）。 编辑 <code>shop/product/detail.html</code> 模板（template），添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> load static <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    <span>{</span><span>%</span> if category <span>%</span><span>}</span><span>{</span><span>{</span> category.title <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span>Products<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"product-detail"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        ![](<span>{</span><span>%</span> if product.image <span>%</span><span>}</span><span>{</span><span>{</span> product.image.url <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span><span>{</span><span>%</span> static )
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span>{</span><span>{</span> product.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> product.category.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> product.category <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"price"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> product.price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
            <span>{</span><span>{</span> product.description|linebreaks <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>我们可以调用相关联的产品类别的 <code>get_absolute_url()</code> 方法来展示有效的属于同一目录的产品。现在，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000</a> ，点击任意产品，查看产品明细页面。看起来像这样：</p>
<p><iframe id="iframe_0.33195157533492536" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-a65586d8380f35aa.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.33195157533492536',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 625px; height: 272px;" frameborder="0" scrolling="no"></iframe></p>
<h2 id="创建购物车"><strong>创建购物车</strong></h2>
<p>在创建了产品目录之后，下一步我们要创建一个购物车系统，这个购物车系统可以让用户选中他们想买的商品。购物车允许用户在最终下单之前选中他们想要的物品并且可以在用户浏览网站时暂时保存它们。购物车存在于会话中，所以购物车中的物品会在用户访问期间被保存。</p>
<p>我们将会使用 Django 的会话框架（seesion framework）来保存购物车。在购物车最终被完成或用户下单之前，购物车将会保存在会话中。我们需要为购物车和购物车里的商品创建额外的 Django 模型（models）。</p>
<h2 id="使用-django-会话"><strong>使用 Django 会话</strong></h2>
<p>Django 提供了一个会话框架，这个框架支持匿名会话和用户会话。会话框架允许你为任意访问对象保存任何数据。会话数据保存在服务端，并且如果你使用基于 cookies 的会话引擎的话， cookies 会包含 session ID 。会话中间件控制发送和接收 cookies 。默认的会话引擎把会话保存在数据库中，但是正如你一会儿会看到的那样，你也可以选择不同的会话引擎。为了使用会话，你必须确认你项目的 <code>MIDDLEWARE_CLASSES</code> 设置中包含了 <code>django.contrib.sessions.middleware.SessionMiddleware</code> 。这个中间件负责控制会话，并且是在你使用命令<code>startproject</code>创建项目时被默认添加的。</p>
<p>会话中间件使当前会话在 <code>request</code> 对象中可用。你可以用 <code>request.seesion</code> 连接当前会话，它的使用方式和 Python 的字典相似。会话字典接收任何默认的可被序列化为 JSON 的 Python 对象。你可以在会话中像这样设置变量：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">request.session[<span class="st"><span class="hljs-string">'foo'</span></span>] <span class="op">=</span> <span class="st"><span class="hljs-string">'bar'</span></span></code></pre></div>
<p>检索会话中的键:</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">request.session.get(<span class="st"><span class="hljs-string">'foo'</span></span>)</code></pre></div>
<p>删除会话中已有键：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-keyword">del</span></span> request.session[<span class="st"><span class="hljs-string">'foo'</span></span>]</code></pre></div>
<p>正如你所见，我们像使用 Python 字典一样使用 <code>request.session</code> 。</p>
<blockquote>
<p>当用户登录时，他们的匿名会话将会丢失，然后新的会话将会为认证后的用户创建。如果你在匿名会话中储存了在登录后依然需要被持有的数据，你需要从旧的会话中复制数据到新的会话。</p>
</blockquote>
<h2 id="会话设置"><strong>会话设置</strong></h2>
<p>你可以使用几种设置来为你的项目配置会话系统。最重要的部分是 <code>SESSION_ENGINE</code> .这个设置让你可以配置会话将会在哪里被储存。默认地， Django 用 <code>django.contrib.sessions</code> 的 <code>Sessions</code> 模型把会话保存在数据库中。</p>
<p>Django 提供了以下几个选择来保存会话数据：</p>
<ul>
<li>Database sessions（数据库会话）:会话数据将会被保存在数据库中。这是默认的会话引擎。</li>
<li>File-based sessions（基于文件的会话）：会话数据保存在文件系统中。</li>
<li>Cached sessions（缓存会话）:会话数据保存在缓存后端中。你可以使用 <code>CACHES</code> 设置来指定一个缓存后端。在缓存系统中保存会话拥有最好的性能。</li>
<li>Cached sessions（缓存会话）：会话数据储存于缓存后端。你可以使用 <code>CACHES</code> 设置来制定一个缓存后端。在缓存系统中储存会话数据会有更好的性能表现。</li>
<li>Cached database sessions（缓存于数据库中的会话）：会话数据保存于可高速写入的缓存和数据库中。只会在缓存中没有数据时才会从数据库中读取数据。</li>
<li>Cookie-based sessions（基于 cookie 的会话）：会话数据储存于发送向浏览器的 cookie 中。</li>
</ul>
<blockquote>
<p>为了得到更好的性能，使用基于缓存的会话引擎（ cach-based session engine）吧。 Django 支持 Mercached ，以及 Redis 的第三方缓存后端和其他的缓存系统。</p>
</blockquote>
<p>你可以用其他的设置来定制你的会话。这里有一些和会话有关的重要设置：</p>
<ul>
<li><code>SESSION_COOKIE_AGE</code>：cookie 会话保持的时间。以秒为单位。默认值为 1209600 （2 周）。</li>
<li><code>SESSION_COOKIE_DOMAIN</code>：这是为会话 cookie 使用的域名。把它的值设置为 <code>.mydomain.com</code> 来使跨域名 cookie 生效。</li>
<li><code>SESSION_COOKIE_SECURE</code>：这是一个布尔值。它表示只有在连接为 HTTPS 时 cookie 才会被发送。</li>
<li><code>SESSION_EXPIRE_AT_BROWSER_CLOSE</code>：这是一个布尔值。它表示会话会在浏览器关闭时就过期。</li>
<li><code>SESSION_SAVE_EVERY_REQUEST</code>：这是一个布尔值。如果为 <code>True</code> ，每一次请求的 session 都将会被储存进数据库中。 session 的过期时间也会每次刷新。</li>
</ul>
<p>在这个网站你可以看到所有的 session 设置：<a href="https://docs.djangoproject.com/en/1.8/ref/settings/#sessions" class="uri">https://docs.djangoproject.com/en/1.8/ref/settings/#sessions</a></p>
<h2 id="会话过期"><strong>会话过期</strong></h2>
<p>你可以通过 <code>SESSION_EXPIRE_AT_BROWSER_CLOSE</code> 选择使用 browser-length 会话或者持久会话。默认的设置是 <code>False</code> ，强制把会话的有效期设置为 <code>SESSION_COOKIE_AGE</code> 的值。如果你把 <code>SESSION_EXPIRE_AT_BROWSER_CLOSE</code> 的值设为 <code>True</code>，会话将会在用户关闭浏览器时过期，且 <code>SESSION_COOKIE_AGE</code> 将不会对此有任何影响。</p>
<p>你可以使用 <code>request.session</code> 的 <code>set_expiry()</code> 方法来覆写当前会话的有效期。</p>
<h2 id="在会话中保存购物车"><strong>在会话中保存购物车</strong></h2>
<p>我们需要创建一个能序列化为 JSON 的简单结构，这样就可以把购物车中的东西储存在会话中。购物车必须包含以下数据，每个物品的数据都要包含在其中：</p>
<ul>
<li><code>Product</code> 实例的 <code>id</code></li>
<li>选择的产品数量</li>
<li>产品的总价格</li>
</ul>
<p>因为产品的价格可能会变化，我们采取当产品被添加进购物车时同时保存产品价格和产品本身的办法。这样做，我们就可以保持用户在把商品添加进购物车时他们看到的商品价格不变了，即使产品的价格在之后有了变更。</p>
<p>现在，你需要把购物车和会话关联起来。购物车像下面这样工作：</p>
<ul>
<li>当需要一个购物车时，我们检查顾客是否已经设置了一个会话键（ session key）。如果会话中没有购物车，我们就创建一个新的购物车，然后把它保存在购物车的会话键中。</li>
<li>对于连续的请求，我们在会话键中执行相同的检查和获取购物车内物品的操作。我们在会话中检索购物车的物品和他们在数据库中相关联的 <code>Product</code> 对象。</li>
</ul>
<p>编辑你的项目中 <code>settings.py</code>，把以下设置添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">CART_SESSION_ID <span class="op">=</span> <span class="st"><span class="hljs-string">'cart'</span></span></code></pre></div>
<p>添加的这个键将会用于我们的会话中来储存购物车。因为 Django 的会话对于每个访问者是独立的<strong>（<a href="mailto:%E8%AF%91%E8%80%85@ucag注">译者@ucag注</a>：原文为 per-visitor ，没能想出一个和它对应的中文词，根据上下文，我就把这个词翻译为了一个短语）</strong>，我们可以在所有的会话中使用相同的会话键。</p>
<p>让我们创建一个应用来管理我们的购物车。打开终端，然后创建一个新的应用，在项目路径下运行以下命令：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">cart</span></code></pre>
<p>然后编辑你添加的项目中的 <code>settings.py</code> ，在 <code>INSTALLED_APPS</code> 中添加 <code>cart</code>：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'shop'</span></span>,
    <span class="co"><span class="hljs-string">'cart'</span></span>,
)</code></pre></div>
<p>在 <code>cart</code> 应用路径内创建一个新的文件，命名为 <code>cart.py</code> ，把以下代码添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> decimal <span class="im"><span class="hljs-keyword">import</span></span> Decimal
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> shop.models <span class="im"><span class="hljs-keyword">import</span></span> Product

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Cart</span><span class="hljs-params">(</span></span><span class="bu"><span class="hljs-class"><span class="hljs-params">object</span></span></span><span class="hljs-class"><span class="hljs-params">)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__init__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request)</span>:</span>
        <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">        Initialize the cart.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">        """</span></span>
        <span class="va">self</span>.session <span class="op">=</span> request.session
        cart <span class="op">=</span> <span class="va">self</span>.session.get(settings.CART_SESSION_ID)
        <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> cart:
            <span class="co"><span class="hljs-comment"># save an empty cart in the session</span></span>
            cart <span class="op">=</span> <span class="va">self</span>.session[settings.CART_SESSION_ID] <span class="op">=</span> {}
        <span class="va">self</span>.cart <span class="op">=</span> cart</code></pre></div>
<p>这个 <code>Cart</code> 类可以让我们管理购物车。我们需要把购物车与一个 <code>request</code> 对象一同初始化。我们使用 <code>self.session = request.session</code> 保存当前会话以便使其对 <code>Cart</code>类的其他方法可用。首先，我们使用 <code>self.session.get(settings.CART_SESSION_ID)</code> 尝试从当前会话中获取购物车。如果当前会话中没有购物车，我们就在会话中设置一个空字典，这样就可以在会话中设置一个空的购物车。我们希望我们的购物车字典使用产品 ID 作为键，以数量和价格为键值对的字典为值。这样做，我们就能保证一个产品在购物车当中不被重复添加；我们也能简化获取任意购物车物品数据的步骤。</p>
<p>让我们写一个方法来向购物车当中添加产品或者更新产品的数量。把 <code>save()</code> 和 <code>add()</code> 方法添加进 <code>Cart</code> 类当中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">add</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, product, quantity</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="dv"><span class="hljs-function"><span class="hljs-params"><span class="hljs-number">1</span></span></span></span><span class="hljs-function"><span class="hljs-params">, update_quantity</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">False</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    Add a product to the cart or update its quantity.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    """</span></span>
    product_id <span class="op">=</span> <span class="bu">str</span>(product.<span class="bu">id</span>)
    <span class="cf"><span class="hljs-keyword">if</span></span> product_id <span class="op"><span class="hljs-keyword">not</span></span> <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.cart:
        <span class="va">self</span>.cart[product_id] <span class="op">=</span> {<span class="st"><span class="hljs-string">'quantity'</span></span>: <span class="dv"><span class="hljs-number">0</span></span>,
                                 <span class="co"><span class="hljs-string">'price'</span></span>: <span class="bu">str</span>(product.price)}
    <span class="cf"><span class="hljs-keyword">if</span></span> update_quantity:
        <span class="va">self</span>.cart[product_id][<span class="st"><span class="hljs-string">'quantity'</span></span>] <span class="op">=</span> quantity
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        <span class="va">self</span>.cart[product_id][<span class="st"><span class="hljs-string">'quantity'</span></span>] <span class="op">+=</span> quantity
    <span class="va">self</span>.save()
    
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">save</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="co"><span class="hljs-comment"># update the session cart</span></span>
    <span class="va">self</span>.session[settings.CART_SESSION_ID] <span class="op">=</span> <span class="va">self</span>.cart
    <span class="co"><span class="hljs-comment"># mark the session as "modified" to make sure it is saved</span></span>
    <span class="va">self</span>.session.modified <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span></code></pre></div>
<p><code>add()</code> 函数接受以下参数：</p>
<ul>
<li><code>product</code>：需要在购物车中更新或者向购物车添加的 <code>Product</code> 对象</li>
<li><code>quantity</code>：一个产品数量的可选参数。默认为 1</li>
<li><code>update_quantity</code>：这是一个布尔值，它表示数量是否需要按照给定的数量参数更新（<code>True</code>），不然新的数量必须要被加进已存在的数量中（<code>False</code>）</li>
</ul>
<p>我们在购物车字典中把产品 <code>id</code> 作为键。我们把产品 <code>id</code> 转换为字符串，因为 Django 使用 JSON 来序列化会话数据，而 JSON 又只接受支字符串的键名。产品 <code>id</code> 为键，一个有 <code>quantity</code> 和 <code>price</code> 的字典作为值。产品的价格从十进制数转换为了字符串，这样才能将它序列化。最后，我们调用 <code>save()</code> 方法把购物车保存到会话中。</p>
<p><code>save()</code> 方法会把购物车中所有的改动都保存到会话中，然后用 <code>session.modified = True</code> 标记改动了的会话。这是为了告诉 Django 会话已经被改动，需要将它保存起来。</p>
<p>我们也需要一个方法来从购物车当中删除购物车。把下面的方法添加进 <code>Cart</code> 类当中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">remove</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, product)</span>:</span>
    <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    Remove a product from the cart.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    """</span></span>
    product_id <span class="op">=</span> <span class="bu">str</span>(product.<span class="bu">id</span>)
    <span class="cf"><span class="hljs-keyword">if</span></span> product_id <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.cart:
        <span class="kw"><span class="hljs-keyword">del</span></span> <span class="va">self</span>.cart[product_id]
        <span class="va">self</span>.save()</code></pre></div>
<p><code>remove</code> 方法从购物车字典中删除给定的产品，然后调用 <code>save()</code> 方法来更新会话中的购物车。</p>
<p>我们将迭代购物车当中的物品，然后获取相应的 <code>Product</code> 实例。为恶劣达到我们的目的，你需要定义 <code>__iter__()</code> 方法。把下列代码添加进 <code>Cart</code> 类中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__iter__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    Iterate over the items in the cart and get the products</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    from the database.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    """</span></span>
    product_ids <span class="op">=</span> <span class="va">self</span>.cart.keys()
    <span class="co"><span class="hljs-comment"># get the product objects and add them to the cart</span></span>
    products <span class="op">=</span> Product.objects.<span class="bu">filter</span>(id__in<span class="op">=</span>product_ids)
    <span class="cf"><span class="hljs-keyword">for</span></span> product <span class="op"><span class="hljs-keyword">in</span></span> products:
        <span class="va">self</span>.cart[<span class="bu">str</span>(product.<span class="bu">id</span>)][<span class="st"><span class="hljs-string">'product'</span></span>] <span class="op">=</span> product
        
    <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.cart.values():
        item[<span class="st"><span class="hljs-string">'price'</span></span>] <span class="op">=</span> Decimal(item[<span class="st"><span class="hljs-string">'price'</span></span>])
        item[<span class="st"><span class="hljs-string">'total_price'</span></span>] <span class="op">=</span> item[<span class="st"><span class="hljs-string">'price'</span></span>] <span class="op">*</span> item[<span class="st"><span class="hljs-string">'quantity'</span></span>]
        <span class="cf"><span class="hljs-keyword">yield</span></span> item</code></pre></div>
<p>在 <code>__iter__()</code> 方法中，我们检索购物车中的 <code>Product</code> 实例来把他们添加进购物车的物品中。之后，我们迭代所有的购物车物品，把他们的 <code>price</code> 转换回十进制数，然后为每个添加一个 <code>total_price</code> 属性。现在我们就可以很容易的在购物车当中迭代物品了。</p>
<p>我们还需要一个方法来返回购物车中物品的总数量。当 <code>len()</code> 方法在一个对象上执行时，Python 会调用对象的 <code>__len__()</code> 方法来检索它的长度。我们将会定义一个定制的 <code>__len__()</code> 方法来返回保存在购物车中保存的所有物品数量。把下面这个 <code>__len__()</code> 方法添加进 <code>Cart</code> 类中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__len__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    Count all items in the cart.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    """</span></span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">sum</span>(item[<span class="st"><span class="hljs-string">'quantity'</span></span>] <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.cart.values())</code></pre></div>
<p>我们返回所有购物车物品的数量。</p>
<p>添加下列方法来计算购物车中物品的总价：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_total_price</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">sum</span>(Decimal(item[<span class="st"><span class="hljs-string">'price'</span></span>]) <span class="op">*</span> item[<span class="st"><span class="hljs-string">'quantity'</span></span>] <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.cart.values())</code></pre></div>
<p>最后，添加一个方法来清空购物车会话：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">clear</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="co"><span class="hljs-comment"># remove cart from session</span></span>
    <span class="kw"><span class="hljs-keyword">del</span></span> <span class="va">self</span>.session[settings.CART_SESSION_ID]
        <span class="va">self</span>.session.modified <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span></code></pre></div>
<p>我们的 <code>Cart</code> 类现在已经准备好管理购物车了。</p>
<h1 id="创建购物车视图"><strong>创建购物车视图</strong></h1>
<p>既然我们已经创建了 <code>Cart</code> 类来管理购物车，我们就需要创建添加，更新，或者删除物品的视图了。我们需要创建以下视图：</p>
<ul>
<li>用于添加或者更新物品的视图，且能够控制当前的和更新的数量</li>
<li>从购物车中删除物品的视图</li>
<li>展示购物车物品和总数的视图</li>
</ul>
<h2 id="添加物品"><strong>添加物品</strong></h2>
<p>为了把物品添加进购物车，我们需要一个允许用户选择数量的表单。在 <code>cart</code> 应用路径下创建一个 <code>forms.py</code> 文件，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms

PRODUCT_QUANTITY_CHOICES <span class="op">=</span> [(i, <span class="bu">str</span>(i)) <span class="cf"><span class="hljs-keyword">for</span></span> i <span class="op"><span class="hljs-keyword">in</span></span> <span class="bu">range</span>(<span class="dv"><span class="hljs-number">1</span></span>, <span class="dv"><span class="hljs-number">21</span></span>)]

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CartAddProductForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    quantity <span class="op">=</span> forms.TypedChoiceField(
                choices<span class="op">=</span>PRODUCT_QUANTITY_CHOICES,
                <span class="bu">coerce</span><span class="op">=</span><span class="bu">int</span>)
    update <span class="op">=</span> forms.BooleanField(required<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>,
                initial<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>,
                widget<span class="op">=</span>forms.HiddenInput)</code></pre></div>
<p>我们将要使用这个表单来向购物车添加产品。我们的 <code>CartAddProductForm</code> 类包含以下两个字段：</p>
<ul>
<li><code>quantity</code>：让用户可以在 1~20 之间选择产品的数量。我们使用了带有 <code>coerce=int</code> 的 <code>TypeChoiceField</code> 字段来把输入转换为整数</li>
<li><code>update</code>：让你展示数量是否要被加进已当前的产品数量上（<code>False</code>），否则如果当前数量必须被用给定的数量给更新（<code>True</code>）。我们为这个字段使用了<code>HiddenInput</code> 控件，因为我们不想把它展示给用户。</li>
</ul>
<p>让我们一个新的视图来想购物车中添加物品。编辑 <code>cart</code> 应用的 <code>views.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, redirect, get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> django.views.decorators.http <span class="im"><span class="hljs-keyword">import</span></span> require_POST
<span class="im"><span class="hljs-keyword">from</span></span> shop.models <span class="im"><span class="hljs-keyword">import</span></span> Product
<span class="im"><span class="hljs-keyword">from</span></span> .cart <span class="im"><span class="hljs-keyword">import</span></span> Cart
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> CartAddProductForm

<span class="at"><span class="hljs-meta">@require_POST</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_add</span><span class="hljs-params">(request, product_id)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    product <span class="op">=</span> get_object_or_404(Product, <span class="bu">id</span><span class="op">=</span>product_id)
    form <span class="op">=</span> CartAddProductForm(request.POST)
    <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
        cd <span class="op">=</span> form.cleaned_data
        cart.add(product<span class="op">=</span>product,
                quantity<span class="op">=</span>cd[<span class="st"><span class="hljs-string">'quantity'</span></span>],
                update_quantity<span class="op">=</span>cd[<span class="st"><span class="hljs-string">'update'</span></span>])
    <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'cart:cart_detail'</span></span>)</code></pre></div>
<p>这个视图是为了想购物车添加新的产品或者更新当前产品的数量。我们使用 <code>require_POST</code> 装饰器来只响应 POST 请求，因为这个视图将会变更数据。这个视图接收产品 ID 作为参数。我们用给定的 ID 来检索 <code>Product</code> 实例，然后验证 <code>CartAddProductForm</code>。如果表单是合法的，我们将在购物车中添加或者更新产品。我们将创建 <code>cart_detail</code> 视图。</p>
<p>我们还需要一个视图来删除购物车中的物品。将以下代码添加进 <code>cart</code> 应用的 <code>views.py</code> 中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_remove</span><span class="hljs-params">(request, product_id)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    product <span class="op">=</span> get_object_or_404(Product, <span class="bu">id</span><span class="op">=</span>product_id)
    cart.remove(product)
    <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'cart:cart_detail'</span></span>)</code></pre></div>
<p><code>cart_detail</code> 视图接收产品 ID 作为参数。我们根据给定的产品 ID 检索相应的 <code>Product</code> 实例，然后将它从购物车中删除。然后，我们将用户重定向到 <code>cart_detail</code> URL。</p>
<p>最后，我们需要一个视图来展示购物车和其中的物品。讲一下代码添加进 <code>veiws.py</code> 中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_detail</span><span class="hljs-params">(request)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'cart/detail.html'</span></span>, {<span class="st"><span class="hljs-string">'cart'</span></span>: cart})</code></pre></div>
<p><code>cart_detail</code> 视图获取当前购物车并展示它。</p>
<p>我们已经创建了视图来向购物车中添加物品，或从购物车中更新数量，删除物品，还有展示他们。然我们为这些视图添加 URL 模式。在 <code>cart</code> 应用中创建一个新的文件，命名为 <code>urls.py</code>。把下面这些 URL 模式添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^$'</span></span>, views.cart_detail, name<span class="op">=</span><span class="st"><span class="hljs-string">'cart_detail'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^add/(?P&lt;product_id&gt;\d+)/$'</span></span>,
            views.cart_add,
            name<span class="op">=</span><span class="st"><span class="hljs-string">'cart_add'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^remove/(?P&lt;product_id&gt;\d+)/$'</span></span>,
            views.cart_remove,
            name<span class="op">=</span><span class="st"><span class="hljs-string">'cart_remove'</span></span>),
]</code></pre></div>
<p>编辑 <code>myshop</code> 应用的主 <code>urls.py</code> 文件，添加以下 URL 模式来引用 <code>cart</code> URLs：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
    url(<span class="vs"><span class="hljs-string">r'^cart/'</span></span>, include(<span class="st"><span class="hljs-string">'cart.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'cart'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^'</span></span>, include(<span class="st"><span class="hljs-string">'shop.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'shop'</span></span>)),
]</code></pre></div>
<p>确保你在 <code>shop.urls</code> 之前引用它，因为它比前者更加有限制性。</p>
<h2 id="创建展示购物车的模板"><strong>创建展示购物车的模板</strong></h2>
<p><code>cart_add</code> 和 <code>cart_remove</code> 视图没有渲染任何模板，但是我们需要为 <code>cart_detail</code> 创建模板。</p>
<p>在 <code>cart</code> 应用路径下创建以下文件结构：</p>
<pre><code class="hljs">templates/
    cart/
        detail.html</code></pre>
<p>编辑 <code>cart/detail.html</code> 模板，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> load static <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
    Your shopping cart
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Your shopping cart<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">table</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">thead</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Image<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Product<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Quantity<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Remove<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Unit price<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>                
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Price<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">thead</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tbody</span>&gt;</span></span>
        <span>{</span><span>%</span> for item in cart <span>%</span><span>}</span>
            <span>{</span><span>%</span> with product=item.product <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>
                    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> product.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                        ![](<span>{</span><span>%</span> if product.image <span>%</span><span>}</span><span>{</span><span>{</span> product.image.url <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span><span>{</span><span>%</span> static )
                    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> product.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> item.quantity <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">cart:cart_remove"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">product.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Remove<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
            <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
        <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"total"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"4"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tbody</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">table</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"text-right"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">shop:product_list"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button light"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Continue shopping<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"#"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Checkout<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板被用于展示购物车的内容。它包含了一个保存于当前购物车物品的表格。我们允许用用户使用发送到 <code>cart_add</code> 表单来改变选中的产品数量。我们通过提供一个 <em>Remove</em> 链接来允许用户从购物车中删除物品。</p>
<h2 id="向购物车中添加物品"><strong>向购物车中添加物品</strong></h2>
<p>现在，我们需要在产品详情页添加一个 <strong>Add to cart</strong> 按钮。编辑 <code>shop</code> 应用中的 <code>views.py</code>，然后把 <code>CartAddProductForm</code> 添加进 <code>product_detail</code> 视图中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> cart.forms <span class="im"><span class="hljs-keyword">import</span></span> CartAddProductForm

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">product_detail</span><span class="hljs-params">(request, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="hljs-function"><span class="hljs-params">, slug)</span>:</span>
    product <span class="op">=</span> get_object_or_404(Product, <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                                slug<span class="op">=</span>slug,
                                available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    cart_product_form <span class="op">=</span> CartAddProductForm()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
            <span class="st"><span class="hljs-string">'shop/product/detail.html'</span></span>,
            {<span class="st"><span class="hljs-string">'product'</span></span>: product,
            <span class="co"><span class="hljs-string">'cart_product_form'</span></span>: cart_product_form})</code></pre></div>
<p>编辑 <code>shop</code> 应用的 <code>shop/product/detail.html</code> 模板，然后将如下表格按照这样添加产品价格：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"price"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> product.price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">cart:cart_add"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">product.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>{</span> cart_product_form <span>}</span><span>}</span>
<span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Add to cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span></code></pre></div>
<p>确保用 <code>python manage.py runserver</code> 运行开发服务器。现在，打开 <a href="http://127.0.0.1:8000/%EF%BC%8C%E5%AF%BC%E8%88%AA%E5%88%B0%E4%BA%A7%E5%93%81%E8%AF%A6%E6%83%85%E9%A1%B5%E3%80%82%E7%8E%B0%E5%9C%A8%E5%AE%83%E5%8C%85%E5%90%AB%E4%BA%86%E4%B8%80%E4%B8%AA%E8%A1%A8%E5%8D%95%E6%9D%A5%E9%80%89%E6%8B%A9%E6%95%B0%E9%87%8F%E5%9C%A8%E5%B0%86%E4%BA%A7%E5%93%81%E6%B7%BB%E5%8A%A0%E8%BF%9B%E8%B4%AD%E7%89%A9%E8%BD%A6%E4%B9%8B%E5%89%8D%E3%80%82%E8%BF%99%E4%B8%AA%E9%A1%B5%E9%9D%A2%E7%9C%8B%E8%B5%B7%E6%9D%A5%E5%83%8F%E8%BF%99%E6%A0%B7" class="uri">http://127.0.0.1:8000/，导航到产品详情页。现在它包含了一个表单来选择数量在将产品添加进购物车之前。这个页面看起来像这样</a>：</p>
<p><iframe id="iframe_0.11778421616587664" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-1430958dff103ea9.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.11778421616587664',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 546px; height: 233px;" frameborder="0" scrolling="no"></iframe></p>
<p>选择一个数量，然后点击 <strong>Add to cart</strong> 按钮。表单将会通过 POST 方法提交到 <code>cart_add</code> 视图。视图会把产品添加进当前会话的购物车当中，包括当前产品的价格和选定的数量。然后，用户将会被重定向到购物车详情页，它长得像这个样子：</p>
<p><iframe id="iframe_0.6933246036468308" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-2c7e857daa3bcd60.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.6933246036468308',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 627px; height: 359px;" frameborder="0" scrolling="no"></iframe></p>
<h2 id="在购物车中更新产品数量"><strong>在购物车中更新产品数量</strong></h2>
<p>当用户看到购物车时，他们可能想要在下单之前改变产品数量。我们将会允许用户在详情页改变产品数量。</p>
<p>编辑 <code>cart</code> 应用的 <code>views.py</code>，然后把 <code>cart_detail</code> 改成这个样子：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_detail</span><span class="hljs-params">(request)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> cart:
        item[<span class="st"><span class="hljs-string">'update_quantity_form'</span></span>] <span class="op">=</span> CartAddProductForm(
                                    initial<span class="op">=</span>{<span class="st"><span class="hljs-string">'quantity'</span></span>: item[<span class="st"><span class="hljs-string">'quantity'</span></span>],
                                    <span class="st"><span class="hljs-string">'update'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>})
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'cart/detail.html'</span></span>, {<span class="st"><span class="hljs-string">'cart'</span></span>: cart})</code></pre></div>
<p>我们为每一个购物车中的物品创建了 <code>CartAddProductForm</code> 实例来允许用户改变产品的数量。我们把表单和当前物品数量一同初始化，然后把 <code>update</code> 字段设为 <code>True</code> ，这样当我们提交表单到 <code>cart_add</code> 视图时，当前的数量就被新的数量替换了。</p>
<p>现在，编辑 <code>cart</code> 应用的 <code>cart/detail.html</code> 模板，然后找到这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span> <span>{</span><span>{</span> item.quantity <span>}</span><span>}</span> <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span></code></pre></div>
<p>把它替换为下面这样的代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">cart:cart_add"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">product.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>{</span> item.update_quantity_form.quantity <span>}</span><span>}</span>
<span>{</span><span>{</span> item.update_quantity_form.update <span>}</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Update"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span></code></pre></div>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/cart/" class="uri">http://127.0.0.1:8000/cart/</a> 。你将会看到一个表单来编辑每个物品的数量，长得像下面这样：</p>
<p><iframe id="iframe_0.6667557989258892" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-6c6cc0b2791f0937.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.6667557989258892',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 615px; height: 352px;" frameborder="0" scrolling="no"></iframe></p>
<p>改变物品的数量，然后点击 <strong>Update</strong> 按钮来测试新的功能。</p>
<h1 id="为当前购物车创建上下文处理器"><strong>为当前购物车创建上下文处理器</strong></h1>
<p>你可能已经注意到我们在网站的头部展示了 <strong>Your cart is empty</strong> 的信息。当我们开始向购物车添加物品时，我们将看到它已经替换为了购物车中物品的总数和总花费。由于这是个展示在整个页面的东西，我们将创建一个上下文处理器来引用当前请求中的购物车，尽管我们的视图函数已经处理了它。</p>
<h2 id="上下文处理器"><strong>上下文处理器</strong></h2>
<p>上下文处理器是一个接收 <code>request</code> 对象为参数并返回一个已经添加了请求上下文字典的 Python 函数。他们在你需要让什么东西在所有模板都可用时迟早会派上用场。</p>
<p>一般的，当你用 <code>startproject</code> 命令创建一个新的项目时，你的项目将会包含下面的模板上下文处理器，他们位于 <code>TEMPLATES</code> 设置中的 <code>context_processors</code> 内：</p>
<ul>
<li><code>django.template.context_processors.debug</code>：在上下文中设置 <code>debug</code> 布尔值和 <code>sql_queries</code> 变量，来表示在 request 中执行的 SQL 查询语句表</li>
<li><code>django.template.context_processors.request</code>：在上下文中设置 request 变量</li>
<li><code>django.contrib.auth.context_processors.auth</code>：在请求中设置用户变量</li>
<li><code>django.contrib.messages.context_processors.messages</code>：在包含所有使用消息框架发送的信息的上下文中设置一个 <code>messages</code> 变量</li>
</ul>
<p>Django 也使用 <code>django.template.context_processors.csrf</code> 来避免跨站请求攻击。这个上下文处理器不在设置中，但是它总是可用的并且由安全原因不可被关闭。</p>
<p>你可以在这个网站看到所有的内建上下文处理器：<a href="https://docs.djangoproject.com/en/1.8/ref/templates/api/#built-in-template-context-processors" class="uri">https://docs.djangoproject.com/en/1.8/ref/templates/api/#built-in-template-context-processors</a></p>
<h2 id="把购物车添加进请求上下文中"><strong>把购物车添加进请求上下文中</strong></h2>
<p>让我们创建一个上下文处理器来将当前购物车添加进模板请求上下文中。这样我们就可以在任意模板中获取任意购物车了。</p>
<p>在 <code>cart</code> 应用路径里添加一个新文件，并命名为 <code>context_processors.py</code> 。上下文处理器可以位于你代码中的任何地方，但是在这里创建他们将会使你的代码变得组织有序。将以下代码添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .cart <span class="im"><span class="hljs-keyword">import</span></span> Cart

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart</span><span class="hljs-params">(request)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> {<span class="st"><span class="hljs-string">'cart'</span></span>: Cart(request)}</code></pre></div>
<p>如你所见，一个上下文处理器是一个函数，这个函数接收一个 <code>request</code> 对象作为参数，然后返回一个对象字典，这些对象可用于所有使用 <code>RequestContext</code> 渲染的模板。在我们的上下文处理器中，我们使用 <code>request</code> 对象实例化了购物车，然后让它作为一个名为 <code>cart</code> 的参数对模板可用。</p>
<p>编辑项目中的 <code>settings.py</code> ，然后把 <code>cart.context_processors.cart</code> 添加进 <code>TEMPLATE</code> 内的 <code>context_processors</code> 选项中。改变后的设置如下：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">TEMPLATES <span class="op">=</span> [
    {
        <span class="st"><span class="hljs-string">'BACKEND'</span></span>: <span class="st"><span class="hljs-string">'django.template.backends.django.DjangoTemplates'</span></span>,
        <span class="co"><span class="hljs-string">'DIRS'</span></span>: [],
        <span class="co"><span class="hljs-string">'APP_DIRS'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>,
        <span class="co"><span class="hljs-string">'OPTIONS'</span></span>: {
            <span class="st"><span class="hljs-string">'context_processors'</span></span>: [
                <span class="st"><span class="hljs-string">'django.template.context_processors.debug'</span></span>,
                <span class="co"><span class="hljs-string">'django.template.context_processors.request'</span></span>,
                <span class="co"><span class="hljs-string">'django.contrib.auth.context_processors.auth'</span></span>,
                <span class="co"><span class="hljs-string">'django.contrib.messages.context_processors.messages'</span></span>,
                <span class="co"><span class="hljs-string">'cart.context_processors.cart'</span></span>,
            ],
        },
    },
]</code></pre></div>
<p>你的上下文处理器将会在使用 <code>RequestContext</code> 渲染 模板时执行。 <code>cart</code> 变量将会被设置在模板上下文中。</p>
<blockquote>
<p>上下文处理器会在所有的使用 <code>RequestContext</code> 的请求中执行。你可能想要创建一个定制的模板标签来代替一个上下文处理器，如果你想要链接到数据库的话。</p>
</blockquote>
<p>现在，编辑 <code>shop</code>应用的 <code>shop/base.html</code> 模板，然后找到这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
Your cart is empty.
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre></div>
<p>把它替换为下面的代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>%</span> with total_items=cart|length <span>%</span><span>}</span>
        <span>{</span><span>%</span> if cart|length &gt; 0 <span>%</span><span>}</span>
            Your cart:
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">cart:cart_detail"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
                <span>{</span><span>{</span> total_items <span>}</span><span>}</span> item<span>{</span><span>{</span> total_items|pluralize <span>}</span><span>}</span>,
                $<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
        <span>{</span><span>%</span> else <span>%</span><span>}</span>
            Your cart is empty.
        <span>{</span><span>%</span> endif <span>%</span><span>}</span>
    <span>{</span><span>%</span> endwith <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre></div>
<p>使用 <code>python manage.py runserver</code> 重载你的服务器。打开 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ,添加一些产品到购物车里。在网站头里，你可以看到当前物品总数和总花费，就象这样：</p>
<p><iframe id="iframe_0.12568617326650133" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-f2f07d94d9ab7266.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.12568617326650133',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 627px; height: 80px;" frameborder="0" scrolling="no"></iframe></p>
<h1 id="保存用户订单"><strong>保存用户订单</strong></h1>
<p>当购物车已经结账完毕时，你需要把订单保存进数据库中。订单将要保存客户信息和他们购买的产品信息。</p>
<p>使用下面的命令创建一个新的应用来管理用户订单：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">orders</span></code></pre>
<p>编辑项目中的 <code>settings.py</code> ，然后把 <code>orders</code> 添加进 <code>INSTALLED_APPS</code> 中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'orders'</span></span>,
)</code></pre></div>
<p>现在你已经激活了你的新应用。</p>
<h1 id="创建订单模型"><strong>创建订单模型</strong></h1>
<p>你需要一个模型来保存订单的详细信息，第二个模型用来保存购买的物品，包括物品的价格和数量。编辑 <code>orders</code> 应用的 <code>models.py</code> ，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> shop.models <span class="im"><span class="hljs-keyword">import</span></span> Product

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Order</span><span class="hljs-params">(models.Model)</span>:</span>
    first_name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">50</span></span>)
    last_name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">50</span></span>)
    email <span class="op">=</span> models.EmailField()
    address <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">250</span></span>)
    postal_code <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">20</span></span>)
    city <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">100</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    paid <span class="op">=</span> models.BooleanField(default<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>)
    
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'-created'</span></span>,)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'Order {}'</span></span>.<span class="bu">format</span>(<span class="va">self</span>.<span class="bu">id</span>)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_total_cost</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="bu">sum</span>(item.get_cost() <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.items.<span class="bu">all</span>())
        
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderItem</span><span class="hljs-params">(models.Model)</span>:</span>
    order <span class="op">=</span> models.ForeignKey(Order, related_name<span class="op">=</span><span class="st"><span class="hljs-string">'items'</span></span>)
    product <span class="op">=</span> models.ForeignKey(Product,
                    related_name<span class="op">=</span><span class="st"><span class="hljs-string">'order_items'</span></span>)
    price <span class="op">=</span> models.DecimalField(max_digits<span class="op">=</span><span class="dv"><span class="hljs-number">10</span></span>, decimal_places<span class="op">=</span><span class="dv"><span class="hljs-number">2</span></span>)
    quantity <span class="op">=</span> models.PositiveIntegerField(default<span class="op">=</span><span class="dv"><span class="hljs-number">1</span></span>)
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'{}'</span></span>.<span class="bu">format</span>(<span class="va">self</span>.<span class="bu">id</span>)
        
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_cost</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.price <span class="op">*</span> <span class="va">self</span>.quantity</code></pre></div>
<p><code>Order</code> 模型包含几个用户信息的字段和一个 <code>paid</code> 布尔值字段，这个字段默认值为 <code>False</code> 。待会儿，我们将使用这个字段来区分支付和未支付订单。我们也定义了一个 <code>get_total_cost()</code> 方法来得到订单中购买物品的总花费。</p>
<p><code>OrderItem</code> 模型让我们可以保存物品，数量和每个物品的支付价格。我们引用 <code>get_cost()</code> 来返回物品的花费。</p>
<p>给 <code>orders</code> 应用下运行首次迁移：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你将看到如下输出：</p>
<pre class="shell"><code class="hljs sql">Migrations for 'orders':
    0001_initial.py:
        - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> <span class="hljs-keyword">Order</span>
        - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> OrderItem</code></pre>
<p>运行以下命令来应用新的迁移：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你的订单模型已经同步到了数据库中</p>
<h2 id="在管理站点引用订单模型"><strong>在管理站点引用订单模型</strong></h2>
<p>让我们把订单模型添加到管理站点。编辑 <code>orders</code> 应用的 <code>admin.py</code>：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Order, OrderItem

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderItemInline</span><span class="hljs-params">(admin.TabularInline)</span>:</span>
    model <span class="op">=</span> OrderItem
    raw_id_fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'product'</span></span>]
    
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'first_name'</span></span>, <span class="st"><span class="hljs-string">'last_name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>,
                    <span class="co"><span class="hljs-string">'address'</span></span>, <span class="st"><span class="hljs-string">'postal_code'</span></span>, <span class="st"><span class="hljs-string">'city'</span></span>, <span class="st"><span class="hljs-string">'paid'</span></span>,
                    <span class="co"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>]
    list_filter <span class="op">=</span> [<span class="st"><span class="hljs-string">'paid'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>]
    inlines <span class="op">=</span> [OrderItemInline]
    
admin.site.register(Order, OrderAdmin)</code></pre></div>
<p>我们在 <code>OrderItem</code> 使用 <code>ModelInline</code> 来把它引用为 <code>OrderAdmin</code> 类的内联元素。一个内联元素允许你在同一编辑页引用模型，并且将这个模型作为父模型。</p>
<p>用 <code>python manage.py runserver</code> 命令打开开发服务器，访问 <a href="http://127.0.0.1:8000/admin/orders/order/add/" class="uri">http://127.0.1:8000/admin/orders/order/add/</a> 。你将会看到如下页面：</p>
<p><iframe id="iframe_0.8973858115574718" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-5bcb0d299a74175c.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.8973858115574718',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 627px; height: 400px;" frameborder="0" scrolling="no"></iframe></p>
<h2 id="创建顾客订单"><strong>创建顾客订单</strong></h2>
<p>我们需要使用订单模型来保存在用户最终下单时在购物车中的物品，创建新的订单的工作流程如下：</p>
<ul>
<li><ol>
<li>向用户展示一个订单表来让他们填写数据</li>
</ol></li>
<li><ol>
<li>我们用用户输入的数据创建一个新的 <code>Order</code> 实例，然后我们创建每个物品相关联的 <code>OrderItem</code> 实例。</li>
</ol></li>
<li><ol>
<li>我们清空购物车，然后把用户重定向到成功页面</li>
</ol></li>
</ul>
<p>首先，我们需要一个表单来输入订单详情。在<code>orders</code> 应用路径内创建一个新的文件，命名为 <code>forms.py</code> 。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Order

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderCreateForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Order
        fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'first_name'</span></span>, <span class="st"><span class="hljs-string">'last_name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>, <span class="st"><span class="hljs-string">'address'</span></span>,
                <span class="co"><span class="hljs-string">'postal_code'</span></span>, <span class="st"><span class="hljs-string">'city'</span></span>]</code></pre></div>
<p>这是我们将要用于创建新的 <code>Order</code> 对象的表单。现在，我们需要一个视图来管理表格以及创建一个新的订单。编辑<code>orders</code>应用的 <code>views.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> OrderItem
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> OrderCreateForm
<span class="im"><span class="hljs-keyword">from</span></span> cart.cart <span class="im"><span class="hljs-keyword">import</span></span> Cart

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">order_create</span><span class="hljs-params">(request)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    <span class="cf"><span class="hljs-keyword">if</span></span> request.method <span class="op">==</span> <span class="st"><span class="hljs-string">'POST'</span></span>:
        form <span class="op">=</span> OrderCreateForm(request.POST)
        <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
            order <span class="op">=</span> form.save()
            <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> cart:
                OrderItem.objects.create(order<span class="op">=</span>order,
                    product<span class="op">=</span>item[<span class="st"><span class="hljs-string">'product'</span></span>],
                    price<span class="op">=</span>item[<span class="st"><span class="hljs-string">'price'</span></span>],
                    quantity<span class="op">=</span>item[<span class="st"><span class="hljs-string">'quantity'</span></span>])
            <span class="co"><span class="hljs-comment"># clear the cart</span></span>
            cart.clear()
            <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                <span class="st"><span class="hljs-string">'orders/order/created.html'</span></span>,
                {<span class="st"><span class="hljs-string">'order'</span></span>: order})
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        form <span class="op">=</span> OrderCreateForm()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
            <span class="st"><span class="hljs-string">'orders/order/create.html'</span></span>,
            {<span class="st"><span class="hljs-string">'cart'</span></span>: cart, <span class="st"><span class="hljs-string">'form'</span></span>: form})</code></pre></div>
<p>在 <code>order_create</code> 视图中，我们将用 <code>cart = Cart(request)</code> 获取到当前会话中的购物车。基于请求方法，我们将执行以下几个任务：</p>
<ul>
<li><strong>GET 请求</strong>：实例化 <code>OrderCreateForm</code> 表单然后渲染模板 <code>orders/order/create.html</code></li>
<li><strong>POST 请求</strong>：验证提交的数据。如果数据是合法的，我们将使用 <code>order = form.save()</code> 来创建一个新的 <code>Order</code> 实例。然后我们将会把它保存进数据库中，之后再把它保存进 <code>order</code> 变量里。在创建 <code>order</code> 之后，我们将迭代无购车的物品然后为每个物品创建 <code>OrderItem</code>。最后，我们清空购物车。</li>
</ul>
<p>现在，在 <code>orders</code> 应用路径下创建一个新的文件，把它命名为 <code>urls.py</code>。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
        url(<span class="vs"><span class="hljs-string">r'^create/$'</span></span>,
            views.order_create,
            name<span class="op">=</span><span class="st"><span class="hljs-string">'order_create'</span></span>),
]</code></pre></div>
<p>这个是 <code>order_create</code> 视图的 URL 模式。编辑 <code>myshop</code> 的 <code>urls.py</code> ，把下面的模式引用进去。记得要把它放在 <code>shop.urls</code> 模式之前：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^orders/'</span>, include(<span class="hljs-string">'orders.urls'</span>, namespace=<span class="hljs-string">'orders'</span>)),</code></pre>
<p>编辑 <code>cart</code> 应用的 <code>cart/detail.html</code> 模板，找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"#"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Checkout<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span></code></pre></div>
<p>替换为：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">orders:order_create"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"button"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
Checkout
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span></code></pre></div>
<p>用户现在可以从购物车详情页导航到订单表了。我们依然需要定义一个下单模板。在 <code>orders</code> 应用路径下创建如下文件结构：</p>
<pre><code class="hljs apache"><span class="hljs-attribute">templates</span>/
    <span class="hljs-attribute">orders</span>/
        <span class="hljs-attribute"><span class="hljs-nomarkup">order</span></span>/
            <span class="hljs-attribute">create</span>.html
            <span class="hljs-attribute">created</span>.html</code></pre>
<p>编辑 <code>ordrs/order/create.html</code> 模板，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
Checkout
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Checkout<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"order-info"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Your order<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span></span>
            <span>{</span><span>%</span> for item in cart <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
                <span>{</span><span>{</span> item.quantity <span>}</span><span>}</span>x <span>{</span><span>{</span> item.product.name <span>}</span><span>}</span>
                <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>$<span>{</span><span>{</span> item.total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
            <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Total: $<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>

<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"."</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"order-form"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Place order"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
    <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>模板展示的购物车物品包括物品总量和下单表。</p>
<p>编辑 <code>orders/order/created.html</code> 模板，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
Thank you
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Thank you<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Your order has been successfully completed. Your order number is
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">strong</span>&gt;</span></span><span>{</span><span>{</span> order.id <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">strong</span>&gt;</span></span>.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这是当订单成功创建时我们渲染的模板。打开开发服务器，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，在购物车当中添加几个产品进去，然后结账。<br>
你就会看到下面这个页面：</p>
<p><iframe id="iframe_0.024222283803713385" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-579333923e4f73fc.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.024222283803713385',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 626px; height: 462px;" frameborder="0" scrolling="no"></iframe></p>
<p>用合法的数据填写表单，然后点击 <strong>Place order</strong> 按钮。订单就会被创建，然后你将会看到成功页面：</p>
<p><iframe id="iframe_0.3858937920741945" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-98bebd6f51080551.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.3858937920741945',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 556px; height: 160px;" frameborder="0" scrolling="no"></iframe></p>
<h1 id="使用-celery-执行异步操作"><strong>使用 Celery 执行异步操作</strong></h1>
<p>你在视图执行的每个操作都会影响响应的时间。在很多场景下你可能想要尽快的给用户返回响应，并且让服务器异步地执行一些操作。这特别和耗时进程或从属于失败的进程时需要重新操作时有着密不可分的关系。比如，一个视频分享平台允许用户上传视频但是需要相当长的时间来转码上传的视频。这个网站可能会返回一个响应给用户，告诉他们转码即将开始，然后开始异步转码。另一个例子是给用户发送邮件。如果你的网站发送了通知邮件，SMTP 连接可能会失败或者减慢响应的速度。执行异步操作来避免阻塞执行就变得必要起来。</p>
<p>Celery 是一个分发队列，它可以处理大量的信息。它既可以执行实时操作也支持任务调度。使用 Celery 不仅可以让你很轻松的创建异步任务还可以让这些任务尽快执行，但是你需要在一个指定的时间调度他们执行。</p>
<p>你可以在这个网站找到 Celery 的官方文档：<a href="http://celery.readthedocs.org/en/latest/" class="uri">http://celery.readthedocs.org/en/latest/</a></p>
<h2 id="安装-celery"><strong>安装 Celery</strong></h2>
<p>让我们安装 Celery 然后把它整合进你的项目中。用下面的命令安装 Celery：</p>
<pre class="shell"><code class="hljs cmake">pip <span class="hljs-keyword">install</span> celery==<span class="hljs-number">3.1</span>.<span class="hljs-number">18</span></code></pre>
<p>Celery 需要一个消息代理（message broker）来管理请求。这个代理负责向 Celery 的 worker 发送消息，当接收到消息时 worker 就会执行任务。让我们安装一个消息代理。</p>
<h2 id="安装-rabbitmq"><strong>安装 RabbitMQ</strong></h2>
<p>有几个 Celery 的消息代理可供选择，包括键值对储存，比如 Redis 或者是实时消息系统，比如 RabbitMQ。我们会用 RabbitMQ 配置 Celery ，因为它是 Celery 推荐的 message worker。</p>
<p>如果你用的是 Linux，你可以用下面这个命令安装 RabbitMQ ：</p>
<pre class="shell"><code class="hljs swift">apt-<span class="hljs-keyword">get</span> install rabbitmg</code></pre>
<p><strong>（<a href="mailto:%E8%AF%91%E8%80%85@夜夜月注">译者@夜夜月注</a>：这是debian系linux的安装方式）</strong></p>
<p>如果你需要在 Mac OSX 或者 Windows 上安装 RabbitMQ，你可以在这个网站找到独立的支持版本：<br>
<a href="https://www.rabbitmq.com/download.html" class="uri">https://www.rabbitmq.com/download.html</a></p>
<p>在安装它之后，使用下面的命令执行 RabbitMQ：</p>
<pre class="shell"><code class="hljs">rabbitmg-server</code></pre>
<p>你将会在最后一行看到这样的输出：</p>
<pre class="shell"><code class="hljs python">Starting broker... completed <span class="hljs-keyword">with</span> <span class="hljs-number">10</span> plugins</code></pre>
<p>RabbitMQ 正在运行了，准备接收消息。</p>
<h2 id="把-celery-添加进你的项目"><strong>把 Celery 添加进你的项目</strong></h2>
<p>你必须为 Celery 实例提供配置。在 <code>myshop</code> 的 <code>settings.py</code> 文件的旁边创建一个新的文件，命名为 <code>celery.py</code> 。这个文件会包含你项目的 Celery 配置。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">import</span></span> os
<span class="im"><span class="hljs-keyword">from</span></span> celery <span class="im"><span class="hljs-keyword">import</span></span> Celery
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings

<span class="co"><span class="hljs-comment"># set the default Django settings module for the 'celery' program.</span></span>
os.environ.setdefault(<span class="st"><span class="hljs-string">'DJANGO_SETTINGS_MODULE'</span></span>, <span class="st"><span class="hljs-string">'myshop.settings'</span></span>)

app <span class="op">=</span> Celery(<span class="st"><span class="hljs-string">'myshop'</span></span>)

app.config_from_object(<span class="st"><span class="hljs-string">'django.conf:settings'</span></span>)
app.autodiscover_tasks(<span class="kw"><span class="hljs-keyword">lambda</span></span>: settings.INSTALLED_APPS)</code></pre></div>
<p>在这段代码中，我们为 Celery 命令行程序设置了 <code>DJANGO_SETTINGS_MODULE</code> 变量。然后我们用 <code>app=Celery('myshop')</code> 创建了一个实例。我们用 <code>config_from_object()</code> 方法来加载项目设置中任意的定制化配置。最后，我们告诉 Celery 自动查找我们列举在 <code>INSTALLED_APPS</code> 设置中的异步应用任务。Celery 将在每个应用路径下查找 <code>task.py</code> 来加载定义在其中的异步任务。</p>
<p>你需要在你项目中的 <code>__init__.py</code> 文件中导入 <code>celery</code>来确保在 Django 开始的时候就会被加载。编辑 <code>myshop/__init__.py</code> 然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># import celery</span></span>
<span class="im"><span class="hljs-keyword">from</span></span> .celery <span class="im"><span class="hljs-keyword">import</span></span> app <span class="im"><span class="hljs-keyword">as</span></span> celery_app</code></pre></div>
<p>现在，你可以为你的项目开始编写异步任务了。</p>
<blockquote>
<p><code>CELERY_ALWAYS_EAGER</code> 设置允许你在本地用异步的方式执行任务而不是把他们发送向队列中。这在不运行 Celery 的情况下， 运行单元测试或者是运行在本地环境中的项目是很有用的。</p>
</blockquote>
<h2 id="向你的应用中添加异步任务"><strong>向你的应用中添加异步任务</strong></h2>
<p>我们将创建一个异步任务来发送消息邮件来让用户知道他们下单了。</p>
<p>约定俗成的一般用法是，在你的应用路径下的 <code>tasks</code> 模型里引入你应用的异步任务。在 <code>orders</code> 应用内创建一个新的文件，并命名为 <code>task.py</code> 。这是 Celery 寻找异步任务的地方。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> celery <span class="im"><span class="hljs-keyword">import</span></span> task
<span class="im"><span class="hljs-keyword">from</span></span> django.core.mail <span class="im"><span class="hljs-keyword">import</span></span> send_mail
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Order

<span class="at"><span class="hljs-meta">@task</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">order_created</span><span class="hljs-params">(order_id)</span>:</span>
    <span class="co"><span class="hljs-string">"""</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    Task to send an e-mail notification when an order is</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    successfully created.</span></span><span class="hljs-string">
</span><span class="co"><span class="hljs-string">    """</span></span>
    order <span class="op">=</span> Order.objects.get(<span class="bu">id</span><span class="op">=</span>order_id)
    subject <span class="op">=</span> <span class="st"><span class="hljs-string">'Order nr. {}'</span></span>.<span class="bu">format</span>(order.<span class="bu">id</span>)
    message <span class="op">=</span> <span class="st"><span class="hljs-string">'Dear {},</span></span><span class="ch"><span class="hljs-string">\n\n</span></span><span class="st"><span class="hljs-string">You have successfully placed an order.\</span></span><span class="hljs-string">
</span><span class="st"><span class="hljs-string">                Your order id is {}.'</span></span>.<span class="bu">format</span>(order.first_name,
                                            order.<span class="bu">id</span>)
    mail_sent <span class="op">=</span> send_mail(subject,
                        message,
                        <span class="st"><span class="hljs-string">'admin@myshop.com'</span></span>,
                        [order.email])
    <span class="cf"><span class="hljs-keyword">return</span></span> mail_sent</code></pre></div>
<p>我们通过使用 <code>task</code> 装饰器来定义我们的 <code>order_created</code> 任务。如你所见，一个 Celery 任务 只是一个用 <code>task</code> 装饰的 Python 函数。我们的 <code>task</code> 函数接收一个 <code>order_id</code> 参数。通常推荐的做法是只传递 ID 给任务函数然后在任务被执行的时候需找相关的对象，我们使用 Django 提供的 <code>send_mail()</code> 函数来发送一封提示邮件给用户告诉他们下单了。如果你不想安装邮件设置，你可以通过一下 <code>settings</code>.py` 中的设置告诉 Django 把邮件传给控制台：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">EMAIL_BACKEND <span class="op">=</span> <span class="st"><span class="hljs-string">'django.core.mail.backends.console.EmailBackend'</span></span></code></pre></div>
<blockquote>
<p>异步任务不仅仅适用于耗时进程，也适用于失败进程组中的进程，这些进程或许不会消耗太多时间，但是他们或许会链接失败或者需要再次尝试连接策略。</p>
</blockquote>
<p>现在我们要把任务添加到 <code>order_create</code> 视图中。打开 <code>orders</code> 应用的 <code>views.py</code> 文件，按照如下导入任务：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .tasks <span class="im"><span class="hljs-keyword">import</span></span> order_created</code></pre></div>
<p>然后在清除购物车之后调用 <code>order_created</code> 异步任务：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># clear the cart</span></span>
cart.clear()
<span class="co"><span class="hljs-comment"># launch asynchronous task</span></span>
order_created.delay(order.<span class="bu">id</span>)</code></pre></div>
<p>我们调用任务的 <code>delay()</code> 方法并异步地执行它。之后任务将会被添加进队列中，将会尽快被一个 worker 执行。</p>
<p>打开另外一个 shell ，使用以下命令开启 celery worker ：</p>
<pre class="shell"><code class="hljs nginx"><span class="hljs-attribute">celery</span> -A myshop worker -<span class="hljs-number">1</span> <span class="hljs-literal">info</span></code></pre>
<p>Celery worker 现在已经运行，准备好执行任务了。确保 Django 的开发服务器也在运行当中。访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ,在购物车中添加一些商品，然后完成一个订单。在 shell 中，你已经打开过了 Celery worker 所以你可以看到以下的相似输出：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-attr">[2015-09-14 19:43:47,526: INFO/MainProcess]</span> <span class="hljs-selector-tag">Received</span> <span class="hljs-selector-tag">task</span>: <span class="hljs-selector-tag">orders</span>.
<span class="hljs-selector-tag">tasks</span><span class="hljs-selector-class">.order_created</span><span class="hljs-selector-attr">[933e383c-095e-4cbd-b909-70c07e6a2ddf]</span>
<span class="hljs-selector-attr">[2015-09-14 19:43:50,851: INFO/MainProcess]</span> <span class="hljs-selector-tag">Task</span> <span class="hljs-selector-tag">orders</span><span class="hljs-selector-class">.tasks</span>.
<span class="hljs-selector-tag">order_created</span><span class="hljs-selector-attr">[933e383c-095e-4cbd-b909-70c07e6a2ddf]</span> <span class="hljs-selector-tag">succeeded</span> <span class="hljs-selector-tag">in</span>
3<span class="hljs-selector-class">.318835098994896s</span>: 1</code></pre>
<p>任务已经被执行了，你会接收到一封订单通知邮件。</p>
<h2 id="监控-celery"><strong>监控 Celery</strong></h2>
<p>你或许想要监控执行了的异步任务。下面就是一个基于 web 的监控 Celery 的工具。你可以用下面的命令安装 Flower:</p>
<pre class="shell"><code class="hljs cmake">pip <span class="hljs-keyword">install</span> flower</code></pre>
<p>安装之后，你可以在你的项目路径下用以下命令启动 Flower ：</p>
<pre class="shell"><code class="hljs nginx"><span class="hljs-attribute">celery</span> -A myshop flower</code></pre>
<p>在你的浏览器中访问 <a href="http://localhost:555/dashboard" class="uri">http://localhost:555/dashboard</a> ，你可以看到激活了的 Celery worker 和正在执行的异步任务统计：</p>
<p><iframe id="iframe_0.6607269854446989" src="data:text/html;charset=utf8,%3Cstyle%3Ebody%7Bmargin:0;padding:0%7D%3C/style%3E%3Cimg%20id=%22img%22%20src=%22http://upload-images.jianshu.io/upload_images/3966530-23edddf3ec71141e.png?imageMogr2/auto-orient/strip%257CimageView2/2/w/1240&amp;_=6495402%22%20style=%22border:none;max-width:848px%22%3E%3Cscript%3Ewindow.onload%20=%20function%20()%20%7Bvar%20img%20=%20document.getElementById('img');%20window.parent.postMessage(%7BiframeId:'iframe_0.6607269854446989',width:img.width,height:img.height%7D,%20'http://www.cnblogs.com');%7D%3C/script%3E" style="border: none; width: 626px; height: 189px;" frameborder="0" scrolling="no"></iframe></p>
<p>你可以在这个网站找到 Flower 的文档：<a href="http://flower.readthedocs.org/en/latest/" class="uri">http://flower.readthedocs.org/en/latest/</a></p>
<h1 id="总结"><strong>总结</strong></h1>
<p>在这一章中，你创建了一个最基本的商店应用。你创建了产品目录以及使用会话的购物车。你实现了定制化的上下文处理器来使购物车在你的模板中可用，实现了一个下单表格。你也学到了如何用 Celery 执行异步任务。</p>
<p>在下一章中，你将会学习在你的商店中整合一个支付网关，添加管理站点的定制化动作，以 CSV 的形式导出数据，以及动态的生成 PDF 文件。</p>
</div>