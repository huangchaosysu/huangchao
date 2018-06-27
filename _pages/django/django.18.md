---
layout: single
permalink: /django/example9/
title: "Django By Example 第九章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第九章"><strong>第九章</strong></h1>
<h2 id="拓展你的商店"><strong>拓展你的商店</strong></h2>
<p>在上一章中，你学习了如何把支付网关整合进你的商店。你处理了支付通知，学会了如何生成 CSV 和 PDF 文件。在这一章中，你会把优惠券添加进你的商店中。你将学到国际化（internationalization）和本地化（localization）是如何工作的，你还会创建一个推荐引擎。<br>
在这一章中将会包含一下知识点：</p>
<ul>
<li>创建一个优惠券系统来应用折扣</li>
<li>把国际化添加进你的项目中</li>
<li>使用 Rosetta 来管理翻译</li>
<li>使用 django-parler 来翻译模型（model）</li>
<li>建立一个产品推荐引擎</li>
</ul>
<h2 id="创建一个优惠券系统"><strong>创建一个优惠券系统</strong></h2>
<p>很多的在线商店会送出很多优惠券，这些优惠券可以在顾客的采购中兑换为相应的折扣。在线优惠券通常是由一串给顾客的代码构成，这串代码在一个特定的时间段内是有效的。这串代码可以被兑换一次或者多次。</p>
<p>我们将会为我们的商店创建一个优惠券系统。优惠券将会在顾客在某一个特定的时间段内输入时生效。优惠券没有任何兑换数的限制，他们也可用于购物车的总金额中。对于这个功能，我们将会创建一个模型（model）来储存优惠券代码，优惠券有效的时间段，以及折扣的力度。</p>
<p>在 <code>myshop</code> 项目内使用如下命令创建一个新的应用：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">coupons</span></code></pre>
<p>编辑 <code>myshop</code> 的 <code>settings.py</code> 文件，像下面这样把把应用添加到 <code>INSTALLED_APPS</code> 中：</p>
<pre><code class="hljs lisp">INSTALLED_APPS = (
      <span class="hljs-name">#</span> ...
      'coupons',
)</code></pre>
<p>新的应用已经在我们的 Django 项目中激活了。</p>
<h2 id="创建优惠券模型model"><strong>创建优惠券模型（model）</strong></h2>
<p>让我们开始创建 <code>Coupon</code> 模型（model）。编辑 <code>coupons</code> 应用中的 <code>models.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models
<span class="im"><span class="hljs-keyword">from</span></span> django.core.validators <span class="im"><span class="hljs-keyword">import</span></span> MinValueValidator,<span class="op">\</span>
                                    MaxValueValidator
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Coupon</span><span class="hljs-params">(models.Model)</span>:</span>
    code <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">50</span></span>,
                            unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    valid_from <span class="op">=</span> models.DateTimeField()
    valid_to <span class="op">=</span> models.DateTimeField()
    discount <span class="op">=</span> models.IntegerField(
                validators<span class="op">=</span>[MinValueValidator(<span class="dv"><span class="hljs-number">0</span></span>),
                            MaxValueValidator(<span class="dv"><span class="hljs-number">100</span></span>)])
    active <span class="op">=</span> models.BooleanField()
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> </span><span class="fu"><span class="hljs-function"><span class="hljs-title">__str__</span></span></span><span class="hljs-function"><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.code</code></pre></div>
<p>我们将会用这个模型（model）来储存优惠券。 <code>Coupon</code> 模型（model）包含以下几个字段：</p>
<ul>
<li><code>code</code>：用户必须要输入的代码来将优惠券应用到他们购买的商品中</li>
<li><code>valid_from</code>：表示优惠券会在何时生效的时间和日期值</li>
<li><code>valid_to</code>：表示优惠券会在何时过期</li>
<li><code>discount</code>：折扣率（这是一个百分比，所以它的值的范围是 0 到 1000）。我们使用验证器来限制接收的最小值和最大值</li>
<li><code>active</code>：表示优惠券是否激活的布尔值</li>
</ul>
<p>执行下面的命令来为 <code>coupons</code> 生成首次迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>输出应该包含以下这几行：</p>
<pre><code class="hljs groovy">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'coupons'</span>:
    <span class="hljs-number">0001</span>_initial.<span class="hljs-string">py:</span>
        - Create model Coupon</code></pre>
<p>之后我们执行下面的命令来应用迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你可以看见包含下面这一行的输出：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">coupons</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>迁移现在已经被应用到了数据库中。让我们把 <code>Coupon</code> 模型（model）添加到管理站点。编辑 <code>coupons</code> 应用的 <code>admin.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models improt Coupon

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CouponAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'code'</span></span>, <span class="st"><span class="hljs-string">'valid_from'</span></span>, <span class="st"><span class="hljs-string">'valid_to'</span></span>,
                    <span class="co"><span class="hljs-string">'discount'</span></span>, <span class="st"><span class="hljs-string">'active'</span></span>]
    list_filter <span class="op">=</span> [<span class="st"><span class="hljs-string">'active'</span></span>, <span class="st"><span class="hljs-string">'valid_from'</span></span>, <span class="st"><span class="hljs-string">'valid_to'</span></span>]
    search_fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'code'</span></span>]
admin.site.register(Coupon, CouponAdmin)</code></pre></div>
<p><code>Coupon</code> 模型（model）现在已经注册进了管理站点中。确保你已经用命令 <code>python manage.py runserver</code> 打开了开发服务器。访问 <a href="http://127.0.0.1:8000/admin/coupons/add" class="uri">http://127.0.0.1:8000/admin/coupons/add</a> 。你可以看见下面的表单：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-1.png" alt="django-9-1"></p>
<p>填写表单创建一个在当前日期有效的新优惠券，确保你点击了 <strong>Active</strong> 复选框，然后点击 <strong>Save</strong>按钮。</p>
<h2 id="把应用优惠券到购物车中"><strong>把应用优惠券到购物车中</strong></h2>
<p>我们可以保存新的优惠券以及检索目前的优惠券。现在我们需要一个方法来让顾客可以应用他们的优惠券到他们购买的产品中。花点时间来想想这个功能该如何实现。应用一张优惠券的流程如下：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>用户将产品添加进购物车
<span class="hljs-bullet">2. </span>用户在购物车详情页的表单中输入优惠代码
<span class="hljs-bullet">3. </span>当用户输入优惠代码然后提交表单时，我们查找一张和所给优惠代码相符的有效优惠券。我们必须检查用户输入的优惠券代码， <span class="hljs-code">`active`</span> 属性为 <span class="hljs-code">`True`</span> ，当前时间位于 <span class="hljs-code">`valid_from 和 `</span>valid_to` 之间。
<span class="hljs-bullet">4. </span>如果查找到了相应的优惠券，我们把它保存在用户会话中，然后展示包含折扣了的购物车以及更新总价。
<span class="hljs-bullet">5. </span>当用户下单时，我们把优惠券保存到所给的订单中。</code></pre>
<p>在 <code>coupons</code> 应用路径下创建一个新的文件，命名为 <code>forms.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CouponApplyForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    code <span class="op">=</span> forms.CharField()</code></pre></div>
<p>我们将会用这个表格来让用户输入优惠券代码。编辑 <code>coupons</code> 应用中的 <code>views.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, redirect
<span class="im"><span class="hljs-keyword">from</span></span> django.utils <span class="im"><span class="hljs-keyword">import</span></span> timezone
<span class="im"><span class="hljs-keyword">from</span></span> django.views.decorators.http <span class="im"><span class="hljs-keyword">import</span></span> require_POST
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Coupon
<span class="im"><span class="hljs-keyword">from</span></span> .forms <span class="im"><span class="hljs-keyword">import</span></span> CouponApplyForm

<span class="at"><span class="hljs-meta">@require_POST</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">coupon_apply</span><span class="hljs-params">(request)</span>:</span>
    now <span class="op">=</span> timezone.now()
    form <span class="op">=</span> CouponApplyForm(request.POST)
    <span class="cf"><span class="hljs-keyword">if</span></span> form.is_valid():
        code <span class="op">=</span> form.cleaned_data[<span class="st"><span class="hljs-string">'code'</span></span>]
        <span class="cf"><span class="hljs-keyword">try</span></span>:
            coupon <span class="op">=</span> Coupon.objects.get(code__iexact<span class="op">=</span>code,
                                    valid_from__lte<span class="op">=</span>now,
                                    valid_to__gte<span class="op">=</span>now,
                                    active<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
            request.session[<span class="st"><span class="hljs-string">'coupon_id'</span></span>] <span class="op">=</span> coupon.<span class="bu">id</span>
        <span class="cf"><span class="hljs-keyword">except</span></span> Coupon.DoesNotExist:
            request.session[<span class="st"><span class="hljs-string">'coupon_id'</span></span>] <span class="op">=</span> <span class="va"><span class="hljs-keyword">None</span></span>
    <span class="cf"><span class="hljs-keyword">return</span></span> redirect(<span class="st"><span class="hljs-string">'cart:cart_detail'</span></span>)</code></pre></div>
<p><code>coupon_apply</code> 视图（view）验证优惠券然后把它保存在用户会话（session）中。我们使用 <code>require_POST</code> 装饰器来限制这个视图（view）仅接受 POST 请求。在视图（view）中，我们执行了以下几个任务：</p>
<pre><code class="hljs markdown">1.我们用上传的数据实例化了 <span class="hljs-code">`CouponApplyForm`</span> 然后检查表单是否合法。
<span class="hljs-bullet">2. </span>如果表单是合法的，我们就从表单的 <span class="hljs-code">`cleaned_data`</span> 字典中获取 <span class="hljs-code">`code`</span> 。我们尝试用所给的代码检索 <span class="hljs-code">`Coupon`</span> 对象。我们使用 <span class="hljs-code">`iexact`</span> 字段来对照查找大小写不敏感的精确匹配项。优惠券在当前必须是激活的（<span class="hljs-code">`active=True`</span>）以及必须在当前日期内是有效的。我们使用 Django 的 <span class="hljs-code">`timezone.now()`</span> 函数来获得当前的时区识别时间和日期（time-zone-aware） 然后我们把它和 <span class="hljs-code">`valid_from`</span> 和 <span class="hljs-code">`valid_to`</span> 字段做比较，对这两个日期分别执行 <span class="hljs-code">`lte`</span> （小于等于）运算和 <span class="hljs-code">`gte`</span> （大于等于）运算来进行字段查找。
<span class="hljs-bullet">3. </span>我们在用户的会话中保存优惠券的 <span class="hljs-code">`id`</span>`。
<span class="hljs-bullet">4. </span>我们把用户重定向到 <span class="hljs-code">`cart_detail`</span> URL 来展示应用了优惠券的购物车。</code></pre>
<p>我们需要一个 <code>coupon_apply</code> 视图（view）的 URL 模式。在 <code>coupon</code> 应用路径下创建一个新的文件，命名为 <code>urls.py</code> ，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^apply/$'</span></span>, views.coupon_apply, name<span class="op">=</span><span class="st"><span class="hljs-string">'apply'</span></span>),
]</code></pre></div>
<p>然后，编辑 <code>myshop</code> 项目的主 <code>urls.py</code> 文件，引入 <code>coupons</code> 的 URL 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^coupons/'</span></span>, include(<span class="st"><span class="hljs-string">'coupons.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'coupons'</span></span>)),</code></pre></div>
<p>记得把这个放在 <code>shop.urls</code> 模式之前。</p>
<p>现在编辑 <code>cart</code> 应用的 <code>cart.py</code>，包含以下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> coupons.models <span class="im"><span class="hljs-keyword">import</span></span> Coupon</code></pre></div>
<p>把下面这行代码添加进 <code>Cart</code> 类的 <code>__init__()</code> 方法中来从会话中初始化优惠券：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># store current applied coupon</span></span>
<span class="va">self</span>.coupon_id <span class="op">=</span> <span class="va">self</span>.session.get(<span class="st"><span class="hljs-string">'coupon_id'</span></span>)</code></pre></div>
<p>在这行代码中，我们尝试从当前会话中得到 <code>coupon_id</code> 会话键，然后把它保存在 <code>Cart</code> 对象中。把以下方法添加进 <code>Cart</code> 对象中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="at"><span class="hljs-meta">@property</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">coupon</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">if</span></span> <span class="va">self</span>.coupon_id:
        <span class="cf"><span class="hljs-keyword">return</span></span> Coupon.objects.get(<span class="bu">id</span><span class="op">=</span><span class="va">self</span>.coupon_id)
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va"><span class="hljs-keyword">None</span></span>

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_discount</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">if</span></span> <span class="va">self</span>.coupon:
        <span class="cf"><span class="hljs-keyword">return</span></span> (<span class="va">self</span>.coupon.discount <span class="op">/</span> Decimal(<span class="st"><span class="hljs-string">'100'</span></span>)) <span class="op">\</span>
                <span class="op">*</span> <span class="va">self</span>.get_total_price()
    <span class="cf"><span class="hljs-keyword">return</span></span> Decimal(<span class="st"><span class="hljs-string">'0'</span></span>)

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_total_price_after_discount</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.get_total_price() <span class="op">-</span> <span class="va">self</span>.get_discount()</code></pre></div>
<p>下面是这几个方法：</p>
<ul>
<li><code>coupon()</code>：我们定义这个方法作为 <code>property</code> 。如果购物车包含 <code>coupon_id</code> 函数，就会返回一个带有给定 <code>id</code> 的<code>Coupon</code> 对象</li>
<li><code>get_discount()</code>：如果购物车包含 <code>coupon</code> ，我们就检索它的折扣比率，然后返回购物车中被扣除折扣的总和。</li>
<li><code>get_total_price_after_discount()</code>：返回被减去折扣之后的总价。</li>
</ul>
<p><code>Cart</code> 类现在已经准备好处理应用于当前会话的优惠券了，然后将它应用于相应折扣中。</p>
<p>让我们在购物车详情视图（view）中引入优惠券系统。编辑 <code>cart</code> 应用的 <code>views.py</code> ，然后把下面这一行添加到顶部：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> coupons.forms <span class="im"><span class="hljs-keyword">import</span></span> CouponApplyForm</code></pre></div>
<p>之后，编辑 <code>cart_detail</code> 视图（view），然后添加新的表单：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_detail</span><span class="hljs-params">(request)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> cart:
        item[<span class="st"><span class="hljs-string">'update_quantity_form'</span></span>] <span class="op">=</span> CartAddProductForm(
                    initial<span class="op">=</span>{<span class="st"><span class="hljs-string">'quantity'</span></span>: item[<span class="st"><span class="hljs-string">'quantity'</span></span>],
                    <span class="st"><span class="hljs-string">'update'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>})
    coupon_apply_form <span class="op">=</span> CouponApplyForm()
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
            <span class="st"><span class="hljs-string">'cart/detail.html'</span></span>,
            {<span class="st"><span class="hljs-string">'cart'</span></span>: cart,
            <span class="co"><span class="hljs-string">'coupon_apply_form'</span></span>: coupon_apply_form})</code></pre></div>
<p>编辑 <code>cart</code> 应用的 <code>acrt/detail.html</code> 文件，找到下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"total"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"4"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span></code></pre></div>
<p>把它们换成以下几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> if cart.coupon <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"subtotal"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>Subtotal<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"4"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>
    "<span>{</span><span>{</span> cart.coupon.code <span>}</span><span>}</span>" coupon
    (<span>{</span><span>{</span> cart.coupon.discount <span>}</span><span>}</span>% off)
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"4"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num neg"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    - $<span>{</span><span>{</span> cart.get_discount|floatformat:"2" <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"total"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"4"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    $<span>{</span><span>{</span> cart.get_total_price_after_discount|floatformat:"2" <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span></code></pre></div>
<p>这段代码用于展示可选优惠券以及折扣率。如果购物车中有优惠券，我们就在第一行展示购物车的总价作为 <strong>小计</strong>。然后我们在第二行展示当前可应用于购物车的优惠券。最后，我们调用 <code>cart</code> 对象的 <code>get_total_price_after_discount()</code> 方法来展示折扣了的总价格。</p>
<p>在同一个文件中，在 <code>&lt;/table&gt;</code> 标签之后引入以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Apply a coupon:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">coupons:apply"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>{</span> coupon_apply_form <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Apply"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span></code></pre></div>
<p>我们将会展示一个表单来让用户输入优惠券代码，然后将它应用于当前的购物车当中。</p>
<p>访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000</a> ，向购物车当中添加一个商品，然后在表单中输入你创建的优惠代码来应用你的优惠券。你可以看到购物车像下面这样展示优惠券折扣：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-2.png" alt="django-9-2"></p>
<p>让我们把优惠券添加到购物流程中的下一步。编辑 <code>orders</code> 应用的 <code>orders/order/create.html</code> 模板（template），找到下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span></span>
<span>{</span><span>%</span> for item in cart <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
    <span>{</span><span>{</span> item.quantity <span>}</span><span>}</span>x <span>{</span><span>{</span> item.product.name <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>$<span>{</span><span>{</span> item.total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span></code></pre></div>
<p>把它替换为下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span></span>
<span>{</span><span>%</span> for item in cart <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
    <span>{</span><span>{</span> item.quantity <span>}</span><span>}</span>x <span>{</span><span>{</span> item.product.name <span>}</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>$<span>{</span><span>{</span> item.total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span>{</span><span>%</span> if cart.coupon <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
"<span>{</span><span>{</span> cart.coupon.code <span>}</span><span>}</span>" (<span>{</span><span>{</span> cart.coupon.discount <span>}</span><span>}</span>% off)
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>- $<span>{</span><span>{</span> cart.get_discount|floatformat:"2" <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span></code></pre></div>
<p>订单汇总现在已经包含使用了的优惠券，如果有优惠券的话。现在找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Total: $<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>把他们换成以下一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Total: $<span>{</span><span>{</span> cart.get_total_price_after_discount|floatformat:"2" <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span></code></pre></div>
<p>这样做之后，总价也将会在减去优惠券折扣被计算出来。<br>
访问 <a href="http://127.0.0.1:8000/orders/create/" class="uri">http://127.0.0.1:8000/orders/create/</a> 。你会看到包含使用了优惠券的订单小计：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-3.png" alt="django-9-3"></p>
<p>用户现在可以在购物车当中使用优惠券了。尽管，当用户结账时，我们依然需要在订单中储存优惠券信息。</p>
<h2 id="在订单中使用优惠券"><strong>在订单中使用优惠券</strong></h2>
<p>我们会储存每张订单中使用的优惠券。首先，我们需要修改 <code>Order</code> 模型（model）来储存相关联的 <code>Coupon</code> 对象，如果有这个对象的话。</p>
<p>编辑 <code>orders</code> 应用的 <code>models.py</code> 文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> decimal <span class="im"><span class="hljs-keyword">import</span></span> Decimal
<span class="im"><span class="hljs-keyword">from</span></span> django.core.validators <span class="im"><span class="hljs-keyword">import</span></span> MinValueValidator, <span class="op">\</span>
                                    MaxValueValidator
<span class="im"><span class="hljs-keyword">from</span></span> coupons.models <span class="im"><span class="hljs-keyword">import</span></span> Coupon</code></pre></div>
<p>然后，把下列字段添加进 <code>Order</code> 模型（model）中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">coupon <span class="op">=</span> models.ForeignKey(Coupon,
                            related_name<span class="op">=</span><span class="st"><span class="hljs-string">'orders'</span></span>,
                            null<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>,
                            blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
discount <span class="op">=</span> models.IntegerField(default<span class="op">=</span><span class="dv"><span class="hljs-number">0</span></span>,
                        validators<span class="op">=</span>[MinValueValidator(<span class="dv"><span class="hljs-number">0</span></span>),
                                MaxValueValidator(<span class="dv"><span class="hljs-number">100</span></span>)])</code></pre></div>
<p>这些字段让用户可以在订单中储存可选的优惠券信息，以及优惠券的相应折扣。折扣被存在关联的 <code>Coupon</code> 对象中，但是我们在 <code>Order</code> 模型（model）中引入它以便我们在优惠券被更改或者删除时好保存它。</p>
<p>因为 <code>Order</code> 模型（model）已经被改变了，我们需要创建迁移。执行下面的命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">makemigrations</span></code></pre>
<p>你可以看到如下输出：</p>
<pre><code class="hljs vbnet">Migrations <span class="hljs-keyword">for</span> <span class="hljs-comment">'orders':</span>
    <span class="hljs-number">0002</span>_auto_20150606_1735.py:
        - Add field coupon <span class="hljs-keyword">to</span> <span class="hljs-keyword">order</span>
        - Add field discount <span class="hljs-keyword">to</span> <span class="hljs-keyword">order</span></code></pre>
<p>用下面的命令来执行迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span> <span class="hljs-selector-tag">orders</span></code></pre>
<p>你必须要确保新的迁移已经被应用了。 <code>Order</code> 模型（model）的字段变更现在已经同步到了数据库中。</p>
<p>回到 <code>models.py</code> 文件中，按照如下更改 <code>Order</code> 模型（model）的 <code>get_total_cost()</code> 方法：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_total_cost</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    total_cost <span class="op">=</span> <span class="bu">sum</span>(item.get_cost() <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> <span class="va">self</span>.items.<span class="bu">all</span>())
    <span class="cf"><span class="hljs-keyword">return</span></span> total_cost <span class="op">-</span> total_cost <span class="op">*</span> <span class="op">\</span>
        (<span class="va">self</span>.discount <span class="op">/</span> Decimal(<span class="st"><span class="hljs-string">'100'</span></span>))</code></pre></div>
<p><code>Order</code> 模型（model）的 <code>get_total_cost()</code> 现在已经把使用了的折扣包含在内了，如果有折扣的话。</p>
<p>编辑 <code>orders</code> 应用的 <code>views.py</code> 文件，更改 <code>order_create</code> 视图（view）以便在创建新的订单时保存相关联的优惠券和折扣。找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">order <span class="op">=</span> form.save()</code></pre></div>
<p>把它替换为下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">order <span class="op">=</span> form.save(commit<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>)
<span class="cf"><span class="hljs-keyword">if</span></span> cart.coupon:
    order.coupon <span class="op">=</span> cart.coupon
    order.discount <span class="op">=</span> cart.coupon.discount
order.save()</code></pre></div>
<p>在新的代码中，我们使用 <code>OrderCrateForm</code> 表单的 <code>save()</code> 方法创建了一个 <code>Order</code> 对象。我们使用 <code>commit=False</code> 来避免将它保存在数据库中。如果购物车当中有优惠券，我们就会保存相关联的优惠券和折扣。然后才把 <code>order</code> 对象保存到数据库中。</p>
<p>确保用 <code>python manage.py runserver</code> 运行了开发服务器。使用 <code>./ngrok http:8000</code> 命令来启动 Ngrok 。</p>
<p>在你的浏览器中打开 Ngrok 提供的 URL , 然后用用你创建的优惠券完成一次购物。当你完成一次成功的支付后，有可以访问 <a href="http://127.0.0.1:8000/admin/orders/order/" class="uri">http://127.0.0.1:8000/admin/orders/order/</a> ，检查 <code>order</code> 对象是否包含了优惠券和折扣，如下：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-4.png" alt="django-9-4"></p>
<p>你也可以更改管理界面的订单详情模板（template）和订单的 PDF 账单，以便用展示购物车的方式来展示使用了的优惠券。</p>
<p>下面。我们将会为我们的项目添加国际化（internationalization）。</p>
<h1 id="添加国际化internationalization和本地化localization"><strong>添加国际化（internationalization）和本地化（localization）</strong></h1>
<p>Django 提供了完整的国际化和本地化的支持。这使得你可以把你的项目翻译为多种语言以及处理地区特性的 日期，时间，数字，和时区 的格式。让我们一起来搞清楚国际化和本地化的区别。国际化（通常简写为：<strong>i18n</strong>）是为让软件适应潜在的不同语言和多种语言的使用做的处理，这样它就不是以某种特定语言或某几种语言为硬编码的软件了。本地化（简写为：<strong>l10n</strong>）是对软件的翻译以及使软件适应某种特定语言的处理。Django 自身已经使用自带的国际化框架被翻译为了50多种语言。</p>
<h2 id="使用-django-国际化"><strong>使用 Django 国际化</strong></h2>
<p>国际化框架让你可以容易的在 Python 代码和模板（template）中标记需要翻译的字符串。它依赖于 GNU 文本获取集来生成和管理消息文件。<strong>消息文件</strong>是一个表示一种语言的纯文本文件。它包含某一语言的一部分或者所有的翻译字符串。消息文件有 <code>.po</code> 扩展名。</p>
<p>一旦翻译完成，信息文件就会被编译一遍快速的连接到被翻译的字符串。编译后的翻译文件的扩展名为 <code>.mo</code> 。</p>
<h2 id="国际化和本地化设置"><strong>国际化和本地化设置</strong></h2>
<p>Django 提供了几种国际化的设置。下面是几种最有关联的设置：</p>
<ul>
<li><code>USE_I18N</code>：布尔值。用于设定 Django 翻译系统是否启动。默认值为 <code>True</code>。<br>
-<code>USE_L10N</code>：布尔值。表示本地格式化是否启动。当被激活式，本地格式化被用于展示日期和数字。默认为 <code>False</code> 。</li>
<li><code>USE_TZ</code>：布尔值。用于指定日期和时间是否是时区别（timezone-aware）。</li>
<li><code>LANGUAGE_CODE</code>：项目的默认语言。在标准的语言 ID 格式中，比如，<code>en-us</code> 是美式英语，<code>en-gb</code> 是英式英语。这个设置要 <code>USE_I18N</code> 为 <code>True</code> 才能生效。你可以在这个网站找到一个合法的语言 ID 表：<code>http://www.i18nguy.com/unicode/language-identifiers.html</code> 。</li>
<li><code>LANGUAGES</code>：包含项目可用语言的元组。它们由包含 <strong>语言代码</strong> 和 <strong>语言名字</strong> 的双元组构成的。你可以在 <code>django.conf.global_settions</code> 里看到可用语言的列表。当你选择你的网站将会使用哪一种语言时，你就把 <code>LANGUAGES</code> 设置为那个列表的子列表。</li>
<li><code>LOCAL_PATHS</code>：Django 寻找包含翻译的信息文件的路径列表。</li>
<li><code>TIME_ZONE</code>：代表项目时区的字符串。当你使用 <code>startproject</code> 命令创建新项目时它被设置为 <code>UTC</code> 。你也可以把它设置为其他的时区，比如 <code>Europe/Madrid</code> 。</li>
</ul>
<p>这是一些可用的国际化和本地化的设置。你可以在这个网站找到全部的（设置）列表： <a href="https://docs.djangoproject.com/en/1.8/ref/settings/#globalization-i18n-l10n" class="uri">https://docs.djangoproject.com/en/1.8/ref/settings/#globalization-i18n-l10n</a> 。</p>
<h2 id="国际化管理命令"><strong>国际化管理命令</strong></h2>
<p>Django 使用 <code>manage.py</code> 或者 <code>django-admin.py</code> 工具包管理翻译，包含以下命令：</p>
<ul>
<li><code>makemessages</code>：运行于源代码树中，寻找所有被用于翻译的字符串，然后在 <code>locale</code> 路径中创建或更新 <code>.po</code> 信息文件。每一种语言创建一个单一的 <code>.po</code> 文件。</li>
<li><code>compilemessages</code>： 把扩展名为 <code>.po</code> 的信息文件编译为用于检索翻译的 <code>.mo</code> 文件。</li>
</ul>
<p>你需要文本获取工具集来创建，更新，以及编译信息文件。大部分的 Linux 发行版都包含文本获取工具集。如果你在使用 Mac OS X，用 Honebrew (<code>http://brew.sh</code>)是最简单的安装它的方法，使用以下命令 ：<code>brew install gettext</code>。你或许也需要将它和命令行强制连接 <code>brew link gettext --force</code> 。对于 Windows ，安装步骤如下 ： <a href="https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#gettext-on-windows" class="uri">https://docs.djangoproject.com/en/1.8/topics/i18n/translation/#gettext-on-windows</a> 。</p>
<h2 id="怎么把翻译添加到-django-项目中"><strong>怎么把翻译添加到 Django 项目中</strong></h2>
<p>让我们看看国际化项目的过程。我们需要像下面这样做：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>我们在 Python 代码和模板（template）中中标记需要翻译的字符串。
<span class="hljs-bullet">2. </span>我们运行 <span class="hljs-code">`makemessages`</span> 命令来创建或者更新包含所有翻译字符串的信息文件。
<span class="hljs-bullet">3. </span>我们翻译包含在信息文件中的字符串，然后使用  <span class="hljs-code">`compilemessages`</span> 编译他们。</code></pre>
<h2 id="django-如何决定当前语言"><strong>Django 如何决定当前语言</strong></h2>
<p>Django 配备有一个基于请求数据的中间件，这个中间件用于决定当前语言是什么。这个中间件是位于 <code>django.middleware.locale.localMiddleware</code> 的 <code>LocaleMiddleware</code> ,它执行下面的任务：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>如果使用 <span class="hljs-code">`i18_patterns`</span> —— 这是你使用的被翻译的 URL 模式，它在被请求的 URL 中寻找一个语言前缀来决定当前语言。
<span class="hljs-bullet">2. </span>如果没有找到语言前缀，就会在当前用户会话中寻找当前的 <span class="hljs-code">`LANGUAGE_SESSION_KEY`</span> 。
<span class="hljs-bullet">3. </span>如果在会话中没有设置语言，就会寻找带有当前语言的 cookie 。这个 cookie 的定制名可在 <span class="hljs-code">`LANGUAGE_COOKIE_NAME`</span> 中设置。默认地，这个 cookie 的名字是 <span class="hljs-code">`django_language`</span> 。
<span class="hljs-bullet">4. </span>如果没有找到 cookie，就会在请求 HTTP 头中寻找 <span class="hljs-code">`Accept-Language`</span> 。
<span class="hljs-bullet">5. </span>如果 <span class="hljs-code">`Accept-Language`</span> 头中没有指定语言，Django 就使用在 <span class="hljs-code">`LANGUAGE_CODE`</span> 设置中定义的语言。</code></pre>
<p>默认的，Django 会使用 <code>LANGUAGE_OCDE</code> 中设置的语言，除非你正在使用 <code>LocaleMiddleware</code> 。上述操作进会在使用这个中间件时生效。</p>
<h2 id="为我们的项目准备国际化"><strong>为我们的项目准备国际化</strong></h2>
<p>让我们的项目准备好使用不同的语言吧。我们将要为我们的商店创建英文版和西班牙语版。编辑项目中的 <code>settings.py</code> 文件，添加下列 <code>LANGUAGES</code> 设置。把它放在 <code>LANGUAGE_OCDE</code> 旁边：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">LANGUAGES <span class="op">=</span> (
        (<span class="st"><span class="hljs-string">'en'</span></span>, <span class="st"><span class="hljs-string">'English'</span></span>),
        (<span class="st"><span class="hljs-string">'es'</span></span>, <span class="st"><span class="hljs-string">'Spanish'</span></span>),
)</code></pre></div>
<p><code>LANGUAGES</code> 设置包含两个由语言代码和语言名的元组构成，比如 <code>en-us</code> 或 <code>en-gb</code> ，或者一般的设置为 <code>en</code> 。这样设置之后，我们指定我们的应用只会对英语和西班牙语可用。如果我们不指定 <code>LANGUAGES</code> 设置，网站将会对所有 Django 的翻译语言有效。</p>
<p>确保你的 <code>LANGUAGE_OCDE</code> 像如下设置：</p>
<pre><code class="hljs ini"><span class="hljs-attr">LANGUAGE_OCDE</span> = <span class="hljs-string">'en'</span></code></pre>
<p>把 <code>django.middleware.locale.LocaleMiddleware</code> 添加到 <code>MIDDLEWARE_CLASSES</code> 设置中。确保这个设置在 <code>SessionsMiddleware</code> 之后，因为 <code>LocaleMiddleware</code> 需要使用会话数据。它同样必须放在 <code>CommonMiddleware</code> 之前，因为后者需要一种激活了的语言来解析请求 URL 。<code>MIDDLEWARE_CLASSES</code> 设置看起来应该如下：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">MIDDLEWARE_CLASSES <span class="op">=</span> (
    <span class="st"><span class="hljs-string">'django.contrib.sessions.middleware.SessionMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.locale.LocaleMiddleware'</span></span>,
    <span class="co"><span class="hljs-string">'django.middleware.common.CommonMiddleware'</span></span>,
    <span class="co"><span class="hljs-comment"># ...</span></span>
)</code></pre></div>
<blockquote>
<p>中间件的顺序非常重要，因为每个中间件所依赖的数据可能是由其他中间件处理之后的才获得的。中间件按照 <code>MIDDLEWARE_CLASSES</code> 的顺序应用在每个请求上，以及反序应用于响应中。</p>
</blockquote>
<p>在主项目路径下，在 <code>manage.py</code> 文件同级，创建以下文件结构：</p>
<pre><code class="hljs">locale/
    en/
    es/</code></pre>
<p><code>locale</code> 路径是放置应用信息文件的地方。再次编辑 <code>settings.py</code> ，然后添加以下设置：</p>
<pre><code class="hljs lisp">LOCALE_PATHS = (
    <span class="hljs-name">os</span>.path.join(<span class="hljs-name">BASE_DIR</span>, 'locale/'),
)</code></pre>
<p><code>LOCALE_PATHS</code> 设置指定了 Django 寻找翻译文件的路径。第一个路径有最高优先权。</p>
<p>当你在你的项目路径下使用 <code>makemessages</code> 命令时，信息文件将会在 <code>locale</code> 路径下生成。尽管，对于包含 <code>locale</code> 路径的应用，信息文件就会保存在这个应用的 <code>locale</code> 路径中。</p>
<h2 id="翻译-python-代码"><strong>翻译 Python 代码</strong></h2>
<p>我们翻译在 Python 代码中的字母，你可以使用 <code>django.utils.translation</code> 中的 <code>gettext()</code> 函数来标记需要翻译的字符串。这个函数翻译信息然后返回一个字符串。约定俗成的用法是导入这个函数后把它命名为 <code>_</code> （下划线）。</p>
<p>你可以在这个网站找到所有关于翻译的文档 ： <a href="https://docs.djangoproject.com/en/1.8/topics/i18n/translation/" class="uri">https://docs.djangoproject.com/en/1.8/topics/i18n/translation/</a> 。</p>
<h2 id="标准翻译"><strong>标准翻译</strong></h2>
<p>下面的代码展示了如何标记一个翻译字符串：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext <span class="im"><span class="hljs-keyword">as</span></span> _
output <span class="op">=</span> _(<span class="st"><span class="hljs-string">'Text to be translated.'</span></span>)</code></pre></div>
<h2 id="惰性翻译lazy-translation"><strong>惰性翻译（Lazy translation）</strong></h2>
<p>Django 对所有的翻译函数引入了惰性（lazy）版，这些惰性函数都带有后缀 <code>_lazy()</code> 。当使用惰性函数时，字符串在值被连接时就会被翻译，而不是在被调用时翻译（这也是它为什么被惰性翻译的原因）。惰性函数迟早会派上用场，特别是当标记字符串在模型（model）加载的执行路径中时。</p>
<blockquote>
<p>使用 <code>gettext_lazy()</code> 而不是 <code>gettext()</code> ，字符串就会在连接到值的时候被翻译而不会在函数调用的时候被翻译。Django 为所有的翻译都提供了惰性版本。</p>
</blockquote>
<h2 id="翻译引入的变量"><strong>翻译引入的变量</strong></h2>
<p>标记的翻译字符串可以包含占位符来引入翻译中的变量。下面就是一个带有占位符的翻译字字符串的例子：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext <span class="im"><span class="hljs-keyword">as</span></span> _
month <span class="op">=</span> _(<span class="st"><span class="hljs-string">'April'</span></span>)
day <span class="op">=</span> <span class="st"><span class="hljs-string">'14'</span></span>
output <span class="op">=</span> _(<span class="st"><span class="hljs-string">'Today is </span></span><span class="sc"><span class="hljs-string">%(month)s</span></span><span class="st"><span class="hljs-string"> </span></span><span class="sc"><span class="hljs-string">%(day)s</span></span><span class="st"><span class="hljs-string">'</span></span>) <span class="op">%</span> {<span class="st"><span class="hljs-string">'month'</span></span>: month,
                                            <span class="co"><span class="hljs-string">'day'</span></span>: day}</code></pre></div>
<p>通过使用占位符，你可以重新排序文字变量。举个例子，以前的英文版本是 'Today is April 14' ，但是西班牙的版本是这样的 'Hoy es 14 de Abril' 。当你的翻译字符串有多于一个参数时，我们总是使用字符串插值来代替位置插值</p>
<h2 id="翻译中的复数形式"><strong>翻译中的复数形式</strong></h2>
<p>对于复数形式，你可以使用 <code>gettext()</code> 和 <code>gettext_lazy()</code> 。这些函数基于一个可以表示对象数量的参数来翻译单数和复数形式。下面这个例子展示了如何使用它们：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">output <span class="op">=</span> ngettext(<span class="st"><span class="hljs-string">'there is </span></span><span class="sc"><span class="hljs-string">%(count)d</span></span><span class="st"><span class="hljs-string"> product'</span></span>,
                <span class="co"><span class="hljs-string">'there are %(count)d products'</span></span>,
                count) <span class="op">%</span> {<span class="st"><span class="hljs-string">'count'</span></span>: count}</code></pre></div>
<p>现在你已经基本知道了如何在你的 Python 代码中翻译字符了，现在是把翻译应用到项目中的时候了。</p>
<h2 id="翻译你的代码"><strong>翻译你的代码</strong></h2>
<p>编辑项目中的 <code>settings.py</code> ，导入 <code>gettext_lazy()</code> 函数，然后像下面这样修改 <code>LANGUAGES</code> 的设置：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

LANGUAGES <span class="op">=</span> (
    (<span class="st"><span class="hljs-string">'en'</span></span>, _(<span class="st"><span class="hljs-string">'English'</span></span>)),
    (<span class="st"><span class="hljs-string">'es'</span></span>, _(<span class="st"><span class="hljs-string">'Spanish'</span></span>)),
)</code></pre></div>
<p>我们使用 <code>gettext_lazy()</code> 而不是 <code>gettext()</code> 来避免循环导入，这样就可以在语言名被连接时就翻译它们。</p>
<p>在你的项目路径下执行下面的命令：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">django<span class="op">-</span>admin makemessages <span class="op">--</span><span class="bu">all</span></code></pre></div>
<p>你可以看到如下输出：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">processing</span> locale es
processing locale en</code></pre>
<p>看下 <code>locale/</code> 路径。你可以看到如下文件结构：</p>
<pre><code class="hljs">en/
    LC_MESSAGES/
        django.po
es/
    LC_MESSAGES/
        django.po</code></pre>
<p><code>.po</code> 文件已经为每一个语言创建好了。用文本编辑器打开 <code>es/LC_MESSAGES/django.po</code> 。在文件的末尾，你可以看到如下几行：</p>
<pre><code class="hljs nginx"><span class="hljs-comment">#: settings.py:104</span>
<span class="hljs-attribute">msgid</span> <span class="hljs-string">"English"</span>
msgstr <span class="hljs-string">""</span>

<span class="hljs-comment">#: settings.py:105</span>
msgid <span class="hljs-string">"Spanish"</span>
msgstr <span class="hljs-string">""</span></code></pre>
<p>每一个翻译字符串前都有一个显示该文件详情的注释以及它在哪一行被找到。每个翻译都包含两个字符串：</p>
<ul>
<li><code>msgid</code>：在源代码中的翻译字符串</li>
<li><code>msgstr</code>：对应语言的翻译，默认为空。这是你输入所给字符串翻译的地方。</li>
</ul>
<p>按照如下，根据所给的 <code>msgid</code> 在 <code>msgtsr</code> 中填写翻译：</p>
<pre><code class="hljs nginx"><span class="hljs-comment">#: settings.py:104</span>
<span class="hljs-attribute">msgid</span> <span class="hljs-string">"English"</span>
msgstr <span class="hljs-string">"Inglés"</span>

<span class="hljs-comment">#: settings.py:105</span>
msgid <span class="hljs-string">"Spanish"</span>
msgstr <span class="hljs-string">"Español"</span></code></pre>
<p>保存修改后的信息文件，打开 shell ，运行下面的命令：</p>
<pre><code class="hljs">django-admin compilemessages</code></pre>
<p>如果一切顺利，你可以看到像如下的输出：</p>
<pre><code class="hljs delphi">processing <span class="hljs-keyword">file</span> django.po <span class="hljs-keyword">in</span> myshop/locale/en/LC_MESSAGES
processing <span class="hljs-keyword">file</span> django.po <span class="hljs-keyword">in</span> myshop/locale/es/LC_MESSAGES</code></pre>
<p>输出给出了被编译的信息文件的相关信息。再看一眼 <code>locale</code> 路径的 <code>myshop</code> 。你可以看到下面的文件：</p>
<pre><code class="hljs">en/
    LC_MESSAGES/
        django.mo
        django.po
es/
    LC_MESSAGES/
        django.mo
        django.po</code></pre>
<p>你可以看到每个语言的 <code>.mo</code> 的编译文件已经生成了。</p>
<p>我们已经翻译了语言本身的名字。现在让我们翻译展示在网站中模型（model）字段的名字。编辑 <code>orders</code> 应用的 <code>models.py</code> ，为 <code>Order</code> 模型（model）添加翻译的被标记名：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Order</span><span class="hljs-params">(models.Model)</span>:</span>
    first_name <span class="op">=</span> models.CharField(_(<span class="st"><span class="hljs-string">'first name'</span></span>),
                                max_length<span class="op">=</span><span class="dv"><span class="hljs-number">50</span></span>)
    last_name <span class="op">=</span> models.CharField(_(<span class="st"><span class="hljs-string">'last name'</span></span>),
                                max_length<span class="op">=</span><span class="dv"><span class="hljs-number">50</span></span>)
    email <span class="op">=</span> models.EmailField(_(<span class="st"><span class="hljs-string">'e-mail'</span></span>),)
    address <span class="op">=</span> models.CharField(_(<span class="st"><span class="hljs-string">'address'</span></span>),
                                max_length<span class="op">=</span><span class="dv"><span class="hljs-number">250</span></span>)
    postal_code <span class="op">=</span> models.CharField(_(<span class="st"><span class="hljs-string">'postal code'</span></span>),
                                max_length<span class="op">=</span><span class="dv"><span class="hljs-number">20</span></span>)
    city <span class="op">=</span> models.CharField(_(<span class="st"><span class="hljs-string">'city'</span></span>),
                        max_length<span class="op">=</span><span class="dv"><span class="hljs-number">100</span></span>)
<span class="co"><span class="hljs-comment">#...</span></span></code></pre></div>
<p>我们添加为当用户下一个新订单时展示的字段添加了名字，它们分别是 <code>first_name</code> ,<code>last_name</code>, <code>email</code>, <code>address</code>, <code>postal_code</code> ,<code>city</code>。记住，你也可以使用每个字段的 <code>verbose_name</code> 属性。</p>
<p>在 <code>orders</code> 应用路径内创建以下路径：</p>
<pre><code class="hljs">locale/
    en/
    es/</code></pre>
<p>通过创建 <code>locale</code> 路径，这个应用的翻译字符串就会储存在这个路径下的信息文件里，而不是在主信息文件里。这样做之后，你就可以为每个应用生成独自的翻译文件。</p>
<p>在项目路径下打开 shell ，运行下面的命令：</p>
<pre><code class="hljs fortran">django-admin makemessages --<span class="hljs-built_in">all</span></code></pre>
<p>你可以看到如下输出：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">processing</span> locale es
processing locale en</code></pre>
<p>用文本编辑器打开 <code>es/LC_MESSAGES/django.po</code> 文件。你将会看到每一个模型（model）的翻译字符串。为每一个所给的 <code>msgid</code> 字符串填写 <code>msgstr</code> 的翻译：</p>
<pre><code class="hljs nginx"><span class="hljs-comment">#: orders/models.py:10</span>
<span class="hljs-attribute">msgid</span> <span class="hljs-string">"first name"</span>
msgstr <span class="hljs-string">"nombre"</span>

<span class="hljs-comment">#: orders/models.py:12</span>
msgid <span class="hljs-string">"last name"</span>
msgstr <span class="hljs-string">"apellidos"</span>

<span class="hljs-comment">#: orders/models.py:14</span>
msgid <span class="hljs-string">"e-mail"</span>
msgstr <span class="hljs-string">"e-mail"</span>

<span class="hljs-comment">#: orders/models.py:15</span>
msgid <span class="hljs-string">"address"</span>
msgstr <span class="hljs-string">"dirección"</span>

<span class="hljs-comment">#: orders/models.py:17</span>
msgid <span class="hljs-string">"postal code"</span>
msgstr <span class="hljs-string">"código postal"</span>

<span class="hljs-comment">#: orders/models.py:19</span>
msgid <span class="hljs-string">"city"</span>
msgstr <span class="hljs-string">"ciudad"</span></code></pre>
<p>在你添加完翻译之后，保存文件。</p>
<p>在文本编辑器内，你可以使用 Poedit 来编辑翻译。 Poedit 是一个用来编辑翻译的软件，它使用 gettext 。它有 Linux ，Windows，Mac OS X 版，你可以在这个网站下载 Poedit : <a href="http://poedit.net/" class="uri">http://poedit.net/</a> 。</p>
<p>让我们来翻译项目中的表单吧。 <code>orders</code> 应用的 <code>OrderCrateForm</code> 还没有被翻译，因为它是一个 <code>ModelForm</code> ,使用 <code>Order</code> 模型（model）的 <code>verbose_name</code> 属性作为每个字段的标签。我们将会翻译 <code>cart</code> 和 <code>coupons</code> 应用的表单。</p>
<p>编辑 <code>cart</code> 应用路径下的 <code>forms.py</code> 文件，给 <code>CartAddProductForm</code> 的 <code>quantity</code> 字段添加一个 <code>label</code> 属性，然后按照如下标记这个需要翻译的字段：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

PRODUCT_QUANTITY_CHOICES <span class="op">=</span> [(i, <span class="bu">str</span>(i)) <span class="cf"><span class="hljs-keyword">for</span></span> i <span class="op"><span class="hljs-keyword">in</span></span> <span class="bu">range</span>(<span class="dv"><span class="hljs-number">1</span></span>, <span class="dv"><span class="hljs-number">21</span></span>)]

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CartAddProductForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    quantity <span class="op">=</span> forms.TypedChoiceField(
                        choices<span class="op">=</span>PRODUCT_QUANTITY_CHOICES,
                        <span class="bu">coerce</span><span class="op">=</span><span class="bu">int</span>,
                        label<span class="op">=</span>_(<span class="st"><span class="hljs-string">'Quantity'</span></span>))
    update <span class="op">=</span> forms.BooleanField(required<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>,
                        initial<span class="op">=</span><span class="va"><span class="hljs-keyword">False</span></span>,
                        widget<span class="op">=</span>forms.HiddenInput)</code></pre></div>
<p>编辑 <code>coupons</code> 应用的 <code>forms.py</code> ，按照如下翻译 <code>CouponApplyForm</code> ：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CouponApplyForm</span><span class="hljs-params">(forms.Form)</span>:</span>
    code <span class="op">=</span> forms.CharField(label<span class="op">=</span>_(<span class="st"><span class="hljs-string">'Coupon'</span></span>))</code></pre></div>
<h2 id="翻译模板templates"><strong>翻译模板（templates）</strong></h2>
<p>Django 提供了 <code><span>{</span><span>%</span> trans <span>%</span><span>}</span></code> 和 <code><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span></code> 模板（template）标签来翻译模板（template）中的字符串。为了使用翻译模板（template）标签，你需要在你的模板（template）顶部添加 <code><span>{</span><span>%</span>  load i18n <span>%</span><span>}</span></code> 来载入她们。</p>
<h2 id="trans-模板template标签"><strong><span>{</span><span>%</span> trans <span>%</span><span>}</span>模板（template）标签</strong></h2>
<p><code><span>{</span><span>%</span> trans <span>%</span><span>}</span></code>模板（template）标签让你可以标记需要翻译的字符串，常量，或者是参数。在内部，Django 对所给文本执行 <code>gettext()</code> 。这是如何标记模板（template）中的翻译字符串：</p>
<pre><code class="hljs django"><span class="xml"></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">trans</span></span> "Text to be translated" <span>%</span><span>}</span></span><span class="xml"></span></code></pre>
<p>你可使用 <code>as</code> 来储存你在模板（template）内使用的全局变量里的被翻译内容。下面这个例子保存了一个变量中叫做 <code>greeting</code> 的被翻译文本：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> trans "Hello!" as greeting <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span>{</span><span>{</span> greeting <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span></code></pre></div>
<p><code><span>{</span><span>%</span> trans <span>%</span><span>}</span></code> 标签对于简单的翻译字符串是很有用的，但是它不能包含变量的翻译内容。</p>
<h2 id="blocktrans-模板template标签"><strong><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span>模板（template）标签</strong></h2>
<p><code><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span></code> 模板（template）标签让你可以标记含有占位符的变量和字符的内容。线面这个例子展示了如何使用 <code><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span></code> 标签来标记包含一个 <code>name</code> 变量的翻译内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span>Hello <span>{</span><span>{</span> name <span>}</span><span>}</span>!<span>{</span><span>%</span> endblocktrans <span>%</span><span>}</span></code></pre></div>
<p>你可以用 <code>with</code> 来引入模板（template）描述，比如连接对象的属性或者应用模板（template）的变量过滤器。你必须为他们用占位符。你不能在 <code>blocktrans</code> 内连接任何描述或者对象属性。下面的例子展示了如何使用 <code>with</code> 来引入一个应用了 <code>capfirst</code> 过滤器的对象属性：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> blocktrans with name=user.name|capfirst <span>%</span><span>}</span>
Hello <span>{</span><span>{</span> name <span>}</span><span>}</span>!
<span>{</span><span>%</span> endblocktrans <span>%</span><span>}</span></code></pre></div>
<blockquote>
<p>当你的字符串中含有变量时使用 <code><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span></code> 代替 <code><span>{</span><span>%</span> trans <span>%</span><span>}</span></code> 。</p>
</blockquote>
<h2 id="翻译商店模板template"><strong>翻译商店模板（template）</strong></h2>
<p>编辑 <code>shop</code> 应用的 <code>shop/base.html</code> 。确保你已经在顶部载入了 <code>i18n</code> 标签，然后按照如下标记翻译字符串：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load i18n <span>%</span><span>}</span>
<span>{</span><span>%</span> load static <span>%</span><span>}</span>
<span class="dt"><span class="hljs-meta">&lt;!DOCTYPE </span></span><span class="hljs-meta">html</span><span class="dt"><span class="hljs-meta">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">head</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">meta</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">charset</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"utf-8"</span></span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">title</span>&gt;</span></span>
<span>{</span><span>%</span> block title <span>%</span><span>}</span><span>{</span><span>%</span> trans "My shop" <span>%</span><span>}</span><span>{</span><span>%</span> endblock <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">title</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">link</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">css</span>/<span class="hljs-attr">base.css</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">rel</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"stylesheet"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">head</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"header"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"logo"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>%</span> trans "My shop" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"subheader"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>%</span> with total_items=cart|length <span>%</span><span>}</span>
<span>{</span><span>%</span> if cart|length &gt; 0 <span>%</span><span>}</span>
<span>{</span><span>%</span> trans "Your cart" <span>%</span><span>}</span>:
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">cart:cart_detail"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>%</span> blocktrans with total_items_plural=total_
items|pluralize
total_price=cart.get_total_price <span>%</span><span>}</span>
<span>{</span><span>{</span> total_items <span>}</span><span>}</span> item<span>{</span><span>{</span> total_items_plural <span>}</span><span>}</span>,
$<span>{</span><span>{</span> total_price <span>}</span><span>}</span>
<span>{</span><span>%</span> endblocktrans <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span>{</span><span>%</span> else <span>%</span><span>}</span>
<span>{</span><span>%</span> trans "Your cart is empty." <span>%</span><span>}</span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span>{</span><span>%</span> endwith <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"content"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>%</span> block content <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre></div>
<p>注意展示在购物车小计的 <code><span>{</span><span>%</span> blocktrans <span>%</span><span>}</span></code> 标签。购物车小计在之前是这样的：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>{</span> total_items <span>}</span><span>}</span> item<span>{</span><span>{</span> total_items|pluralize <span>}</span><span>}</span>,
$<span>{</span><span>{</span> cart.get_total_price <span>}</span><span>}</span></code></pre></div>
<p>我们使用 <code><span>{</span><span>%</span> blocktrans with ... <span>%</span><span>}</span></code> 来使用 <code>total_ items|pluralize</code> （模板（template）标签生效的地方）和 <code>cart_total_price</code> （连接对象方法的地方）的占位符：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> blocktrans with total_items_plural=total_items|pluralize
total_price=cart.get_total_price <span>%</span><span>}</span>
<span>{</span><span>{</span> total_items <span>}</span><span>}</span> item<span>{</span><span>{</span> total_items_plural <span>}</span><span>}</span>,
$<span>{</span><span>{</span> total_price <span>}</span><span>}</span>
<span>{</span><span>%</span> endblocktrans <span>%</span><span>}</span></code></pre></div>
<p>下面，编辑 <code>shop</code> 应用的 <code>shop/product/detail.html</code> 模板（template）然后在顶部载入 <code>i18n</code> 标签，但是它必须位于 <code><span>{</span><span>%</span> extends <span>%</span><span>}</span></code> 标签的下面：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load i18n <span>%</span><span>}</span></code></pre></div>
<p>找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"Add to cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span></code></pre></div>
<p>把它替换为下面这一行：</p>
<pre><code class="hljs django"><span class="xml"><span class="hljs-tag">&lt;<span class="hljs-name">input</span> <span class="hljs-attr">type</span>=<span class="hljs-string">"submit"</span> <span class="hljs-attr">value</span>=<span class="hljs-string">"</span></span></span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">trans</span></span> "Add to cart" <span>%</span><span>}</span></span><span class="xml"><span class="hljs-tag"><span class="hljs-string">"</span>&gt;</span></span></code></pre>
<p>现在翻译 <code>orders</code> 应用模板（template）。编辑 <code>orders</code> 应用的 <code>orders/order/create.html</code> 模板（template），然后标记翻译文本：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> load i18n <span>%</span><span>}</span>
<span>{</span><span>%</span> block title <span>%</span><span>}</span>
<span>{</span><span>%</span> trans "Checkout" <span>%</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>
<span>{</span><span>%</span> block content <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span><span>{</span><span>%</span> trans "Checkout" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"order-info"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span><span>{</span><span>%</span> trans "Your order" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span>&gt;</span></span>
<span>{</span><span>%</span> for item in cart <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>{</span> item.quantity <span>}</span><span>}</span>x <span>{</span><span>{</span> item.product.name <span>}</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>$<span>{</span><span>{</span> item.total_price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span>{</span><span>%</span> if cart.coupon <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> blocktrans with code=cart.coupon.code
discount=cart.coupon.discount <span>%</span><span>}</span>
"<span>{</span><span>{</span> code <span>}</span><span>}</span>" (<span>{</span><span>{</span> discount <span>}</span><span>}</span>% off)
<span>{</span><span>%</span> endblocktrans <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span>&gt;</span></span>- $<span>{</span><span>{</span> cart.get_discount|floatformat:"2" <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span>{</span><span>%</span> trans "Total" <span>%</span><span>}</span>: $<span>{</span><span>{</span>
cart.get_total_price_after_discount|floatformat:"2" <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">form</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">action</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"."</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">method</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"post"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"order-form"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>{</span> form.as_p <span>}</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"submit"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> trans "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">Place</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">order</span></span></span><span class="er"><span class="hljs-tag">"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span>{</span><span>%</span> csrf_token <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">form</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>看看本章中下列文件中的代码，看看字符串是如何被标记的：</p>
<ul>
<li><code>shop</code> 应用：<code>shop/product/list.html</code></li>
<li><code>orders</code> 应用：<code>orders/order/created.html</code></li>
<li><code>cart</code> 应用：<code>cart/detail.html</code></li>
</ul>
<p>让位我们更新信息文件来引入新的翻译字符串。打开 shell ，运行下面的命令：</p>
<pre><code class="hljs fortran">django-admin makemessages --<span class="hljs-built_in">all</span></code></pre>
<p><code>.po</code> 文件已经在 <code>myshop</code> 项目的 <code>locale</code> 路径下，你将看到 <code>orders</code> 应用现在已经包含我们标记的所有需要翻译的字符串。</p>
<p>编辑项目和<code>orderes</code> 应用中的 <code>.po</code> 翻译文件，然后引入西班牙语翻译。你可以参考本章中翻译了的 <code>.po</code> 文件：</p>
<p>在项目路径下打开 shell ，然后运行下面的命令：</p>
<pre><code class="hljs dos"><span class="hljs-built_in">cd</span> orders/
django-admin compilemessages
<span class="hljs-built_in">cd</span> ../</code></pre>
<p>我们已经编译了 <code>orders</code> 应用的翻译。</p>
<p>运行下面的命令，这样应用中不包含 <code>locale</code> 路径的翻译就被包含进了项目的信息文件中：</p>
<pre><code class="hljs">django-admin compilemessages</code></pre>
<h2 id="使用-rosetta-翻译交互界面"><strong>使用 Rosetta 翻译交互界面</strong></h2>
<p>Rosetta 是一个让你可以编辑翻译的第三方应用，它有类似 Django 管理站点的交互界面。Rosetta 让编辑 <code>.po</code> 文件和更新它变得很简单，让我们把它添加进项目中：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-rosetta==<span class="hljs-number">0.7</span>.<span class="hljs-number">6</span></code></pre>
<p>然后，把 <code>rosetts</code> 添加进项目中的<code>setting.py</code>文件中的<code>INSTALLED_APPS</code> 设置中。</p>
<p>你需要把 Rosetta 的 URL 添加进主 URL 配置中。编辑项目中的主 <code>urls.py</code> 文件，把下列 URL 模式添加进去：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^rosetta/'</span></span>, include(<span class="st"><span class="hljs-string">'rosetta.urls'</span></span>)),</code></pre></div>
<p>确保你把它放在了 <code>shop.urls</code> 模式之前以避免错误的匹配。</p>
<p>访问 <a href="http://127.0.0.1:8000/admin/" class="uri">http://127.0.0.1:8000/admin/</a> ,然后使用超级管理员账户登录。然后导航到 <a href="http://127.0.0.1:8000/rosetta/" class="uri">http://127.0.0.1:8000/rosetta/</a> 。你可以看到当前语言列表：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-5.png" alt="django-9-5"></p>
<p>在 <strong>Filter</strong> 模块，点击 <strong>All</strong> 来展示所有可用的信息文件，包括属于 <code>orders</code> 应用的信息文件。点击<strong>Spanish</strong>模块下的 <strong>Myshop</strong> 链接来编辑西班牙语翻译。你可以看到翻译字符串的列表：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-6.png" alt="django-9-6"></p>
<p>你可以在 <strong>Spanish</strong> 行下输入翻译。<strong>Occurences</strong> 行展示了代码中翻译字符串被找到的文件和行数、</p>
<p>包含占位符的翻译看起来像这样：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-7.png" alt="django-9-7"></p>
<p>Rosetta 使用了不同的背景色来展示占位符。当你翻译内容时，确保你没有翻译占位符。比如，用下面这个字符串举例：</p>
<pre><code class="hljs ruby"><span class="hljs-string">%(total_items)</span>s item<span class="hljs-string">%(total_items_plural)</span>s, $%(total_price)s</code></pre>
<p>它翻译为西班牙语之后长得像这样：</p>
<pre><code class="hljs ruby"><span class="hljs-string">%(total_items)</span>s producto<span class="hljs-string">%(total_items_plural)</span>s, $%(total_price)s</code></pre>
<p>你可以看看本章项目中使用相同西班牙语翻译的文件源代码。</p>
<p>当你完成编辑翻译后，点击<strong>Save and translate next block</strong> 按钮来保存翻译到 <code>.po</code> 文件中。Rosetta 在你保存翻译时编辑信息文件，所以并不需要你运行 <code>compilemessages</code> 命令。尽管，Rosetta 需要 <code>locale</code> 路径的写入权限来写入信息文件。确保路径有有效权限。</p>
<p>如果你想让其他用户编辑翻译，访问：<a href="http://127.0.0.1:8000/admin/auth/group/add/,%E7%84%B6%E5%90%8E%E5%88%9B%E5%BB%BA%E4%B8%80%E4%B8%AA%E5%90%8D%E4%B8%BA" class="uri">http://127.0.0.1:8000/admin/auth/group/add/,然后创建一个名为</a> <code>translations</code> 的新组。当编辑一个用户时，在<strong>Permissions</strong>模块下，把 <code>translations</code> 组添加进<strong>ChosenGroups</strong>中。Rosetta 只对超级用户和 <code>translations</code> 中的用户是可用的。</p>
<p>你可以在这个网站阅读 Rosetta 的文档：<a href="http://django-rosetta.readthedocs.org/en/latest/" class="uri">http://django-rosetta.readthedocs.org/en/latest/</a> 。</p>
<blockquote>
<p>当你在生产环境中添加新的翻译时，如果你的 Django 运行在一个真实服务器上，你必须在运行 <code>compilemessages</code> 或保存翻译之后重启你的服务器来让 Rosetta 的更改生效。</p>
</blockquote>
<h2 id="惰性翻译"><strong>惰性翻译</strong></h2>
<p>你或许已经注意到了 Rosetta 有 <strong>Fuzzy</strong> 这一行。这不是 Rosetta 的特性，它是由 gettext 提供的。如果翻译的 flag 是激活的，那么它就不会被包含进编译后的信息文件中。flag 用于需要翻译器修订的翻译字符串。当 <code>.po</code> 文件在更新新的翻译字符串时，一些翻译字符串可能被自动标记为 fuzzy. 当 gettext 找到一些变动不大的 <code>msgid</code> 时就会发生这样的情况，gettext 就会把它认为的旧的翻译和匹配在一起然后会在回把它标记为 fuzzy 以用于回查。翻译器之后会回查模糊翻译，会移除 fuzzy 标签然后再次编译信息文件。</p>
<h2 id="国际化的-url-模式"><strong>国际化的 URL 模式</strong></h2>
<p>Django 提供了用于国际化的 URLs。它包含两种主要用于国际化的 URLs：</p>
<ul>
<li><strong>URL 模式中的语言前缀</strong>：把语言的前缀添加到 URLs 当中，以便在不同的基本 URL 下提供每一种语言的版本。</li>
<li><strong>翻译后的 URL 模式</strong>：标记要翻译的 URL 模式，这样同样的 URL 就可以服务于不同的语言。</li>
</ul>
<p>翻译 URLs 的原因是这样就可以优化你的站点，方便搜索引擎搜索。通过添加语言前缀，你就可以为每一种语言提供索引，而不是所有语言用一种索引。并且, 把 URLs 为不同语言，你就可以提供给搜索引擎更好的搜索序列。</p>
<h2 id="把语言前缀添加到-urls-模式中"><strong>把语言前缀添加到 URLs 模式中</strong></h2>
<p>Django 允许你可以把语言前缀添加到 URLs 模式中。举个例子，网站的英文版可以服务于 <code>/en/</code> 起始路径之下，西班牙语版服务于 <code>/es/</code> 起始路径之下。</p>
<p>为了在你的 URLs 模式中使用不同语言，你必须确保 <code>settings.py</code> 中的 <code>MIDDLEWARE_CLASSES</code> 设置中有 <code>django.middleware.localMiddlewar</code> 。Django 将会使用它来辨别当前请求中的语言。</p>
<p>让我们把语言前缀添加到 URLs 模式中。编辑 <code>myshop</code> 项目的 <code>urls.py</code> ，添加以下库：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls.i18n <span class="im"><span class="hljs-keyword">import</span></span> i18n_patterns</code></pre></div>
<p>然后添加 <code>i18n_patterns()</code> :</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> i18n_patterns(
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
    url(<span class="vs"><span class="hljs-string">r'^cart/'</span></span>, include(<span class="st"><span class="hljs-string">'cart.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'cart'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^orders/'</span></span>, include(<span class="st"><span class="hljs-string">'orders.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'orders'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^payment/'</span></span>, include(<span class="st"><span class="hljs-string">'payment.urls'</span></span>,
                        namespace<span class="op">=</span><span class="st"><span class="hljs-string">'payment'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^paypal/'</span></span>, include(<span class="st"><span class="hljs-string">'paypal.standard.ipn.urls'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^coupons/'</span></span>, include(<span class="st"><span class="hljs-string">'coupons.urls'</span></span>,
                    namespace<span class="op">=</span><span class="st"><span class="hljs-string">'coupons'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^rosetta/'</span></span>, include(<span class="st"><span class="hljs-string">'rosetta.urls'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^'</span></span>, include(<span class="st"><span class="hljs-string">'shop.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'shop'</span></span>)),
    )</code></pre></div>
<p>你可以把 <code>i18n_patterns()</code> 和 <code>patterns()</code> URLs 模式结合起来，这样一些模式就会包含语言前缀另一些就不会包含。尽管，最好还是使用翻译后的 URLs 来避免 URL 匹配一个未翻译的 URL 模式的可能。</p>
<p>打开开发服务器，访问 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a> ，因为你在 Django 中使用 <code>LocaleMiddleware</code> 来执行 <code>How Django determines the current language</code> 中描述的步骤来决定当前语言，然后它就会把你重定向到包含相同语言前缀的 URL。看看浏览器中 URL ，它看起来像这样：<a href="http://127.0.0.1:8000/en/.%E5%BD%93%E5%89%8D%E8%AF%AD%E8%A8%80%E5%B0%86%E4%BC%9A%E8%A2%AB%E8%AE%BE%E7%BD%AE%E5%9C%A8%E6%B5%8F%E8%A7%88%E5%99%A8%E7%9A%84" class="uri">http://127.0.0.1:8000/en/.当前语言将会被设置在浏览器的</a> <code>Accept-Language</code> 头中，设为英语或者西班牙语或者是 <code>LANGUAGE_OCDE</code>（English） 中的默认设置。</p>
<h2 id="翻译-url-模式"><strong>翻译 URL 模式</strong></h2>
<p>Django 支持 URL 模式中有翻译了的字符串。你可以为每一种语言使用不同的 URL 模式。你可以使用 <code>ugettet_lazy()</code> 函数标记 URL 模式中需要翻译的字符串。</p>
<p>编辑 <code>myshop</code> 项目中的 <code>urls.py</code> 文件，把翻译字符串添加进 <code>cart</code>,<code>orders</code>,<code>payment</code>,<code>coupons</code> 的 URLs 模式的正则表达式中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

urlpatterns <span class="op">=</span> i18n_patterns(
    url(<span class="vs"><span class="hljs-string">r'^admin/'</span></span>, include(admin.site.urls)),
    url(_(<span class="vs"><span class="hljs-string">r'^cart/'</span></span>), include(<span class="st"><span class="hljs-string">'cart.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'cart'</span></span>)),
    url(_(<span class="vs"><span class="hljs-string">r'^orders/'</span></span>), include(<span class="st"><span class="hljs-string">'orders.urls'</span></span>,
                    namespace<span class="op">=</span><span class="st"><span class="hljs-string">'orders'</span></span>)),
    url(_(<span class="vs"><span class="hljs-string">r'^payment/'</span></span>), include(<span class="st"><span class="hljs-string">'payment.urls'</span></span>,
                    namespace<span class="op">=</span><span class="st"><span class="hljs-string">'payment'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^paypal/'</span></span>, include(<span class="st"><span class="hljs-string">'paypal.standard.ipn.urls'</span></span>)),
    url(_(<span class="vs"><span class="hljs-string">r'^coupons/'</span></span>), include(<span class="st"><span class="hljs-string">'coupons.urls'</span></span>,
                    namespace<span class="op">=</span><span class="st"><span class="hljs-string">'coupons'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^rosetta/'</span></span>, include(<span class="st"><span class="hljs-string">'rosetta.urls'</span></span>)),
    url(<span class="vs"><span class="hljs-string">r'^'</span></span>, include(<span class="st"><span class="hljs-string">'shop.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'shop'</span></span>)),
)</code></pre></div>
<p>编辑 <code>orders</code> 应用的 <code>urls.py</code> 文件，标记需要翻译的 URLs 模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> .<span class="im"><span class="hljs-keyword">import</span></span> views
<span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

urlpatterns <span class="op">=</span> [
    url(_(<span class="vs"><span class="hljs-string">r'^create/$'</span></span>), views.order_create, name<span class="op">=</span><span class="st"><span class="hljs-string">'order_create'</span></span>),
    <span class="co"><span class="hljs-comment"># ...</span></span>
]</code></pre></div>
<p>编辑 <code>payment</code> 应用的 <code>urls.py</code> 文件，把代码改成这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views
<span class="im"><span class="hljs-keyword">from</span></span> django.utils.translation <span class="im"><span class="hljs-keyword">import</span></span> gettext_lazy <span class="im"><span class="hljs-keyword">as</span></span> _

urlpatterns <span class="op">=</span> [
    url(_(<span class="vs"><span class="hljs-string">r'^process/$'</span></span>), views.payment_process, name<span class="op">=</span><span class="st"><span class="hljs-string">'process'</span></span>),
    url(_(<span class="vs"><span class="hljs-string">r'^done/$'</span></span>), views.payment_done, name<span class="op">=</span><span class="st"><span class="hljs-string">'done'</span></span>),
    url(_(<span class="vs"><span class="hljs-string">r'^canceled/$'</span></span>),
        views.payment_canceled,
        name<span class="op">=</span><span class="st"><span class="hljs-string">'canceled'</span></span>),
]</code></pre></div>
<p>我们不需要翻译 <code>shop</code> 应用的 URL 模式，因为它们是用变量创建的，而且也没有包含其他的任何字符。</p>
<p>打开 shell ，运行下面的命令来把新的翻译更新到信息文件：</p>
<pre><code class="hljs fortran">django-admin makemessages --<span class="hljs-built_in">all</span></code></pre>
<p>确保开发服务器正在运行中，访问：<a href="http://127.0.0.1:8000/en/rosetta/" class="uri">http://127.0.0.1:8000/en/rosetta/</a> ,点击<strong>Spanish</strong>下的<strong>Myshop</strong> 链接。你可以使用显示过滤器（display filter）来查看没有被翻译的字符串。确保你的 URL 翻译有正则表达式中的特殊字符。翻译 URLs 是一个精细活儿；如果你替换了正则表达式，你可能会破坏 URL。</p>
<h2 id="允许用户切换语言"><strong>允许用户切换语言</strong></h2>
<p>因为我们正在提供多语种服务，我们应当让用户可以选择站点的语言。我们将会为我们的站点添加语言选择器。语言选择器由可用的语言列表组成，我们使用链接来展示这些语言选项：</p>
<p>编辑 <code>shop/base.html</code> 模板（template），找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"header"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"logo"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>%</span> trans "My shop" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre></div>
<p>把他们换成以下几行：：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"header"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"logo"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>%</span> trans "My shop" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span>{</span><span>%</span> get_current_language as LANGUAGE_CODE <span>%</span><span>}</span>
<span>{</span><span>%</span> get_available_languages as LANGUAGES <span>%</span><span>}</span>
<span>{</span><span>%</span> get_language_info_list for LANGUAGES as languages <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"languages"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span><span>{</span><span>%</span> trans "Language" <span>%</span><span>}</span>:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"languages"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>%</span> for language in languages <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"/<span>{</span><span>{</span> language.code <span>}</span><span>}</span>/"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">if</span> <span class="hljs-attr">language.code</span></span></span><span class="hljs-tag"> </span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">=</span></span></span><span class="hljs-tag">
</span><span class="ot"><span class="hljs-tag"><span class="hljs-attr">LANGUAGE_CODE</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"selected"</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string"><span>{</span><span>%</span></span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span>{</span><span>{</span> language.name_local <span>}</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span></code></pre></div>
<p>我们是这样创建我们的语言选择器的：</p>
<pre><code class="hljs django"><span class="xml">1. 首先使用 `</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">load</span></span> i18n <span>%</span><span>}</span></span><span class="xml">` 加载国际化
2. 使用 `</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">get_current_language</span></span> <span>%</span><span>}</span></span><span class="xml">` 标签来检索当前语言
3. 使用 `</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">get_available_languages</span></span> <span>%</span><span>}</span></span><span class="xml">` 模板（template）标签来过去 `LANGUAGES` 中定义语言
4. 使用 `</span><span class="hljs-template-tag"><span>{</span><span>%</span> <span class="hljs-name"><span class="hljs-name">get_language_info_list</span></span> <span>%</span><span>}</span></span><span class="xml">` 来提供简易的语言属性连接入口
5. 我们创建了一个 HTML 列表来展示所有的可用语言列表然后我们为当前激活语言添加了 `selected` 属性</span></code></pre>
<p>我们使用基于项目设置中语言变量的 i18n<code>提供的模板（template）标签。现在访问：</code><a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000/</a>` ，你应该能在站点顶部的右边看到语言选择器：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-8.png" alt="django-9-8"></p>
<p>用户现在可以轻易的切换他们的语言了。</p>
<h2 id="使用-django-parler-翻译模型models"><strong>使用 django-parler 翻译模型（models）</strong></h2>
<p>Django 没有提供开箱即用的模型（models）翻译的解决办法。你必须要自己实现你自己的解决办法来管理不同语言中的内容或者使用第三方模块来管理模型（model）翻译。有几种第三方应用允许你翻译模型（model）字段。每一种手采用了不同的方法保存和连接翻译。其中一种应用是 django-parler 。这个模块提供了非常有效的办法来翻译模型（models）并且它和 Django 的管理站点整合的非常好。</p>
<p>django-parler 为每一个模型（model）生成包含翻译的分离数据库表。这个表包含了所有的翻译的字段和源对象的外键翻译。它也包含了一个语言字段，一位内每一行都会保存一个语言的内容。</p>
<h2 id="安装-django-parler"><strong>安装 django-parler </strong></h2>
<p>使用 pip 安装 django-parler :</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-parler==<span class="hljs-number">1.5</span>.<span class="hljs-number">1</span></code></pre>
<p>编辑项目中 <code>settings.py</code> ，把 <code>parler</code> 添加到 <code>INSTALLED_APPS</code> 中，同样也把下面的配置添加到配置文件中：</p>
<pre><code class="hljs python">PARLER_LANGUAGES = {
    <span class="hljs-keyword">None</span>: (
        {<span class="hljs-string">'code'</span>: <span class="hljs-string">'en'</span>,},
        {<span class="hljs-string">'code'</span>: <span class="hljs-string">'es'</span>,},
    ),
    <span class="hljs-string">'default'</span>: {
        <span class="hljs-string">'fallback'</span>: <span class="hljs-string">'en'</span>,
        <span class="hljs-string">'hide_untranslated'</span>: <span class="hljs-keyword">False</span>,
    }
}</code></pre>
<p>这个设置为 django-parler 定义了可用语言 <code>en</code> 和 <code>es</code> 。我们指定默认语言为 <code>en</code> ，然后我们指定 django-parler 应该隐藏未翻译的内容。</p>
<h2 id="翻译模型model字段"><strong>翻译模型（model）字段</strong></h2>
<p>让我们为我们的产品目录添加翻译。 django-parler 提供了 <code>TranslatedModel</code> 模型（model）类和 <code>TranslatedFields</code> 闭包（wrapper）来翻译模型（model）字段。编辑 <code>shop</code> 应用路径下的 <code>models.py</code> 文件，添加以下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> parler.models <span class="im"><span class="hljs-keyword">import</span></span> TranslatableModel, TranslatedFields</code></pre></div>
<p>然后更改 <code>Category</code> 模型（model）来使 <code>name</code> 和 <code>slug</code> 字段可以被翻译。<br>
我们依然保留了每行未翻译的字段：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Category</span><span class="hljs-params">(TranslatableModel)</span>:</span>
    name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                            db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>,
                            unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    translations <span class="op">=</span> TranslatedFields(
        name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                                db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                                db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>,
                                unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
)</code></pre></div>
<p><code>Category</code> 模型（model）继承了 <code>TranslatableModel</code> 而不是 <code>models.Model</code>。并且 <code>name</code> 和 <code>slug</code> 字段都被引入到了 <code>TranslatedFields</code> 闭包（wrapper）中。</p>
<p>编辑 <code>Product</code> 模型（model），为 <code>name</code>,<code>slug</code> ,<code>description</code> 添加翻译，同样我们也保留了每行未翻译的字段：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Product</span><span class="hljs-params">(TranslatableModel)</span>:</span>
    name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    translations <span class="op">=</span> TranslatedFields(
        name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    )
    category <span class="op">=</span> models.ForeignKey(Category,
                        related_name<span class="op">=</span><span class="st"><span class="hljs-string">'products'</span></span>)
    image <span class="op">=</span> models.ImageField(upload_to<span class="op">=</span><span class="st"><span class="hljs-string">'products/%Y/%m/</span></span><span class="sc"><span class="hljs-string">%d</span></span><span class="st"><span class="hljs-string">'</span></span>,
                        blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    price <span class="op">=</span> models.DecimalField(max_digits<span class="op">=</span><span class="dv"><span class="hljs-number">10</span></span>, decimal_places<span class="op">=</span><span class="dv"><span class="hljs-number">2</span></span>)
    stock <span class="op">=</span> models.PositiveIntegerField()
    available <span class="op">=</span> models.BooleanField(default<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>django-parler 为每个可翻译模型（model）生成了一个模型（model）。在下面的图片中，你可以看到 <code>Product</code> 字段和生成的 <code>ProductTranslation</code> 模型（model）：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-9.png" alt="django-9-9"></p>
<p>django-parler 生成的 <code>ProductTranslation</code> 模型（model）有 <code>name</code>,<code>slug</code>,<code>description</code> 可翻译字段，一个 <code>language_code</code> 字段，和主要的 <code>Product</code> 对象的 <code>ForeignKey</code> 字段。<code>Product</code> 和 <code>ProductTranslation</code> 之间有一个一对多关系。一个 <code>ProductTranslation</code> 对象会为每个可用语言生成 <code>Product</code> 对象。</p>
<p>因为 Django 为翻译都使用了相互独立的表，所以有一些 Django 的特性我们是用不了。使用翻译后的字段来默认排序是不可能的。你可以在查询集（QuerySets）中使用翻译字段来过滤，但是你不可以在 <code>ordering Meta</code> 选项中引入可翻译字段。编辑 <code>shop</code> 应用的 <code>models.py</code> ，然后注释掉 <code>Category Meta</code> 类的 <code>ordering</code> 属性：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
    <span class="co"><span class="hljs-comment"># ordering = ('name',)</span></span>
    verbose_name <span class="op">=</span> <span class="st"><span class="hljs-string">'category'</span></span>
    verbose_name_plural <span class="op">=</span> <span class="st"><span class="hljs-string">'categories'</span></span></code></pre></div>
<p>我们也必须注释掉 <code>Product Meta</code> 类的 <code>index_together</code> 属性，因为当前的 django-parler 的版本不支持对它的验证。编辑 <code>Product Meta</code> ：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
    ordering <span class="op">=</span> (<span class="st"><span class="hljs-string">'-created'</span></span>,)
    <span class="co"><span class="hljs-comment"># index_together = (('id', 'slug'),)</span></span></code></pre></div>
<p>你可以在这个网站阅读更多 django-parler 兼容的有关内容：<br>
<a href="http://django-parler.readthedocs.org/en/latest/compatibility.html" class="uri">http://django-parler.readthedocs.org/en/latest/compatibility.html</a> 。</p>
<h2 id="创建一次定制的迁移"><strong>创建一次定制的迁移</strong></h2>
<p>当你创建新的翻译模型（models）时，你需要执行 <code>makemessages</code> 来生成模型（models）的迁移，然后把变更同步到数据库中。尽管当使已存在字段可翻译化时，你或许有想要在数据库中保留的数据。我们将迁移当前数据到新的翻译模型（models）中。因此，我们添加了翻译字段但是有意保存了原始字段。</p>
<p>翻译添加到当前字段的步骤如下：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>在保留原始字段的情况下，创建新翻译模型（model）的迁移
<span class="hljs-bullet">2. </span>创建定制化迁移，从当前已有的字段中拷贝一份数据到翻译模型（models）中
<span class="hljs-bullet">3. </span>从源模型（models）中删除字段</code></pre>
<p>运行下面的命令来为我们添加到 <code>Category</code> 和 <code>Product</code> 模型（model）中的翻译字段创建迁移：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">python</span> manage.py makemigrations shop --name <span class="hljs-string">"add_translation_model"</span></code></pre>
<p>你可以看到如下输出：</p>
<pre><code class="hljs sql">Migrations for 'shop':
    0002_add_translation_model.py:
        - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> CategoryTranslation
        - <span class="hljs-keyword">Create</span> <span class="hljs-keyword">model</span> ProductTranslation
        - <span class="hljs-keyword">Change</span> Meta options <span class="hljs-keyword">on</span> <span class="hljs-keyword">category</span>
        - <span class="hljs-keyword">Alter</span> index_together <span class="hljs-keyword">for</span> product (<span class="hljs-number">0</span> <span class="hljs-keyword">constraint</span>(s))
        - <span class="hljs-keyword">Add</span> <span class="hljs-keyword">field</span> <span class="hljs-keyword">master</span> <span class="hljs-keyword">to</span> producttranslation
        - <span class="hljs-keyword">Add</span> <span class="hljs-keyword">field</span> <span class="hljs-keyword">master</span> <span class="hljs-keyword">to</span> categorytranslation
        - <span class="hljs-keyword">Alter</span> unique_together <span class="hljs-keyword">for</span> producttranslation (<span class="hljs-number">1</span> <span class="hljs-keyword">constraint</span>(s))
        - <span class="hljs-keyword">Alter</span> unique_together <span class="hljs-keyword">for</span> categorytranslation (<span class="hljs-number">1</span> <span class="hljs-keyword">constraint</span>(s))</code></pre>
<h2 id="迁移已有数据"><strong>迁移已有数据</strong></h2>
<p>现在我们需要创建定制迁移来把已有数据拷贝到新的翻译模型（model）中。使用这个命令创建一个空的迁移：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">python</span> manage.py makemigrations --empty shop --name <span class="hljs-string">"migrate_translatable_fields"</span></code></pre>
<p>你将会看到如下输出：</p>
<pre><code class="hljs python">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'shop'</span>:
    <span class="hljs-number">0003</span>_migrate_translatable_fields.py</code></pre>
<p>编辑 <code>shop/migrations/0003_migrate_translatable_fields.py</code> ，然后添加下面的代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># -*- coding: utf-8 -*-</span></span>
<span class="im"><span class="hljs-keyword">from</span></span> __future__ <span class="im"><span class="hljs-keyword">import</span></span> unicode_literals
<span class="im"><span class="hljs-keyword">from</span></span> django.db <span class="im"><span class="hljs-keyword">import</span></span> models, migrations
<span class="im"><span class="hljs-keyword">from</span></span> django.apps <span class="im"><span class="hljs-keyword">import</span></span> apps
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> django.core.exceptions <span class="im"><span class="hljs-keyword">import</span></span> ObjectDoesNotExist

translatable_models <span class="op">=</span> {
    <span class="st"><span class="hljs-string">'Category'</span></span>: [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>],
    <span class="co"><span class="hljs-string">'Product'</span></span>: [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'description'</span></span>],
}

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">forwards_func</span><span class="hljs-params">(apps, schema_editor)</span>:</span>
    <span class="cf"><span class="hljs-keyword">for</span></span> model, fields <span class="op"><span class="hljs-keyword">in</span></span> translatable_models.items():
        Model <span class="op">=</span> apps.get_model(<span class="st"><span class="hljs-string">'shop'</span></span>, model)
        ModelTranslation <span class="op">=</span> apps.get_model(<span class="st"><span class="hljs-string">'shop'</span></span>,
                                <span class="co"><span class="hljs-string">'{}Translation'</span></span>.<span class="bu">format</span>(model))

    <span class="cf"><span class="hljs-keyword">for</span></span> obj <span class="op"><span class="hljs-keyword">in</span></span> Model.objects.<span class="bu">all</span>():
        translation_fields <span class="op">=</span> {field: <span class="bu">getattr</span>(obj, field) <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> fields}
        translation <span class="op">=</span> ModelTranslation.objects.create(
                        master_id<span class="op">=</span>obj.pk,
                        language_code<span class="op">=</span>settings.LANGUAGE_CODE,
                        <span class="op">**</span>translation_fields)

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">backwards_func</span><span class="hljs-params">(apps, schema_editor)</span>:</span>
    <span class="cf"><span class="hljs-keyword">for</span></span> model, fields <span class="op"><span class="hljs-keyword">in</span></span> translatable_models.items():
        Model <span class="op">=</span> apps.get_model(<span class="st"><span class="hljs-string">'shop'</span></span>, model)
        ModelTranslation <span class="op">=</span> apps.get_model(<span class="st"><span class="hljs-string">'shop'</span></span>,
                                <span class="co"><span class="hljs-string">'{}Translation'</span></span>.<span class="bu">format</span>(model))
    <span class="cf"><span class="hljs-keyword">for</span></span> obj <span class="op"><span class="hljs-keyword">in</span></span> Model.objects.<span class="bu">all</span>():
        translation <span class="op">=</span> _get_translation(obj, ModelTranslation)
        <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> fields:
            <span class="bu">setattr</span>(obj, field, <span class="bu">getattr</span>(translation, field))
        obj.save()

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">_get_translation</span><span class="hljs-params">(obj, MyModelTranslation)</span>:</span>
    translations <span class="op">=</span> MyModelTranslation.objects.<span class="bu">filter</span>(master_id<span class="op">=</span>obj.pk)
    <span class="cf"><span class="hljs-keyword">try</span></span>:
        <span class="co"><span class="hljs-comment"># Try default translation</span></span>
        <span class="cf"><span class="hljs-keyword">return</span></span> translations.get(language_code<span class="op">=</span>settings.LANGUAGE_CODE)
    <span class="cf"><span class="hljs-keyword">except</span></span> ObjectDoesNotExist:
        <span class="co"><span class="hljs-comment"># Hope there is a single translation</span></span>
        <span class="cf"><span class="hljs-keyword">return</span></span> translations.get()

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Migration</span><span class="hljs-params">(migrations.Migration)</span>:</span>
    dependencies <span class="op">=</span> [
    (<span class="st"><span class="hljs-string">'shop'</span></span>, <span class="st"><span class="hljs-string">'0002_add_translation_model'</span></span>),
    ]
    operations <span class="op">=</span> [
    migrations.RunPython(forwards_func, backwards_func),
    ]</code></pre></div>
<p>这段迁移包括了用于执行应用和反转迁移的 <code>forwards_func()</code> 和 <code>backwards_func()</code> 。</p>
<p>迁移工作流程如下：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>我们在 <span class="hljs-code">`translatable_models`</span> 字典中定义了模型（model）和它们的可翻译字段
<span class="hljs-bullet">2. </span>为了应用迁移，我们使用 <span class="hljs-code">`app.get_model()`</span> 来迭代包含翻译的模型（model）来得到这个模型（model）和其可翻译的模型（model）
<span class="hljs-bullet">3. </span>我们在数据库中迭代所有的当前对象，然后为定义在项目设置中的 <span class="hljs-code">`LANGUAGE_CODE`</span> 创建一个翻译对象。我们引入了 <span class="hljs-code">`ForeignKey`</span> 到源对像和一份从源字段中可翻译字段的拷贝。</code></pre>
<p>backwards 函数执行的是反转操作，它检索默认的翻译对象，然后把可翻译字段的值拷贝到新的模型（model）中。</p>
<p>最后，我们需要删除我们不再需要的源字段。编辑 <code>shop</code> 应用的 <code>models.py</code> ，然后删除 <code>Category</code> 模型（model）的 <code>name</code> 和 <code>slug</code> 字段：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Category</span><span class="hljs-params">(TranslatableModel)</span>:</span>
    translations <span class="op">=</span> TranslatedFields(
        name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>,
                        db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>,
                        unique<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    )</code></pre></div>
<p>删除 <code>Product</code> 模型（model）的 <code>name</code> 和 <code>slug</code> 字段：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Product</span><span class="hljs-params">(TranslatableModel)</span>:</span>
    translations <span class="op">=</span> TranslatedFields(
        name <span class="op">=</span> models.CharField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        slug <span class="op">=</span> models.SlugField(max_length<span class="op">=</span><span class="dv"><span class="hljs-number">200</span></span>, db_index<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>),
        description <span class="op">=</span> models.TextField(blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    )
    category <span class="op">=</span> models.ForeignKey(Category,
                        related_name<span class="op">=</span><span class="st"><span class="hljs-string">'products'</span></span>)
    image <span class="op">=</span> models.ImageField(upload_to<span class="op">=</span><span class="st"><span class="hljs-string">'products/%Y/%m/</span></span><span class="sc"><span class="hljs-string">%d</span></span><span class="st"><span class="hljs-string">'</span></span>,
                        blank<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    price <span class="op">=</span> models.DecimalField(max_digits<span class="op">=</span><span class="dv"><span class="hljs-number">10</span></span>, decimal_places<span class="op">=</span><span class="dv"><span class="hljs-number">2</span></span>)
    stock <span class="op">=</span> models.PositiveIntegerField()
    available <span class="op">=</span> models.BooleanField(default<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    created <span class="op">=</span> models.DateTimeField(auto_now_add<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    updated <span class="op">=</span> models.DateTimeField(auto_now<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>现在我们必须要创建最后一次迁移来应用其中的变更。如果我们尝试 <code>manage.py</code> 工具，我们将会得到一个错误，因为我们还没有让管理站点适应翻译模型（models）。让我们先来修整一下管理站点。</p>
<h2 id="在管理站点中整合翻译"><strong>在管理站点中整合翻译</strong></h2>
<p>Django-parler 很好和 Django 的管理站点相融合。它用 <code>TranslatableAdmin</code> 重写了 Django 的 <code>ModelAdmin</code> 类 来管理翻译。</p>
<p>编辑 shop 应用的 <code>admin.py</code> 文件，添加下面的库：</p>
<pre><code class="hljs swift">from parler.admin <span class="hljs-keyword">import</span> TranslatableAdmin</code></pre>
<p>把 <code>CategoryAdmin</code> 和 <code>ProductAdmin</code> 的继承类改为 <code>TranslatableAdmin</code> 取代原来的 <code>ModelAdmin</code>。 Django-parler 现在还不支持 <code>prepopulated_fields</code> 属性，但是它支持 <code>get_prepopulated_fields()</code> 方法来提供相同的功能。让我们据此做一些改变。 <code>admin.py</code> 文件看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib <span class="im"><span class="hljs-keyword">import</span></span> admin
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Category, Product
<span class="im"><span class="hljs-keyword">from</span></span> parler.admin <span class="im"><span class="hljs-keyword">import</span></span> TranslatableAdmin

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CategoryAdmin</span><span class="hljs-params">(TranslatableAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>]
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_prepopulated_fields</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, obj</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'name'</span></span>,)}
admin.site.register(Category, CategoryAdmin)

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ProductAdmin</span><span class="hljs-params">(TranslatableAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'name'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'category'</span></span>, <span class="st"><span class="hljs-string">'price'</span></span>, <span class="st"><span class="hljs-string">'stock'</span></span>,
                    <span class="co"><span class="hljs-string">'available'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>]
    list_filter <span class="op">=</span> [<span class="st"><span class="hljs-string">'available'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'updated'</span></span>, <span class="st"><span class="hljs-string">'category'</span></span>]
    list_editable <span class="op">=</span> [<span class="st"><span class="hljs-string">'price'</span></span>, <span class="st"><span class="hljs-string">'stock'</span></span>, <span class="st"><span class="hljs-string">'available'</span></span>]

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_prepopulated_fields</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, obj</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> {<span class="st"><span class="hljs-string">'slug'</span></span>: (<span class="st"><span class="hljs-string">'name'</span></span>,)}

admin.site.register(Product, ProductAdmin)</code></pre></div>
<p>我们已经使管理站点能够和新的翻译模型（model）工作了。现在我们就能把变更同步到数据库中了。</p>
<h2 id="应用翻译模型model迁移"><strong>应用翻译模型（model）迁移</strong></h2>
<p>我们在变更管理站点之前已经从模型（model）中删除了旧的字段。现在我们需要为这次变更创建迁移。打开 shell ，运行下面的命令：</p>
<pre><code class="hljs nginx"><span class="hljs-attribute">python</span> manage.py makemigrations shop --name <span class="hljs-string">"remove_untranslated_fields"</span></code></pre>
<p>你将会看到如下输出：</p>
<pre><code class="hljs python">Migrations <span class="hljs-keyword">for</span> <span class="hljs-string">'shop'</span>:
    <span class="hljs-number">0004</span>_remove_untranslated_fields.py:
        - Remove field name <span class="hljs-keyword">from</span> category
        - Remove field slug <span class="hljs-keyword">from</span> category
        - Remove field description <span class="hljs-keyword">from</span> product
        - Remove field name <span class="hljs-keyword">from</span> product
        - Remove field slug <span class="hljs-keyword">from</span> product</code></pre>
<p>迁移之后，我们就已经删除了源字段保留了可翻译字段。</p>
<p>总结一下，我们创建了以下迁移：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>将可翻译字段添加到模型（models）中
<span class="hljs-bullet">2. </span>将源字段中的数据迁移到可翻译字段中
<span class="hljs-bullet">3. </span>从源模型（models）中删除源字段</code></pre>
<p>运行下面的命令来应用我们创建的迁移：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span> <span class="hljs-selector-tag">shop</span></code></pre>
<p>你将会看到如下输出：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">shop</span><span class="hljs-selector-class">.0002_add_translation_model</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">shop</span><span class="hljs-selector-class">.0003_migrate_translatable_fields</span>... <span class="hljs-selector-tag">OK</span>
<span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">shop</span><span class="hljs-selector-class">.0004_remove_untranslated_fields</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>我们的模型（models）现在已经同步到了数据库中。让我们来翻译一个对象。</p>
<p>用命令 <code>pythong manage.py runserver</code> 运行开发服务器，在浏览器中访问：<a href="http://127.0.0.1:8000/en/admin/shop/category/add/" class="uri">http://127.0.0.1:8000/en/admin/shop/category/add/</a> 。你就会看到包含两个标签的 <strong>Add category</strong> 页，一个标签是英文的，一个标签是西班牙语的。</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-10.png" alt="django-9-10"></p>
<p>你现在可以添加一个翻译然后点击<strong>Save</strong>按钮。确保你在切换标签之前保存了他们，不然让门就会丢失。</p>
<h2 id="使视图views适应翻译"><strong>使视图（views）适应翻译</strong></h2>
<p>我们要使我们的 <code>shop</code> 视图（views）适应我们的翻译查询集（QuerySets）。在命令行运行 <code>python manage.py shell</code> ，看看你是如何检索和查询翻译字段的。为了从当前激活语言中得到一个字段的内容，你只需要用和连接一般字段相同的办法连接这个字段：</p>
<pre><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; from shop.models import Product
<span class="hljs-meta">&gt;&gt;</span>&gt; product=Product.objects.first()
<span class="hljs-meta">&gt;&gt;</span>&gt; product.name
<span class="hljs-string">'Black tea'</span></code></pre>
<p>当你连接到被翻译了字段时，他们已经被用当前语言处理过了。你可以为一个对象设置不同的当前语言，这样你就可以获得指定的翻译了：</p>
<pre><code class="hljs ruby"><span class="hljs-meta">&gt;&gt;</span>&gt; product.set_current_language(<span class="hljs-string">'es'</span>)
<span class="hljs-meta">&gt;&gt;</span>&gt; product.name
<span class="hljs-string">'Té negro'</span>
<span class="hljs-meta">&gt;&gt;</span>&gt; product.get_current_language()
<span class="hljs-string">'es</span></code></pre>
<p>当使用 <code>filter()</code> 执行一次查询集（QuerySets）时，你可以使用 <code>translations__</code> 语法筛选相关联对象:</p>
<pre><code class="hljs fsharp">&gt;&gt;&gt; Product.objects.filter(translations__name='Black tea')
<span class="hljs-meta">[&lt;Product: Black tea&gt;]</span></code></pre>
<p>你也可以使用 <code>language()</code> 管理器来为每个检索的对象指定特定的语言：</p>
<pre><code class="hljs fsharp">&gt;&gt;&gt; Product.objects.language('es').all()
<span class="hljs-meta">[&lt;Product: Té negro&gt;, &lt;Product: Té en polvo&gt;, &lt;Product: Té rojo&gt;,
&lt;Product: Té verde&gt;]</span></code></pre>
<p>如你所见，连接到翻译字段的方法还是很直接的。</p>
<p>让我们修改一下产品目录的视图（views）吧。编辑 <code>shop</code> 应用的 <code>views.py</code> ，在 <code>product_list</code> 视图（view）中找到下面这一行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">category <span class="op">=</span> get_object_or_404(Category, slug<span class="op">=</span>category_slug)</code></pre></div>
<p>把它替换为下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">language <span class="op">=</span> request.LANGUAGE_CODE
category <span class="op">=</span> get_object_or_404(Category,
                    translations__language_code<span class="op">=</span>language,
                    translations__slug<span class="op">=</span>category_slug)</code></pre></div>
<p>然后，编辑 <code>product_detail</code> 视图（view），找到下面这几行：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">product <span class="op">=</span> get_object_or_404(Product,
                            <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                            slug<span class="op">=</span>slug,
                            available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>把他们换成以下几行：：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">language <span class="op">=</span> request.LANGUAGE_CODE
get_object_or_404(Product,
                    <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                    translations__language_code<span class="op">=</span>language,
                    translations__slug<span class="op">=</span>slug,
                    available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)</code></pre></div>
<p>现在 <code>product_list</code> 和 <code>product_detail</code> 已经可以使用翻译字段来检索了。运行开发服务器，访问：<a href="http://127.0.0.1:8000/es/" class="uri">http://127.0.0.1:8000/es/</a> ，你可以看到产品列表页，包含了所有被翻译为西班牙语的产品：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-11.png" alt="django-9-11"></p>
<p>现在每个产品的 URLs 已经使用 <code>slug</code> 字段被翻译为了的当前语言。比如，一个西班牙语产品的 URL 为：<a href="http://127.0.0.1:8000/es/2/te-rojo/" class="uri">http://127.0.0.1:8000/es/2/te-rojo/</a> ,但是它的英文 URL 为：<a href="http://127.0.0.1:8000/en/2/red-tea/" class="uri">http://127.0.0.1:8000/en/2/red-tea/</a> 。如果你导航到产品详情页，你可以看到用被选语言翻译后的 URL 和内容，就像下面的例子：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-12.png" alt="django-9-12"></p>
<p>如果你想要更多了解 django-parler ，你可以在这个网站找到所有的文档：http//django-parler.readthedocs.org/en/latest/ 。</p>
<p>你已经学习了如何翻译 Python 代码，模板（template），URL 模式，模型（model）字段。为了完成国际化和本地化的工作，我们需要展示本地化格式的时间和日期、以及数字。</p>
<h2 id="本地格式化"><strong>本地格式化</strong></h2>
<p>基于用户的语言，你可能想要用不同的格式展示日期、时间和数字。本第格式化可通过修改 <code>settings.py</code> 中的 <code>USE_L1ON</code> 设置为 <code>True</code> 来使本地格式化生效。</p>
<p>当 <code>USE_L1ON</code> 可用时，Django 将会在模板（template）任何输出一个值时尝试使用某一语言的特定格式。你可以看到你的网站中用一个点来做分隔符的十进制英文数字，尽管在西班牙语版中他们使用逗号来做分隔符。这和 Django 里为 <code>es</code> 指定的语言格式。你可以在这里查看西班牙语的格式配置：<a href="https://github.com/django/django/blob/stable/1.8.x/django/conf/locale/es/formats.py" class="uri">https://github.com/django/django/blob/stable/1.8.x/django/conf/locale/es/formats.py</a> 。</p>
<p>通常，你把 <code>USE_L10N</code> 的值设为 <code>True</code> 然后 Django 就会为每一种语言应用本地化格式。虽然在某些情况下你有不想要被格式化的值。这和输出特别相关， JavaScript 或者 JSON 必须要提供机器可读的格式。</p>
<p>Django 提供了 <code><span>{</span><span>%</span> localize <span>%</span><span>}</span></code> 模板（template）表标签来让你可以开启或者关闭模板（template）的本地化。这使得你可以控制本地格式化。你需要载入 <code>l10n</code> 标签来使用这个模板（template）标签。下面的例子是如何在模板（template）中开启和关闭它：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> load l10n <span>%</span><span>}</span>

<span>{</span><span>%</span> localize on <span>%</span><span>}</span>
<span>{</span><span>{</span> value <span>}</span><span>}</span>
<span>{</span><span>%</span> endlocalize <span>%</span><span>}</span>

<span>{</span><span>%</span> localize off <span>%</span><span>}</span>
<span>{</span><span>{</span> value <span>}</span><span>}</span>
<span>{</span><span>%</span> endlocalize <span>%</span><span>}</span></code></pre></div>
<p>Django 同样也提供了 <code>localize</code> 和 <code>unlocalize</code> 模板（template）过滤器来强制或取消一个值的本地化。过滤器可以像下面这样使用：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>{</span> value|localize <span>}</span><span>}</span>
<span>{</span><span>{</span> value|unlocalize <span>}</span><span>}</span></code></pre></div>
<p>你也可以创建定制化的格式文件来指定特定语言的格式。你可以在这个网站找到所有关于本第格式化的信息：<a href="https://docs.djangoproject.com/en/1.8/topics/i18n/formatting/" class="uri">https://docs.djangoproject.com/en/1.8/topics/i18n/formatting/</a> 。</p>
<h2 id="使用-django-localflavor-来验证表单字段"><strong>使用 django-localflavor 来验证表单字段</strong></h2>
<p>django-localflavor 是一个包含特殊工具的第三方模块，比如它的有些表单字段或者模型（model）字段对于每个国家是不同的。它对于验证本地地区，本地电话号码，身份证号，社保号等等都是非常有用的。这个包被集成进了一系列以 ISO 3166 国家代码命名的模块里。</p>
<p>使用下面的命令安装 django-localflavor :</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-localflavor==<span class="hljs-number">1.1</span></code></pre>
<p>编辑项目中的 <code>settings.py</code> ，把 <code>localflavor</code> 添加进 <code>INSTALLED_APPS</code> 设置中。</p>
<p>我们将会添加美国（U.S.）邮政编码字段，这样之后创建一个新的订单就需要一个美国邮政编码了。</p>
<p>编辑 <code>orders</code> 应用的 <code>forms.py</code> ，让它看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django <span class="im"><span class="hljs-keyword">import</span></span> forms
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Order
<span class="im"><span class="hljs-keyword">from</span></span> localflavor.us.forms <span class="im"><span class="hljs-keyword">import</span></span> USZipCodeField

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderCreateForm</span><span class="hljs-params">(forms.ModelForm)</span>:</span>
    postal_code <span class="op">=</span> USZipCodeField()
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
    model <span class="op">=</span> Order
    fields <span class="op">=</span> [<span class="st"><span class="hljs-string">'first_name'</span></span>, <span class="st"><span class="hljs-string">'last_name'</span></span>, <span class="st"><span class="hljs-string">'email'</span></span>, <span class="st"><span class="hljs-string">'address'</span></span>,
                        <span class="co"><span class="hljs-string">'postal_code'</span></span>, <span class="st"><span class="hljs-string">'city'</span></span>,]</code></pre></div>
<p>我们从 <code>localflavor</code> 的 <code>us</code> 包中引入了 <code>USZipCodeField</code> 然后将它应用在 <code>OrderCreateForm</code> 的 <code>postal_code</code> 字段中。访问：<a href="http://127.0.0.1:8000/en/orders/create/" class="uri">http://127.0.0.1:8000/en/orders/create/</a> 然后尝试输入一个 3 位邮政编码。你将会得到 <code>USZipCodeField</code> 引发的验证错误：</p>
<pre><code class="hljs swift"><span class="hljs-type">Enter</span> a <span class="hljs-built_in">zip</span> code <span class="hljs-keyword">in</span> the format <span class="hljs-type">XXXXX</span> or <span class="hljs-type">XXXXX</span>-<span class="hljs-type">XXXX</span>.</code></pre>
<p>这只是一个在你的项目中使用定制字段来达到验证目的的简单例子，localflavor 提供的本地组件对于让你的项目适应不同的国家是非常有用的。你可以阅读 django-localflavor 的文档，看看所有的可用地区组件：<a href="https://django-localflavor.readthedocs.org/en/latest/" class="uri">https://django-localflavor.readthedocs.org/en/latest/</a> 。</p>
<p>下面，我们将会为我们的店铺创建一个推荐引擎。</p>
<h2 id="创建一个推荐引擎"><strong>创建一个推荐引擎</strong></h2>
<p>推荐引擎是一个可以预测用户对于某一物品偏好和比率的系统。这个系统会基于用户的行为和它对于用户的了解选出相关的商品。如今，推荐系统用于许多的在线服务。它们帮助用户从大量的相关数据中挑选他们可能感兴趣的产品。提供良好的推荐可以鼓励用户更多的消费。电商网站受益于日益增长的相关产品推荐销售中。</p>
<p>我们将会创建一个简单但强大的推荐引擎来推荐经常一起购买的商品。我们将基于历史销售来推荐商品，这样就可以辨别出哪些商品通常是一起购买的了。我们将会在两种不同的场景下推荐互补的产品:</p>
<ul>
<li><strong>产品详情页</strong>：我们将会展示和所给商品经常一起购买的产品列表。比如像这样展示：<strong>Users who bought this also bought X, Y, Z</strong>（购买了这个产品的用户也购买了X, Y,Z）。我们需要一个数据结构来让我们能储被展示产品中和每个产品一起购买的次数。</li>
<li><strong>购物车详情页</strong>：基于用户添加到购物车当中的产品，我们将会推荐经常和他们一起购买的产品。在这样的情况下，我们计算的包含相关产品的评分一定要相加。</li>
</ul>
<p>我们将会使用 Redis 来储存被一起购买的产品。记得你已经在<strong>第六章 跟踪用户操作 </strong>使用了 Redis 。如果你还没有安装 Redis ，你可以在那一章找到安装指导。</p>
<h2 id="推荐基于历时购物的产品"><strong>推荐基于历时购物的产品</strong></h2>
<p>现在，让我们推荐给用户一些基于他们添加到购物车的产品。我们将会在 Redis 中为每个产品储存一个键（key）。产品的键将会和它的评分一同储存在 Redis 的有序集中。在一次新的订单被完成时，我们每次都会为一同购买的产品的评分加一。</p>
<p>当一份订单付款成功后，我们保存每个购买产品的键，包括同意订单中的有序产品集。这个有序集让我们可以为一起购买的产品打分。</p>
<p>编辑项目中的额 <code>settings.py</code> ,添加以下设置：</p>
<pre><code class="hljs ini"><span class="hljs-attr">REDIS_HOST</span> = <span class="hljs-string">'localhost'</span>
<span class="hljs-attr">REDIS_PORT</span> = <span class="hljs-number">6379</span>
<span class="hljs-attr">REDIS_DB</span> = <span class="hljs-number">1</span></code></pre>
<p>这里的设置是为了和 Redis 服务器建立连接。在 <code>shop</code> 应用内创建一个新的文件，命名为 <code>recommender.py</code> 。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">import</span></span> redis
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Product

<span class="co"><span class="hljs-comment"># connect to redis</span></span>
r <span class="op">=</span> redis.StrictRedis(host<span class="op">=</span>settings.REDIS_HOST,
                    port<span class="op">=</span>settings.REDIS_PORT,
                    db<span class="op">=</span>settings.REDIS_DB)

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Recommender</span><span class="hljs-params">(</span></span><span class="bu"><span class="hljs-class"><span class="hljs-params">object</span></span></span><span class="hljs-class"><span class="hljs-params">)</span>:</span>

    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">get_product_key</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'product:{}:purchased_with'</span></span>.<span class="bu">format</span>(<span class="bu">id</span>)
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">products_bought</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, products)</span>:</span>
        product_ids <span class="op">=</span> [p.<span class="bu">id</span> <span class="cf"><span class="hljs-keyword">for</span></span> p <span class="op"><span class="hljs-keyword">in</span></span> products]
        <span class="cf"><span class="hljs-keyword">for</span></span> product_id <span class="op"><span class="hljs-keyword">in</span></span> product_ids:
            <span class="cf"><span class="hljs-keyword">for</span></span> with_id <span class="op"><span class="hljs-keyword">in</span></span> product_ids:
                <span class="co"><span class="hljs-comment"># get the other products bought with each product</span></span>
                <span class="cf"><span class="hljs-keyword">if</span></span> product_id <span class="op">!=</span> with_id:
                    <span class="co"><span class="hljs-comment"># increment score for product purchased together</span></span>
                    r.zincrby(<span class="va">self</span>.get_product_key(product_id),
                                    with_id,
                                    amount<span class="op">=</span><span class="dv"><span class="hljs-number">1</span></span>)               </code></pre></div>
<p><code>Recommender</code> 类使我们可以储存购买的产品然后检索所给产品的产品推荐。<code>get_product_key()</code> 方法检索 <code>Product</code> 对象的 <code>id</code> ，然后为储存产品的有序集创建一个 Redis 键（key），看起来像这样：<code>product:[id]:purchased_with</code></p>
<p><code>products_bought()</code> 方法检索被一起购买的产品列表（它们都属于同一个订单）。在这个方法中，我们执行以下几个任务：</p>
<pre><code class="hljs markdown"><span class="hljs-bullet">1. </span>得到所给 <span class="hljs-code">`Product`</span> 对象的产品 ID
<span class="hljs-bullet">2. </span>迭代所有的产品 ID。对于每个 <span class="hljs-code">`id`</span> ，我们迭代所有的产品 ID 并且跳过所有相同的产品，这样我们就可以得到和每个产品一起购买的产品。
<span class="hljs-bullet">3. </span>我们使用 <span class="hljs-code">`get_product_id()`</span> 方法来获取 Redis 产品键。对于一个 ID 为 33 的产品，这个方法返回键 <span class="hljs-code">`product:33:purchased_with`</span> 。这个键用于包含和这个产品被一同购买的产品 ID 的有序集。
<span class="hljs-bullet">4. </span>我们把有序集中的每个产品 <span class="hljs-code">`id`</span> 的评分加一。评分表示另一个产品和所给产品一起购买的次数。</code></pre>
<p>我们有一个方法来保存和对一同购买的产品评分。现在我们需要一个方法来检索被所给购物列表的一起购买的产品。把 <code>suggest_product_for()</code> 方法添加到 <code>Recommender</code> 类中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">suggest_products_for</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, products, max_results</span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="dv"><span class="hljs-function"><span class="hljs-params"><span class="hljs-number">6</span></span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    product_ids <span class="op">=</span> [p.<span class="bu">id</span> <span class="cf"><span class="hljs-keyword">for</span></span> p <span class="op"><span class="hljs-keyword">in</span></span> products]
    <span class="cf"><span class="hljs-keyword">if</span></span> <span class="bu">len</span>(products) <span class="op">==</span> <span class="dv"><span class="hljs-number">1</span></span>:
        <span class="co"><span class="hljs-comment"># only 1 product</span></span>
        suggestions <span class="op">=</span> r.zrange(
                        <span class="va">self</span>.get_product_key(product_ids[<span class="dv"><span class="hljs-number">0</span></span>]),
                        <span class="dv"><span class="hljs-number">0</span></span>, <span class="op"><span class="hljs-number">-</span></span><span class="dv"><span class="hljs-number">1</span></span>, desc<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)[:max_results]
    <span class="cf"><span class="hljs-keyword">else</span></span>:
        <span class="co"><span class="hljs-comment"># generate a temporary key</span></span>
        flat_ids <span class="op">=</span> <span class="st"><span class="hljs-string">''</span></span>.join([<span class="bu">str</span>(<span class="bu">id</span>) <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span> <span class="op"><span class="hljs-keyword">in</span></span> product_ids])
        tmp_key <span class="op">=</span> <span class="st"><span class="hljs-string">'tmp_{}'</span></span>.<span class="bu">format</span>(flat_ids)
        <span class="co"><span class="hljs-comment"># multiple products, combine scores of all products</span></span>
        <span class="co"><span class="hljs-comment"># store the resulting sorted set in a temporary key</span></span>
        keys <span class="op">=</span> [<span class="va">self</span>.get_product_key(<span class="bu">id</span>) <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span> <span class="op"><span class="hljs-keyword">in</span></span> product_ids]
        r.zunionstore(tmp_key, keys)
        <span class="co"><span class="hljs-comment"># remove ids for the products the recommendation is for</span></span>
        r.zrem(tmp_key, <span class="op">*</span>product_ids)
        <span class="co"><span class="hljs-comment"># get the product ids by their score, descendant sort</span></span>
        suggestions <span class="op">=</span> r.zrange(tmp_key, <span class="dv"><span class="hljs-number">0</span></span>, <span class="op"><span class="hljs-number">-</span></span><span class="dv"><span class="hljs-number">1</span></span>,
                        desc<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)[:max_results]
        <span class="co"><span class="hljs-comment"># remove the temporary key</span></span>
        r.delete(tmp_key)
    suggested_products_ids <span class="op">=</span> [<span class="bu">int</span>(<span class="bu">id</span>) <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span> <span class="op"><span class="hljs-keyword">in</span></span> suggestions]

    <span class="co"><span class="hljs-comment"># get suggested products and sort by order of appearance</span></span>
    suggested_products <span class="op">=</span> <span class="bu">list</span>(Product.objects.<span class="bu">filter</span>(id__in<span class="op">=</span>suggested_
    products_ids))
    suggested_products.sort(key<span class="op">=</span><span class="kw"><span class="hljs-keyword">lambda</span></span> x: suggested_products_ids.
    index(x.<span class="bu">id</span>))
    <span class="cf"><span class="hljs-keyword">return</span></span> suggested_products</code></pre></div>
<p><code>suggest_product_for()</code> 方法接收下列参数：</p>
<ul>
<li><code>products</code>：这是一个 <code>Product</code> 对象列表。它可以包含一个或者多个产品</li>
<li><code>max_results</code>：这是一个整数，用于展示返回的推荐的最大数量</li>
</ul>
<p>在这个方法中，我们执行以下的动作：</p>
<pre><code class="hljs basic"><span class="hljs-number">1.</span> 得到所给 `Product` 对象的 ID
<span class="hljs-number">2.</span> 如果只有一个产品，我们就检索一同购买的产品的 ID，并按照他们的购买时间来排序。这样做，我们就可以使用 Redis 的 `ZRANGE` 命令。我们通过 `max_results` 属性来限制结果的数量。
<span class="hljs-number">3.</span> 如果有多于一个的产品被给予，我们就生成一个临时的和产品 ID 一同创建的 Redis 键。
<span class="hljs-number">4.</span> 我们把包含在每个所给产品的有序集中东西的评分组合并相加，我们使用 Redis 的 `ZUNIONSTORE` 命令来实现这个操作。`ZUNIONSTORE` 命令执行了对有序集的所给键的求和，然后在新的 Redis 键中保存每个元素的求和。你可以在这里阅读更多关于这个命令的信息：http://redisio/commands/ZUNIONSTORE 。我们在临时键中保存分数的求和。
<span class="hljs-number">5.</span> 因为我们已经求和了评分，我们或许会获取我们推荐的重复商品。我们就使用 `Z<span class="hljs-comment">REM` 命令来把他们从生成的有序集中删除。</span>
<span class="hljs-number">6.</span> 我们从临时键中检索产品 ID，使用 `Z<span class="hljs-comment">REM` 命令来按照他们的评分排序。我们把结果的数量限制在 `max_results` 属性指定的值内。然后我们删除了临时键。</span>
<span class="hljs-number">7.</span> 最后，我们用所给的 `id` 获取 `Product` 对象，并且按照同样的顺序来排序。</code></pre>
<p>为了更加方便使用，让我们添加一个方法来清除所有的推荐。<br>
把下列方法添加进 <code>Recommender</code> 类中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">clear_purchases</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
    <span class="cf"><span class="hljs-keyword">for</span></span> <span class="bu">id</span> <span class="op"><span class="hljs-keyword">in</span></span> Product.objects.values_list(<span class="st"><span class="hljs-string">'id'</span></span>, flat<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>):
        r.delete(<span class="va">self</span>.get_product_key(<span class="bu">id</span>))</code></pre></div>
<p>让我们试试我们的推荐引擎。确保你在数据库中引入了几个 <code>Product</code> 对象并且在 shell 中使用了下面的命令来初始化 Redis 服务器：</p>
<pre><code class="hljs">src/redis-server</code></pre>
<p>打开另外一个 shell ，执行 <code>python manage.py shell</code> ，输入下面的代码来检索几个产品：</p>
<pre><code class="hljs cs"><span class="hljs-keyword">from</span> shop.models import Product
black_tea = Product.objects.<span class="hljs-keyword">get</span>(translations__name=<span class="hljs-string">'Black tea'</span>)
red_tea = Product.objects.<span class="hljs-keyword">get</span>(translations__name=<span class="hljs-string">'Red tea'</span>)
green_tea = Product.objects.<span class="hljs-keyword">get</span>(translations__name=<span class="hljs-string">'Green tea'</span>)
tea_powder = Product.objects.<span class="hljs-keyword">get</span>(translations__name=<span class="hljs-string">'Tea powder'</span>)</code></pre>
<p>然后，添加一些测试购物到推荐引擎中：</p>
<pre><code class="hljs prolog">from shop.recommender import <span class="hljs-symbol">Recommender</span>
r = <span class="hljs-symbol">Recommender</span>()
r.products_bought([black_tea, red_tea])
r.products_bought([black_tea, green_tea])
r.products_bought([red_tea, black_tea, tea_powder])
r.products_bought([green_tea, tea_powder])
r.products_bought([black_tea, tea_powder])
r.products_bought([red_tea, green_tea])</code></pre>
<p>我们已经保存了下面的评分：</p>
<pre><code class="hljs yaml"><span class="hljs-attr">black_tea:</span> red_tea (<span class="hljs-number">2</span>), tea_powder (<span class="hljs-number">2</span>), green_tea (<span class="hljs-number">1</span>)
<span class="hljs-attr">red_tea:</span> black_tea (<span class="hljs-number">2</span>), tea_powder (<span class="hljs-number">1</span>), green_tea (<span class="hljs-number">1</span>)
<span class="hljs-attr">green_tea:</span> black_tea (<span class="hljs-number">1</span>), tea_powder (<span class="hljs-number">1</span>), red_tea(<span class="hljs-number">1</span>)
<span class="hljs-attr">tea_powder:</span> black_tea (<span class="hljs-number">2</span>), red_tea (<span class="hljs-number">1</span>), green_tea (<span class="hljs-number">1</span>)</code></pre>
<p>让我们看看为单一产品的推荐吧：</p>
<pre><code class="hljs fsharp">&gt;&gt;&gt; r.suggest_products_for([black_tea])
<span class="hljs-meta">[&lt;Product: Tea powder&gt;, &lt;Product: Red tea&gt;, &lt;Product: Green tea&gt;]</span>
&gt;&gt;&gt; r.suggest_products_for([red_tea])
<span class="hljs-meta">[&lt;Product: Black tea&gt;, &lt;Product: Tea powder&gt;, &lt;Product: Green tea&gt;]</span>
&gt;&gt;&gt; r.suggest_products_for([green_tea])
<span class="hljs-meta">[&lt;Product: Black tea&gt;, &lt;Product: Tea powder&gt;, &lt;Product: Red tea&gt;]</span>
&gt;&gt;&gt; r.suggest_products_for([tea_powder])
<span class="hljs-meta">[&lt;Product: Black tea&gt;, &lt;Product: Red tea&gt;, &lt;Product: Green tea&gt;]</span></code></pre>
<p>正如你所看到的那样，推荐产品的顺序正式基于他们的评分。让我们来看看为多个产品的推荐吧：</p>
<pre><code class="hljs fsharp">&gt;&gt;&gt; r.suggest_products_for([black_tea, red_tea])
<span class="hljs-meta">[&lt;Product: Tea powder&gt;, &lt;Product: Green tea&gt;]</span>
&gt;&gt;&gt; r.suggest_products_for([green_tea, red_tea])
<span class="hljs-meta">[&lt;Product: Black tea&gt;, &lt;Product: Tea powder&gt;]</span>
&gt;&gt;&gt; r.suggest_products_for([tea_powder, black_tea])
<span class="hljs-meta">[&lt;Product: Red tea&gt;, &lt;Product: Green tea&gt;]</span></code></pre>
<p>你可以看到推荐产品的顺序和评分总和的顺序是一样的。比如 <code>black_tea</code>, <code>red_tea</code>, <code>tea_powder(2+1)</code> 和 <code>green_tea(1=1)</code> 的产品推荐就是这样。</p>
<p>我们必须保证我们的推荐算法按照预期那样工作。让我们在我们的站点上展示我们的推荐吧。</p>
<p>编辑 <code>shop</code> 应用的 <code>views.py</code> ，添加以下库：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .recommender <span class="im"><span class="hljs-keyword">import</span></span> Recommender</code></pre></div>
<p>把下面的代码添加进 <code>product_detail</code> 视图（view）中的 <code>render()</code> 之前：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">r <span class="op">=</span> Recommender()
recommended_products <span class="op">=</span> r.suggest_products_for([product], <span class="dv"><span class="hljs-number">4</span></span>)</code></pre></div>
<p>我们得到了四个最多产品推荐。 <code>product_detail</code> 视图（view）现在看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> .recommender <span class="im"><span class="hljs-keyword">import</span></span> Recommender

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">product_detail</span><span class="hljs-params">(request, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">id</span></span></span><span class="hljs-function"><span class="hljs-params">, slug)</span>:</span>
    product <span class="op">=</span> get_object_or_404(Product,
                                <span class="bu">id</span><span class="op">=</span><span class="bu">id</span>,
                                slug<span class="op">=</span>slug,
                                available<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    cart_product_form <span class="op">=</span> CartAddProductForm()
    r <span class="op">=</span> Recommender()
    recommended_products <span class="op">=</span> r.suggest_products_for([product], <span class="dv"><span class="hljs-number">4</span></span>)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                    <span class="st"><span class="hljs-string">'shop/product/detail.html'</span></span>,
                    {<span class="st"><span class="hljs-string">'product'</span></span>: product,
                    <span class="co"><span class="hljs-string">'cart_product_form'</span></span>: cart_product_form,
                    <span class="co"><span class="hljs-string">'recommended_products'</span></span>: recommended_products})</code></pre></div>
<p>现在编辑 <code>shop</code> 应用的 <code>shop/product/detail.html</code> 模板（template），把以下代码添加在 <code><span>{</span><span>{</span> product.description|linebreaks <span>}</span><span>}</span></code> 的后面：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> if recommended_products <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"recommendations"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span><span>{</span><span>%</span> trans "People who bought this also bought" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
<span>{</span><span>%</span> for p in recommended_products <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"item"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> p.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">img</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> if p.image <span>%</span><span>}</span><span>{</span><span>{</span> p.image.url <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span><span>{</span><span>%</span></span></span></span><span class="hljs-tag"><span class="hljs-string">
</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">img</span>/<span class="hljs-attr">no_image.png</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> p.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> p.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span></code></pre></div>
<p>运行开发服务器，访问：<a href="http://127.0.0.1:8000/en/" class="uri">http://127.0.0.1:8000/en/</a> 。点击一个产品来查看它的详情页。你应该看到展示在下方的推荐产品，就象这样：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-13.png" alt="django-9-13"></p>
<p>我们也将在购物车当中引入产品推荐。产品推荐将会基于用户购物车中添加的产品。编辑 <code>cart</code> 应用的 <code>views.py</code> ，添加以下库：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> shop.recommender <span class="im"><span class="hljs-keyword">import</span></span> Recommender</code></pre></div>
<p>编辑 <code>cart_detail</code> 视图（view），让它看起来像这样：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">cart_detail</span><span class="hljs-params">(request)</span>:</span>
    cart <span class="op">=</span> Cart(request)
    <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> cart:
        item[<span class="st"><span class="hljs-string">'update_quantity_form'</span></span>] <span class="op">=</span> CartAddProductForm(
                        initial<span class="op">=</span>{<span class="st"><span class="hljs-string">'quantity'</span></span>: item[<span class="st"><span class="hljs-string">'quantity'</span></span>],
                                <span class="st"><span class="hljs-string">'update'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>})
    coupon_apply_form <span class="op">=</span> CouponApplyForm()
    
    r <span class="op">=</span> Recommender()
    cart_products <span class="op">=</span> [item[<span class="st"><span class="hljs-string">'product'</span></span>] <span class="cf"><span class="hljs-keyword">for</span></span> item <span class="op"><span class="hljs-keyword">in</span></span> cart]
    recommended_products <span class="op">=</span> r.suggest_products_for(cart_products,
                                max_results<span class="op">=</span><span class="dv"><span class="hljs-number">4</span></span>)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                <span class="st"><span class="hljs-string">'cart/detail.html'</span></span>,
                {<span class="st"><span class="hljs-string">'cart'</span></span>: cart,
                <span class="co"><span class="hljs-string">'coupon_apply_form'</span></span>: coupon_apply_form,
                <span class="co"><span class="hljs-string">'recommended_products'</span></span>: recommendeproducts})</code></pre></div>
<p>编辑 <code>cart</code> 应用的 <code>cart/detail.html</code> ，把下列代码添加在 <code>&lt;table&gt;</code>HTML 标签后面：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> if recommended_products <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"recommendations cart"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span><span>{</span><span>%</span> trans "People who bought this also bought" <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
<span>{</span><span>%</span> for p in recommended_products <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"item"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> p.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">img</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> if p.image <span>%</span><span>}</span><span>{</span><span>{</span> p.image.url <span>}</span><span>}</span><span>{</span><span>%</span> else <span>%</span><span>}</span><span>{</span><span>%</span></span></span></span><span class="hljs-tag"><span class="hljs-string">
</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">img</span>/<span class="hljs-attr">no_image.png</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span><span>{</span><span>%</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">endif</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>{</span> p.get_absolute_url <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> p.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endfor <span>%</span><span>}</span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endif <span>%</span><span>}</span></code></pre></div>
<p>访问 <a href="http://127.0.0.1:8000/en/" class="uri">http://127.0.0.1:8000/en/</a> ，在购物车当中添加一些商品。东航拟导航向 <a href="http://127.0.0.1:8000/en/cart/" class="uri">http://127.0.0.1:8000/en/cart/</a> 时，你可以在购物车下看到锐减的产品，就像下面这样：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-9-14.png" alt="django-9-14"></p>
<p>恭喜你！你已经使用 Django 和 Redis 构建了一个完整的推荐引擎。</p>
<h2 id="总结"><strong>总结</strong></h2>
<p>在这一章中，你使用会话创建了一个优惠券系统。你学习了国际化和本地化是如何工作的。你也使用 Redis 构建了一个推荐引擎。</p>
<p>在下一章中，你将开始一个新的项目。你将会用 Django 创建一个在线学习平台，并且使用基于类的视图（view）。你将学会创建一个定制的内容管理系统（CMS）。</p>
</div>