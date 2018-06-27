---
layout: single
permalink: /django/example8/
title: "Django By Example 第八章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第八章">第八章</h1>
<h2 id="管理付款和订单">管理付款和订单</h2>
<p>在上一章，你创建了一个基础的在线商店包含一个产品列表以及订单系统。你还学习了如何执行异步的任务通过使用Celery。在这一章中，你会学习到如何集成一个支付网关<strong>（译者注：支付网关（Payment Gateway）是银行金融网络系统和Internet网络之间的接口，是由银行操作的将Internet上传输的数据转换为金融机构内部数据的一组服务器设备，或由指派的第三方处理商家支付信息和顾客的支付指令。以上是我百度的。）</strong>到你的站点中。你还会扩展管理平台站点来管理订单和用不同的格式导出它们。</p>
<p>在这一章中，我们会覆盖以下几点：</p>
<ul>
<li>集成一个支付网关到你的站点中</li>
<li>管理支付通知</li>
<li>导出订单为CSV格式</li>
<li>创建定制视图给管理页面</li>
<li>动态的生成PDF支票</li>
</ul>
<h2 id="集成一个支付网关">集成一个支付网关</h2>
<p>一个支付网关允许你在线处理支付。通过使用一个支付网关，你可以管理顾客的订单以及委托一个可靠的，安全的第三方处理支付。这意味着你无需担心存储信用卡信息到你的系统中。</p>
<p>PayPal 提供了多种方法来集成它的网管到你的站点中。标准的集成由一个<em>Buy now</em>按钮组成，这个按钮你可以已经在别的网站见到过（译者注：国内还是支付宝和微信比较多）。这个按钮会重定向购买者到PayPal去处理支付。我们将要集成PayPal支付标准包含一个定制的<em>Buy now</em>按钮到我们的站点中。PayPal将会处理支付并且发送一个消息通知给我们的服务指明该笔支付的状态。</p>
<h2 id="创建一个paypal账户">创建一个PayPal账户</h2>
<p>你需要有一个PayPal商业账户来集成支付网关到你的站点中。如果你还没有一个PayPal账户，去 <a href="https://www.paypal.com/signup/account" class="uri">https://www.paypal.com/signup/account</a> 注册。确保你选择了一个<em>Bussiness Account</em>并且注册成为PayPal支付标准解决方案，如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-0.png" alt="django-8-0"></p>
<p>填写你的详情在注册表单中并且完成注册流程。PayPal会发送给你一封e-mail来核对你的账户。</p>
<h2 id="安装django-paypal">安装django-paypal</h2>
<p>Django-paypal是一个第三方django应用，它可以简化集成PayPal到Django项目中。我们将要使用它来集成PayPal支付标准解决方案到我们的商店中。你可以找到django-paypal的文档，访问 <a href="http://django-paypal.readthedocs.org/" class="uri">http://django-paypal.readthedocs.org/</a>。</p>
<p>安装django-paypal在shell中通过以下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> django-paypal==<span class="hljs-number">0.2</span>.<span class="hljs-number">5</span> </code></pre>
<p><strong>(译者注：现在应该有最新版本，书上使用的是0.2.5版本)</strong></p>
<p>编辑你的项目中的<em>settings.py</em>文件，添加'paypal.standard.ipn'到<em>INSTALLED_APPS</em>设置中，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'paypal.standard.ipn'</span></span>,
)</code></pre></div>
<p>这个应用提供自django-paypal来集成PayPal支付标准通过<strong>Instant Payment Notification(IPN)</strong>。我们之后会操作支付通知。</p>
<p>添加以下设置到<em>myshop</em>的<em>settings.py</em>文件来配置django-paypal：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># django-paypal settings</span></span>
PAYPAL_RECEIVER_EMAIL <span class="op">=</span> <span class="st"><span class="hljs-string">'mypaypalemail@myshop.com'</span></span>
PAYPAL_TEST <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span></code></pre></div>
<p>以上两个设置含义如下：</p>
<ul>
<li>PAYPAL_RECEIVER_EMAIL：你的PayPal账户e-mail。<a>使用你创建的PayPal账户e-mail替换*mypaypalemail@myshop.com*</a>。</li>
<li>PAYPAL_TEST：一个布尔类型指示是否PayPal的沙箱环境，该环境可以用来处理支付。这个沙箱允许你测试你的PayPal集成在迁移到一个正式生产的环境之前。</li>
</ul>
<p>打开shell运行如下命令来同步django-paypal的模型（models）到数据库中：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">migrate</span></code></pre>
<p>你会看到如下类似的输出：</p>
<pre class="shell"><code class="hljs css"><span class="hljs-selector-tag">Running</span> <span class="hljs-selector-tag">migrations</span>:
    <span class="hljs-selector-tag">Rendering</span> <span class="hljs-selector-tag">model</span> <span class="hljs-selector-tag">states</span>... <span class="hljs-selector-tag">DONE</span>
    <span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">ipn</span><span class="hljs-selector-class">.0001_initial</span>... <span class="hljs-selector-tag">OK</span>
    <span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">ipn</span><span class="hljs-selector-class">.0002_paypalipn_mp_id</span>... <span class="hljs-selector-tag">OK</span>
    <span class="hljs-selector-tag">Applying</span> <span class="hljs-selector-tag">ipn</span><span class="hljs-selector-class">.0003_auto_20141117_1647</span>... <span class="hljs-selector-tag">OK</span></code></pre>
<p>django-paypal的模型（models）如今已经同步到了数据库中。你还需要添加django-paypal的URL模式到你的项目中。编辑主的<em>urls.py</em>文件，该文件位于<em>myshop</em>目录，然后添加以下的URL模式。记住粘贴该URL模式要在<em>shop.urls</em>模式之前为了避免错误的模式匹配：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^paypal/'</span>, include(<span class="hljs-string">'paypal.standard.ipn.urls'</span>)),</code></pre>
<p>让我们添加支付网关到结账流程中。</p>
<h2 id="添加支付网关">添加支付网关</h2>
<p>结账流程工作如下：</p>
<ul>
<li>1.用户添加物品到他们的购物车中</li>
<li>2.用户结账他们的购物车</li>
<li>3.用户被重定向到PayPal进行支付</li>
<li>4.PayPal发送一个支付通知给我们的站点</li>
<li>5.PayPal重定向用户回到我们的网站</li>
</ul>
<p>创建一个新的应用到你的项目中使用如下命令：</p>
<pre><code class="hljs css"><span class="hljs-selector-tag">python</span> <span class="hljs-selector-tag">manage</span><span class="hljs-selector-class">.py</span> <span class="hljs-selector-tag">startapp</span> <span class="hljs-selector-tag">payment</span></code></pre>
<p>我们将要使用这个应用去管理结账过程和用户支付。</p>
<p>编辑你的项目的<em>settings.py</em>文件，添加'payment'到<em>INSTALLED_APPS</em>设置中，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
    <span class="co"><span class="hljs-comment"># ...</span></span>
    <span class="st"><span class="hljs-string">'paypal.standard.ipn'</span></span>,
    <span class="co"><span class="hljs-string">'payment'</span></span>,
)</code></pre></div>
<p><em>payment</em>应用现在已经在项目中激活。编辑<em>orders</em>应用的<em>views.py</em>文件并且确保包含以下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, redirect
<span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse</code></pre></div>
<p>替换以下<em>order_create</em>视图（view）的内容：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># launch asynchronous task</span></span>
order_created.delay(order.<span class="bu">id</span>)
<span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'orders/order/created.html'</span></span>, <span class="bu">locals</span>())</code></pre></div>
<p>新的内容为：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># launch asynchronous task</span></span>
order_created.delay(order.<span class="bu">id</span>) <span class="co"><span class="hljs-comment"># set the order in the session</span></span>
request.session[<span class="st"><span class="hljs-string">'order_id'</span></span>] <span class="op">=</span> order.<span class="bu">id</span> <span class="co"><span class="hljs-comment"># redirect to the payment</span></span>
<span class="cf"><span class="hljs-keyword">return</span></span> redirect(reverse(<span class="st"><span class="hljs-string">'payment:process'</span></span>))</code></pre></div>
<p>在成功的创建一个新的订单之后，我们设置这个订单ID到当前的会话中使用<em>order_id</em>会话键（session key）。之后，我们重定向用户到<code>payment:process</code>URL，这个我们下一步就是创建。</p>
<p>编辑<em>payment</em>应用的<em>views.py</em>文件然后添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> decimal <span class="im"><span class="hljs-keyword">import</span></span> Decimal
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse
<span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> render, get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> paypal.standard.forms <span class="im"><span class="hljs-keyword">import</span></span> PayPalPaymentsForm
<span class="im"><span class="hljs-keyword">from</span></span> orders.models <span class="im"><span class="hljs-keyword">import</span></span> Order

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">payment_process</span><span class="hljs-params">(request)</span>:</span>
    order_id <span class="op">=</span> request.session.get(<span class="st"><span class="hljs-string">'order_id'</span></span>)
    order <span class="op">=</span> get_object_or_404(Order, <span class="bu">id</span><span class="op">=</span>order_id)
    host <span class="op">=</span> request.get_host()
    paypal_dict <span class="op">=</span> {
        <span class="st"><span class="hljs-string">'business'</span></span>: settings.PAYPAL_RECEIVER_EMAIL,
        <span class="co"><span class="hljs-string">'amount'</span></span>: <span class="st"><span class="hljs-string">'</span></span><span class="sc"><span class="hljs-string">%.2f</span></span><span class="st"><span class="hljs-string">'</span></span> <span class="op">%</span> order.get_total_cost().quantize(
                                                Decimal(<span class="st"><span class="hljs-string">'.01'</span></span>)),
        <span class="co"><span class="hljs-string">'item_name'</span></span>: <span class="st"><span class="hljs-string">'Order {}'</span></span>.<span class="bu">format</span>(order.<span class="bu">id</span>),
        <span class="co"><span class="hljs-string">'invoice'</span></span>: <span class="bu">str</span>(order.<span class="bu">id</span>),
        <span class="co"><span class="hljs-string">'currency_code'</span></span>: <span class="st"><span class="hljs-string">'USD'</span></span>,
        <span class="co"><span class="hljs-string">'notify_url'</span></span>: <span class="st"><span class="hljs-string">'http://{}{}'</span></span>.<span class="bu">format</span>(host,
                                        reverse(<span class="st"><span class="hljs-string">'paypal-ipn'</span></span>)),
        <span class="co"><span class="hljs-string">'return_url'</span></span>: <span class="st"><span class="hljs-string">'http://{}{}'</span></span>.<span class="bu">format</span>(host,
                                        reverse(<span class="st"><span class="hljs-string">'payment:done'</span></span>)),
        <span class="co"><span class="hljs-string">'cancel_return'</span></span>: <span class="st"><span class="hljs-string">'http://{}{}'</span></span>.<span class="bu">format</span>(host,
                                    reverse(<span class="st"><span class="hljs-string">'payment:canceled'</span></span>)),
       }
       form <span class="op">=</span> PayPalPaymentsForm(initial<span class="op">=</span>paypal_dict)
       <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                     <span class="st"><span class="hljs-string">'payment/process.html'</span></span>,
                     {<span class="st"><span class="hljs-string">'order'</span></span>: order, <span class="st"><span class="hljs-string">'form'</span></span>:form})</code></pre></div>
<p>在<code>payment_process</code>视图（view）中，我们生成了一个PayPal的<strong>Buy now</strong>按钮用来支付一个订单。首先，我们拿到当前的订单从<code>order_id</code>会话键中，这个键值被之前的<code>order_create</code>视图（view）设置。我们拿到这个<code>order</code>对象通过给予的ID并且构建一个新的<code>PayPalPaymentsForm</code>，该表单表单包含以下字段：</p>
<ul>
<li>business：PayPal商业账户用来处理支付。我们使用e-mail账户，该账户定义在<code>PAYPAL_RECEIVER_EMAIL</code>设置那里。</li>
<li>amount：向顾客索要的总价。</li>
<li>item_name：正在出售的商品名。我们使用订单ID，因为订单可能包含很多产品。</li>
<li>currency_code：本次支付的货币。我们设置这里为USD使用U.S. Dollar<strong>(译者注：传说中的美金)</strong>。需要使用相同的货币，该货币被设置在你的PayPal账户中（例如：EUR 对应欧元）。</li>
<li>notify_url：这个URL PayPal将会发送IPN请求过去。我们使用django-paypal提供的<code>paypal-ipn</code> URL。这个视图（view）与这个URL关联来操作支付通知以及存储它们到数据库中。</li>
<li>return_url：这个URL用来重定向用户当他的支付成功之后。我们使用URL <code>payment:done</code>，这个我们接下来会创建。</li>
<li>cancel_return：这个URL用来重定向用户如果这个支付被取消或者有其他问题。我们使用URL <code>payment:canceled</code>，这个我们接下来会创建。</li>
</ul>
<p><code>PayPalpaymentsForm</code>将会被渲染成一个标准表单带有隐藏的字段，并且用户将来只能看到<strong>Buy now</strong>按钮。当用户点击该按钮，这个表单将会提交到PayPal通过POST渠道。</p>
<p>让我们创建简单的视图（views）给PayPal用来重定向用户当支付成功，或者当支付被取消因为某些原因。添加以下代码到相同的<em>views.py</em>文件：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.views.decorators.csrf <span class="im"><span class="hljs-keyword">import</span></span> csrf_exempt

<span class="at"><span class="hljs-meta">@csrf_exempt</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">payment_done</span><span class="hljs-params">(request)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'payment/done.html'</span></span>)
    
<span class="at"><span class="hljs-meta">@csrf_exempt</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">payment_canceled</span><span class="hljs-params">(request)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request, <span class="st"><span class="hljs-string">'payment/canceled.html'</span></span>)</code></pre></div>
<p>我们使用<code>csrf_exempt</code>装饰器来避免Django期待一个CSRF标记，因为PayPal能重定向用户到以上两个视图（views）通过POST渠道。创建新的文件在<em>payment</em>应用目录下并且命名为<em>urls.py</em>。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views
urlpatterns <span class="op">=</span> [
    url(<span class="vs"><span class="hljs-string">r'^process/$'</span></span>, views.payment_process, name<span class="op">=</span><span class="st"><span class="hljs-string">'process'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^done/$'</span></span>, views.payment_done, name<span class="op">=</span><span class="st"><span class="hljs-string">'done'</span></span>),
    url(<span class="vs"><span class="hljs-string">r'^canceled/$'</span></span>, views.payment_canceled, name<span class="op">=</span><span class="st"><span class="hljs-string">'canceled'</span></span>),
]</code></pre></div>
<p>这些URL是给支付工作流的。我们已经包含了以下URL模式：</p>
<ul>
<li>process：给这个视图（view）用来生成PayPal表单给<strong>Buy now</strong>按钮。</li>
<li>done：给PayPal用来重定向用户当支付成功的时候。</li>
<li>canceled：给PayPal用来重定向用户当支付取消的时候。</li>
</ul>
<p>编辑主的<em>myshop</em>项目的<em>urls.py</em>文件，包含URL模式给<em>payment</em>应用：</p>
<pre><code class="hljs python">url(<span class="hljs-string">r'^payment/'</span>, include(<span class="hljs-string">'payment.urls'</span>,namespace=<span class="hljs-string">'payment'</span>)),</code></pre>
<p>记住粘贴以上内容在<code>shop.urls</code>模式之前用来避免错误的模式匹配。</p>
<p>创建以下文件建构在<em>payment</em>应用目录下：</p>
<pre class="shell"><code class="hljs fsharp">templates/
    payment/
        process.html
        <span class="hljs-keyword">done</span>.html
        canceled.html</code></pre>
<p>编辑<em>payment/process.html</em>模板（template）并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>Pay using PayPal<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Pay using PayPal<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span>{</span><span>{</span> form.render <span>}</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板（template）会渲染<code>PayPalPaymentsForm</code>并且展示<strong>Buy now</strong>按钮。</p>
<p>编辑<em>payment/done.html</em>模板（template）并且添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Your payment was successful<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Your payment has been successfully received.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板（template）的页面给用户重定向当成功支付之后。</p>
<p>编辑<em>payment/canceled.html</em>模板（template）并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "shop/base.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> block content <span>%</span><span>}</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Your payment has not been processed<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>There was a problem processing your payment.<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板（template）的页面给用户重定向当有这个支付过程出现问题或者用户取消了这次支付。</p>
<p>让我们尝试完成的支付过程。</p>
<h2 id="使用paypal的沙箱">使用PayPal的沙箱</h2>
<p>打开 <a href="http://developer.paypal.com/" class="uri">http://developer.paypal.com</a> 在你的浏览器中然后进行登录使用你的PayPal商业账户。点击<strong>Dashboard</strong>菜单项，在左方菜单点击<strong>Accounts</strong>选项在<strong>Sandbox</strong>下方。你会看到你的沙箱测试账户列，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-1.png" alt="django-8-1"></p>
<p>一开始，你将会看到一个商业以及一个个人测试账户由PayPal动态创建。你可以创建新的沙箱测试账户通过使用<strong>Create Account</strong>按钮。</p>
<p>点击<strong>Personal Account</strong>在列中扩大它，之后点击<strong>Profile</strong>链接。你会看到一些信息关于这个测试账户包含e-mail和profile信息，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-2.png" alt="django-8-2"></p>
<p>在<strong>Funding</strong> tab中，你会找到银行账户，信用卡日期，以及PayPal信用余额。</p>
<p>这些测试账户能够被用来做支付在你的网站中当使用沙箱环境。跳转到<strong>Profile</strong> tab然后点击<strong>Change password</strong>链接。创建一个定制密码给这个测试账户。</p>
<p>打开shell并且启动开发服务器使用命令<code>python manage.py runserver</code>。打开 <a href="http://127.0.0.1:8000/" class="uri">http://127.0.0.1:8000</a> 在你的浏览器中，添加一些产品到购物车中，并且填写结账表单。当你点击<strong>Place order</strong>按钮，这个订单会被保存在数据库中，这个订单ID会被保存在当前的会话中，并且你会被重定向到支付处理页面。这个页面从会话中获取订单并且渲染PayPal表单显示一个<strong>Buy now</strong>按钮，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-3.png" alt="django-8-3"></p>
<p>你可以看下HTML源码来看下生成的表单字段。</p>
<p>点击<strong>Buy now</strong>按钮。你会被重定向到PayPal，并且你会看到如下页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-4.png" alt="django-8-4"></p>
<p>输入购买者测试账户e-mail和密码然后点击<strong>Log In</strong>按钮。你会被重定向到以下页面：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-5.png" alt="django-8-5"></p>
<p>现在，点击<strong>Pay now</strong>按钮。最后，你会看到批准页面该页面包含你的交易ID。这个页面看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-6.png" alt="django-8-6"></p>
<p>点击**Return to <a>e-mail@domain.com**按钮</a>。你会被重定向到的URL是你之前在<code>PayPalPaymentsForm</code>中的<code>return_url</code>字段中定义的。这个URL对应<code>payment_done</code>视图（view）。这个页面看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-7.png" alt="django-8-7"></p>
<p>这个支付已经成功了。然而，PayPal并没有发送一个支付状态通知给我们的应用，因为我们运行我们的项目在我们本地主机，IP是 127.0.0.1 这并不是一个公开地址。我们将要学习如何使我们的站点可以从Internet访问并且接收IPN通知。</p>
<h2 id="获取支付通知">获取支付通知</h2>
<p>IPN是一个方法提供自大部分的支付网关用来跟踪实时的购买。一个通知会立即发送到你的服务当这个网关处理了一个支付。这个通知包含所有支付详情，包括状态以及一个支付的签名，该签名可以用来确定这个消息的来源点。这个消息被发送通过一个单独的HTTP请求给你的服务。在出现连接问题的情况下，PayPal将会多次企图通知你的站点。</p>
<p>django-paypal应用内置两种不同的信号给IPNs。如下：</p>
<ul>
<li>valid_ipn_received：会被触发当IPN信息获取自PayPal是正确的并且不是一个已存在数据库中的消息的复制。</li>
<li>invalid_ipn_received：这个信号会触发当IPN获取自PayPal包含无效的数据或者不是一个良好的形式。</li>
</ul>
<p>我们将要创建一个定制的接受函数并且连接它给<code>valid_ipn_received</code>信号用来确定支付。</p>
<p>创建新的文件在<em>payment</em>应用目录下，并且命名为<em>signals.py</em>，添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> paypal.standard.models <span class="im"><span class="hljs-keyword">import</span></span> ST_PP_COMPLETED
<span class="im"><span class="hljs-keyword">from</span></span> paypal.standard.ipn.signals <span class="im"><span class="hljs-keyword">import</span></span> valid_ipn_received
<span class="im"><span class="hljs-keyword">from</span></span> orders.models <span class="im"><span class="hljs-keyword">import</span></span> Order

<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">payment_notification</span><span class="hljs-params">(sender, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
    ipn_obj <span class="op">=</span> sender
    <span class="cf"><span class="hljs-keyword">if</span></span> ipn_obj.payment_status <span class="op">==</span> ST_PP_COMPLETED:
        <span class="co"><span class="hljs-comment"># payment was successful</span></span>
        order <span class="op">=</span> get_object_or_404(Order, <span class="bu">id</span><span class="op">=</span>ipn_obj.invoice)
        <span class="co"><span class="hljs-comment"># mark the order as paid</span></span>
        order.paid <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
        order.save()
        
valid_ipn_received.<span class="ex">connect</span>(payment_notification)</code></pre></div>
<p>我们连接<code>payment_notification</code>接收函数给django-paypal提供的<code>valid_ipn_received</code>信号。这个接收函数工作如下：</p>
<ul>
<li>1.我们获取发送对象，该对象是一个<em>PayPalIPN</em>模型的实例，位于<code>paypal.standard.ipn.models</code>。</li>
<li>2.我们检查<code>payment_status</code>属性来确保它和django-payapl的完整状态相同。这个状态指示这个支付已经成功处理。</li>
<li>3.之后我们使用<code>get_object_or_404()</code>快捷函数来拿到订单，该订单的ID匹配<code>invoice</code>参数我们之前提供给PayPal。</li>
<li>4.我们备注这个订单已经支付通过设置它的<code>paid</code>属性为<code>True</code>并且保存这个订单对象到数据库中。</li>
</ul>
<p>你需要确保你的信号方法已经加载，这样这个接收函数会被调用当<code>valid_ipn_received</code>信号被触发的时候。The best practice is to load your signals when the application containing them is loaded. <strong>（译者注：谁帮我翻一下，好拗口啊）</strong>。这能够实现通过定义一个定制应用配置，这方面会在下一节进行解释。</p>
<h2 id="配置我们的应用">配置我们的应用</h2>
<p>你已经学习了关于应用的配置在<em>第六章 跟踪用户操作</em>。我们将要定义一个定制配置给我们的<em>payment</em>应用为了加载我们的信号接收函数。</p>
<p>创建一个新的文件在<em>payment</em>应用目录下命名为<em>apps.py</em>。添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.apps <span class="im"><span class="hljs-keyword">import</span></span> AppConfig

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">PaymentConfig</span><span class="hljs-params">(AppConfig)</span>:</span>
    name <span class="op">=</span> <span class="st"><span class="hljs-string">'payment'</span></span>
    verbose_name <span class="op">=</span> <span class="st"><span class="hljs-string">'Payment'</span></span>
    
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">ready</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        <span class="co"><span class="hljs-comment"># import signal handlers</span></span>
        <span class="im"><span class="hljs-keyword">import</span></span> payment.signals</code></pre></div>
<p>在上述代码中，我们定义了一个定制<code>AppConfif</code>类给<em>payment</em>应用。<code>name</code>参数是这个应用的名字，<em>verbose_name</em>包含可读的样式。我们导入信号方法在<code>ready()</code>方法中确保它们会被加载当这个应用初始化的时候。</p>
<p>编辑<em>payment</em>应用的<em><strong>init</strong>.py</em>文件，添加以下行：</p>
<pre><code class="hljs ini"><span class="hljs-attr">default_app_config</span> = <span class="hljs-string">'payment.apps.PaymentConfig'</span></code></pre>
<p>以上操作可以使Django动态加载你的定制应用配置类。你可以找到更进一步的信息关于应用配置，通过访问 <a href="https://docs.djangoproject.com/en/1.8/ref/applications/" class="uri">https://docs.djangoproject.com/en/1.8/ref/applications/</a> 。</p>
<h2 id="测试支付通知">测试支付通知</h2>
<p>由于我们工作在本地环境中，我们需要确保我们的站点可以被PayPal获得。有不少应用允许你使你的开发环境在Internet中可获得。我们将要使用Ngrok，它就是其中一个最著名的。</p>
<pre><code class="hljs">./ngrok http 8000</code></pre>
<p>通过这条命名，你告诉Ngrok去创建一条隧道给你的本地主机在端口8000上并且分配一个Internet可访问主机名给它。你可以看到如下类似输出：</p>
<pre class="shell"><code class="hljs groovy">Tunnel Status     online
Version           <span class="hljs-number">2.0</span><span class="hljs-number">.17</span>/<span class="hljs-number">2.0</span><span class="hljs-number">.17</span>
Web Interface     <span class="hljs-string">http:</span><span class="hljs-comment">//127.0.0.1:4040</span>
Forwarding        <span class="hljs-string">http:</span><span class="hljs-comment">//1a1b50f2.ngrok.io -&gt; localhost:8000</span>
Forwarding        <span class="hljs-string">https:</span><span class="hljs-comment">//1a1b50f2.ngrok.io -&gt; localhost:8000</span>

Connnections      ttl     opn     rt1     rt5     p50     p90
                  <span class="hljs-number">0</span>       <span class="hljs-number">0</span>       <span class="hljs-number">0.00</span>    <span class="hljs-number">0.00</span>    <span class="hljs-number">0.00</span>    <span class="hljs-number">0.00</span></code></pre>
<p>Ngrok告诉我们关于我们的站点，运行在本地8000端口使用Django开发服务器，已经可以在Internet访问到通过URLs <a href="http://1a1b50f2.ngrok.io/" class="uri">http://1a1b50f2.ngrok.io</a> 以及 <a href="https://1a1b50f2.ngrok.io/" class="uri">https://1a1b50f2.ngrok.io</a> ，前者是HTTP，后者是HTTPS。Ngrok还提供一个URL来访问一个web接口用来显示信息关于发送到这个服务的请求。</p>
<p>打开Ngrok提供的URL在浏览器中；例如，<a href="http://1a1b50f2.ngrok.io/" class="uri">http://1a1b50f2.ngrok.io</a> 。添加一些产品到购物车中，放置一个订单，然后使用你的PayPal测试账户进行支付。这个时候，PayPal将能够拿到这个URL，这个URL由<code>PayPalPaymentsForm</code>的<code>notify_url</code>字段生成，在<code>payment_process</code>视图（view）中。如果你看一下这个渲染过的表单，你会看到这个HTML表单字段看上去如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">input</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">id</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"id_notify_url"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">name</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"notify_url"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"hidden"</span></span></span><span class="hljs-tag">
</span><span class="ot"><span class="hljs-tag"><span class="hljs-attr">value</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"http://1a1b50f2.ngrok.io/paypal/"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span></code></pre></div>
<p>在结束支付过程之后，打开 <a href="http://127.0.0.1:8000/admin/ipn/paypalipn/" class="uri">http://127.0.0.1:8000/admin/ipn/paypalipn/</a> 在你的浏览器中。你会看到一个IPN对象对应最新的支付状态为<strong>Completed</strong>。这个对象包含所有的支付信息，该对象由PayPal发送给你提供给IPN通知的URL。IPN管理列展示页面看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-8.png" alt="django-8-8"></p>
<p>你还可以启动IPNs通过使用PayPal的IPN模拟器位于 <a href="https://developer.paypal.com/developer/ipnSimulator/" class="uri">https://developer.paypal.com/developer/ipnSimulator/</a> 。这个模拟器允许你指定字段和发送的通知类型。</p>
<p>除了PayPal支付标准外，PayPal提供Website Payments Pro，它是一个订购服务允许你接受支付在你的站点中而不需要重定向用户到PayPal。你可以找到更多信息关于如何集成Website Payments Pro，通过访问 <a href="http://django-paypal.readthedocs.org/en/v0.2.5/pro/index.html" class="uri">http://django-paypal.readthedocs.org/en/v0.2.5/pro/index.html</a>。</p>
<h2 id="导出订单为csv文件">导出订单为CSV文件</h2>
<p>有时候，你可能想要导出包含在模型的信息到一个文件中，这样你可以导入它到其他的系统中。其中一个范围最广的格式用来导出/导入数据就是<strong>Comma-Separated Values(CSV)</strong>。一个CSV文件就是一个纯文本文件包含若干记录。There is usually one record per line, and some delimiter character, usually a literal comma, separates the record fields<strong>（译者注：求翻译。。。）</strong> 。我们将要定制管理平台站点能够导出订单为CSV文件。</p>
<h2 id="添加定制操作到管理平台站点中">添加定制操作到管理平台站点中</h2>
<p>Django提供你多种不同的选项来定制管理平台站点。我们将要修改对象列视图（view）来包含一个定制的管理操作。</p>
<h2 id="一个管理操作工作如下一个用户选择对象从管理对象列页面通过复选框之后选择一个操作去执行在所有被选择的项上然后执行该操作以下">一个管理操作工作如下：一个用户选择对象从管理对象列页面通过复选框，之后选择一个操作去执行在所有被选择的项上，然后执行该操作。以下</h2>
<p>展示操作会位于管理页面的哪个地方：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-9.png" alt="django-8-9"></p>
<blockquote>
<p>创建定制管理操作允许管理人员一次性应用操作多个元素。</p>
</blockquote>
<p>你可以创建一个定制操作通过编写一个经常性的函数获取以下参数：</p>
<ul>
<li>当前展示的<em>ModelAdmin</em></li>
<li>当前请求对象，一个<em>HttpRequest</em>实例</li>
<li>一个查询集（QuerySet）给用户所选择的对象</li>
</ul>
<p>这个函数将会被执行当这个操作被触发在管理平台站点上。</p>
<p>我们将要创建一个定制管理操作来下载订单列表的CSV文件。编辑<em>orders</em>应用的<em>admin.py</em>文件，添加如下代码在<code>OrderAdmin</code>类之前：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">import</span></span> csv
<span class="im"><span class="hljs-keyword">import</span></span> datetime
<span class="im"><span class="hljs-keyword">from</span></span> django.http <span class="im"><span class="hljs-keyword">import</span></span> HttpResponse
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">export_to_csv</span><span class="hljs-params">(modeladmin, request, queryset)</span>:</span>

    opts <span class="op">=</span> modeladmin.model._meta
    response <span class="op">=</span> HttpResponse(content_type<span class="op">=</span><span class="st"><span class="hljs-string">'text/csv'</span></span>)
    response[<span class="st"><span class="hljs-string">'Content-Disposition'</span></span>] <span class="op">=</span> <span class="st"><span class="hljs-string">'attachment; \</span></span><span class="hljs-string">
</span><span class="st"><span class="hljs-string">           filename={}.csv'</span></span>.<span class="bu">format</span>(opts.verbose_name)
    writer <span class="op">=</span> csv.writer(response)
    fields <span class="op">=</span> [field <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> opts.get_fields() <span class="cf"><span class="hljs-keyword">if</span></span> <span class="op"><span class="hljs-keyword">not</span></span> field.many_to_many <span class="op"><span class="hljs-keyword">and</span></span> <span class="op"><span class="hljs-keyword">not</span></span> field.one_to_many]
    <span class="co"><span class="hljs-comment"># Write a first row with header information</span></span>
    writer.writerow([field.verbose_name <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> fields])
    <span class="co"><span class="hljs-comment"># Write data rows</span></span>
    <span class="cf"><span class="hljs-keyword">for</span></span> obj <span class="op"><span class="hljs-keyword">in</span></span> queryset:
        data_row <span class="op">=</span> []
        <span class="cf"><span class="hljs-keyword">for</span></span> field <span class="op"><span class="hljs-keyword">in</span></span> fields:
            value <span class="op">=</span> <span class="bu">getattr</span>(obj, field.name)
            <span class="cf"><span class="hljs-keyword">if</span></span> <span class="bu">isinstance</span>(value, datetime.datetime):
                value <span class="op">=</span> value.strftime(<span class="st"><span class="hljs-string">'</span></span><span class="sc"><span class="hljs-string">%d</span></span><span class="st"><span class="hljs-string">/%m/%Y'</span></span>)
            data_row.append(value)
        writer.writerow(data_row)
    <span class="cf"><span class="hljs-keyword">return</span></span> response
export_to_csv.short_description <span class="op">=</span> <span class="st"><span class="hljs-string">'Export to CSV'</span></span></code></pre></div>
<p>在这代码中，我们执行以下任务：</p>
<ul>
<li>1.我们创建一个<code>HttpResponse</code>实例包含一个定制<code>text/csv</code>内容类型来告诉浏览器这个响应需要处理为一个CSV文件。我们还添加一个<code>Content-Disposition</code>头来指示这个HTTP响应包含一个附件。</li>
<li>2.我们创建一个CSV <code>writer</code>对象，该对象将会被写入<code>response</code>对象。</li>
<li>3.我们动态的获取<code>model</code>字段通过使用模型（moedl）<code>_meta</code>选项的<code>get_fields()</code>方法。我们排除多对多以及一对多的关系。</li>
<li>4.我们编写了一个头行包含字段名。</li>
<li>5.我们迭代给予的查询集（QuerySet）并且为每一个查询集中返回的对象写入行。我们注意格式化<code>datetime</code>对象因为这个输出值给CSV必须是一个字符串。</li>
<li>6.我们定制这个操作的显示名在模板（template）中通过设置一个<code>short_description</code>属性给这个函数。</li>
</ul>
<p>我们已经创建了一个普通的管理操作可以添加到任意的<em>ModelAdmin</em>类。</p>
<p>最后，添加新的<code>export_to_csv</code>管理操作给<code>OrderAdmin</code>类如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    <span class="co"><span class="hljs-comment"># ...</span></span>
    actions <span class="op">=</span> [export_to_csv]</code></pre></div>
<p>打开 <a href="http://127.0.0.1:8000/admin/orders/order/" class="uri">http://127.0.0.1:8000/admin/orders/order/</a> 在你的浏览器中。管理操作看上去如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-10.png" alt="django-8-10"></p>
<p>选择一些订单然后选择<strong>Export to CSV</strong>操作从下拉选框中，之后点击<strong>Go</strong>按钮。你的浏览器会下载生成的CSV文件名为<em>order.csv</em>。打开下载的文件使用一个文本编辑器。你会看到的内容如以下的格式，包含一个头行以及你之前选择的每行订单对象：</p>
<pre><code class="hljs groovy">ID,first name,last name,email,address,postal
code,city,created,updated,paid
<span class="hljs-number">3</span>,Antonio,Melé,antonio.mele<span class="hljs-meta">@gmail</span>.com,Bank Street <span class="hljs-number">33</span>,WS J11,London,<span class="hljs-number">25</span><span class="hljs-regexp">/05/</span><span class="hljs-number">2015</span>,<span class="hljs-number">25</span><span class="hljs-regexp">/05/</span><span class="hljs-number">2015</span>,False
...</code></pre>
<p>如你所见，创建管理操作是非常简单的。</p>
<h2 id="扩展管理站点通过定制视图view">扩展管理站点通过定制视图（view）</h2>
<p>有时候你可能想要定制管理平台站点，比如处理<em>ModelAdmin</em>的配置，管理操作的创建，以及覆盖管理模板（templates）。在这样的场景中，你需要创建一个定制的管理视图（view）。通过一个定制的管理视图（view），你可以构建任何你需要的功能。你只需要确保只有管理用户能访问你的视图并且你维护这个管理的外观和感觉通过你的模板（template）扩展自一个管理模板（template）。</p>
<p>让我们创建一个定制视图（view）来展示关于一个订单的信息。编辑<em>orders</em>应用下的<em>views.py</em>文件，添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.contrib.admin.views.decorators <span class="im"><span class="hljs-keyword">import</span></span> staff_member_required
<span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> .models <span class="im"><span class="hljs-keyword">import</span></span> Order

<span class="at"><span class="hljs-meta">@staff_member_required</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">admin_order_detail</span><span class="hljs-params">(request, order_id)</span>:</span>
    order <span class="op">=</span> get_object_or_404(Order, <span class="bu">id</span><span class="op">=</span>order_id)
    <span class="cf"><span class="hljs-keyword">return</span></span> render(request,
                  <span class="st"><span class="hljs-string">'admin/orders/order/detail.html'</span></span>,
                  {<span class="st"><span class="hljs-string">'order'</span></span>: order})</code></pre></div>
<p>这个<code>staff_member_required</code>装饰器检查用户请求这个页面的<code>is_active</code>以及<code>is_staff</code>字段是被设置为<code>True</code>。在这个视图（view）中，我们获取<code>Order</code>对象通过给予的id以及渲染一个模板来展示这个订单。</p>
<p>现在，编辑<em>orders</em>应用中的<em>urls.py</em>文件并且添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^admin/order/(?P&lt;order_id&gt;\d+)/$'</span></span>,
    views.admin_order_detail,
    name<span class="op">=</span><span class="st"><span class="hljs-string">'admin_order_detail'</span></span>),</code></pre></div>
<p>创建以下文件结构在<em>orders</em>应用的<em>templates/</em>目录下：</p>
<pre class="shell"><code class="hljs apache"><span class="hljs-attribute">admin</span>/
    <span class="hljs-attribute">orders</span>/
        <span class="hljs-attribute"><span class="hljs-nomarkup">order</span></span>/
            <span class="hljs-attribute">detail</span>.html</code></pre>
<p>编辑<em>detail.html</em>模板（template），添加以下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span>{</span><span>%</span> extends "admin/base_site.html" <span>%</span><span>}</span>
<span>{</span><span>%</span> load static <span>%</span><span>}</span>

<span>{</span><span>%</span> block extrastyle <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">link</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">rel</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"stylesheet"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">type</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"text/css"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> static "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">css</span>/<span class="hljs-attr">admin.css</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="hljs-tag"> </span><span class="kw"><span class="hljs-tag">/&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block title <span>%</span><span>}</span>
     Order <span>{</span><span>{</span> order.id <span>}</span><span>}</span> <span>{</span><span>{</span> block.super <span>}</span><span>}</span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block breadcrumbs <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"breadcrumbs"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">admin:index"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Home<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span> <span class="dv">&amp;rsaquo;</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">admin:orders_order_changelist"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Orders<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="dv">&amp;rsaquo;</span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> url "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">admin:orders_order_change"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">order.id</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Order <span>{</span><span>{</span> order.id <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="dv">&amp;rsaquo;</span> Detail
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span>

<span>{</span><span>%</span> block content <span>%</span><span>}</span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>Order <span>{</span><span>{</span> order.id <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">ul</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"object-tools"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">li</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"#"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">onclick</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"window.print();"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Print order<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">li</span>&gt;</span></span> 
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">ul</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">table</span>&gt;</span></span> 
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Created<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> order.created <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Customer<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> order.first_name <span>}</span><span>}</span> <span>{</span><span>{</span> order.last_name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span> 
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>E-mail<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span><span class="hljs-tag">&lt;<span class="hljs-name">a</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">href</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"mailto:<span>{</span><span>{</span> order.email <span>}</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> order.email <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">a</span>&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Address<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> order.address <span>}</span><span>}</span>, <span>{</span><span>{</span> order.postal_code <span>}</span><span>}</span> <span>{</span><span>{</span> order.city
<span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span> 
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Total amount<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span>$<span>{</span><span>{</span> order.get_total_cost <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Status<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>%</span> if order.paid <span>%</span><span>}</span>Paid<span>{</span><span>%</span> else <span>%</span><span>}</span>Pending payment<span>{</span><span>%</span> endif <span>%</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span> 
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">table</span>&gt;</span></span>
  
  <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"module"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">div</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"tabular inline-related last-related"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">table</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h2</span>&gt;</span></span>Items bought<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h2</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">thead</span>&gt;</span></span>
          <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Product<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Price<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Quantity<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
          <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">thead</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tbody</span>&gt;</span></span>
          <span>{</span><span>%</span> for item in order.items.all <span>%</span><span>}</span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"row<span>{</span><span>%</span> cycle "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">1"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">2</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
              <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> item.product.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
              <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
              <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> item.quantity <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
              <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.get_cost <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
          <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
          <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"total"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"3"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
            <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> order.get_total_cost <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
          <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
        <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tbody</span>&gt;</span></span>
      <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">table</span>&gt;</span></span>
    <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
  <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">div</span>&gt;</span></span>
<span>{</span><span>%</span> endblock <span>%</span><span>}</span></code></pre></div>
<p>这个模板（template）是用来显示一个订单详情在管理平台站点中。这个模板（template）扩展Djnago的管理平台站点的<em>admin/base_site.html</em>模板，它包含管理的主要HTML结构和CSS样式。我们加载定制的静态文件<em>css/admin.css</em>。</p>
<p>为了使用静态文件，你需要拿到它们从这章教程的实例代码中。复制位于<em>orders</em>应用的<em>static/</em>目录下的静态文件然后添加它们到你的项目的相同位置。</p>
<p>我们使用定义在父模板（template）的区块包含我们自己的内容。我们展示信息关于订单和购买的商品。</p>
<p>当你想要扩展一个管理模板（template），你需要知道它的结构以及确定存在的区块。你可以找到所有管理模板（template），通过访问 <a href="https://github.com/django/django/tree/1.8.6/django/contrib/admin/templates/admin" class="uri">https://github.com/django/django/tree/1.8.6/django/contrib/admin/templates/admin</a> 。</p>
<p>你也可以重写一个管理模板（template）如果你需要的话。为了重写一个管理模板（template），拷贝它到你的<em>template</em>目录保持相同的相对路径以及文件名。Django管理平台站点将会使用你的定制模板（template）替代默认的模板。</p>
<p>最后，让我们添加一个链接给每个<em>Order</em>对象在管理平台站点的列展示页面。编辑<em>orders</em>应用的<em>admin.py</em>文件然后添加以下代码，在<code>OrderAdmin</code>类上面：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.core.urlresolvers <span class="im"><span class="hljs-keyword">import</span></span> reverse
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">order_detail</span><span class="hljs-params">(obj)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'&lt;a href="{}"&gt;View&lt;/a&gt;'</span></span>.<span class="bu">format</span>(
        reverse(<span class="st"><span class="hljs-string">'orders:admin_order_detail'</span></span>, args<span class="op">=</span>[obj.<span class="bu">id</span>]))
order_detail.allow_tags <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span></code></pre></div>
<p>这个函数需要一个<em>Order</em>对象作为参数并且返回一个HTML链接给<code>admind_order_detail</code> URL。Django会避开默认的HTML输出。我们必须设置<code>allow_tags</code>属性为<code>True</code>来避开auto-escaping。</p>
<blockquote>
<p>设置<code>allow_tags</code>属性为<code>True</code>来避免HTML-escaping在一些<em>Model</em>方法，<em>ModelAdmin</em>方法，以及任何其他的调用中。当你使用<code>allow_tags</code>的时候，能确保避开用户输入的跨域脚本。</p>
</blockquote>
<p>之后，编辑<code>OrderAdmin</code>类来展示链接：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'id'</span></span>,
                    <span class="co"><span class="hljs-string">'first_name'</span></span>, 
                    <span class="co"><span class="hljs-comment"># ... </span></span>
                    <span class="co"><span class="hljs-string">'updated'</span></span>, 
                    order_detail]</code></pre></div>
<p>打开 <a href="http://127.0.0.1:8000/admin/orders/order/" class="uri">http://127.0.0.1:8000/admin/orders/order/</a> 在你的浏览器中。每一行现在都会包含一个<strong>View</strong>链接如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-11.png" alt="django-8-11"></p>
<p>点击某个订单的<strong>View</strong>链接来加载定制订单详情页面。你会看到一个页面如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-12.png" alt="django-8-12"></p>
<h2 id="生成动态的pdf发票">生成动态的PDF发票</h2>
<p>如今我们已经有了一个完整的结账和支付系统，我们可以生成一张PDF发票给每个订单。有几个Python库可以生成PDF文件。一个最流行的生成PDF的Python库是Reportlab。你可以找到关于如何使用Reportlab输出PDF文件的信息，通过访问 <a href="https://docs.djangoproject.com/en/1.8/howto/outputting-pdf/" class="uri">https://docs.djangoproject.com/en/1.8/howto/outputting-pdf/</a> 。</p>
<p>在大部分的场景中，你还需要添加定制样式和格式给你的PDF文件。你会发现渲染一个HTML模板（template）以及转化该模板（template）为一个PDF文件更加的方便，保持Python远离表现层。我们要遵循这个方法并且使用一个模块来生成PDF文件通过Django。我们将要使用WeasyPrint，它是一个Python库可以生成PDF文件从HTML模板中。</p>
<h2 id="安装weasyprint">安装WeasyPrint</h2>
<p>首先，安装WeasyPrint的依赖给你的OS，这些依赖你可以找到通过访问 <a href="http://weasyprint.org/docs/install/#platforms" class="uri">http://weasyprint.org/docs/install/#platforms</a> 。</p>
<p>之后，安装WeasyPrint通过<em>pip</em>渠道使用如下命令：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> WeasyPrint==<span class="hljs-number">0.24</span></code></pre>
<h2 id="创建一个pdf模板template">创建一个PDF模板（template）</h2>
<p>我们需要一个HTML文档给WeasyPrint输入。我们将要创建一个HTML模板（template），渲染它使用Django，并且传递它给WeasyPrint来生成PDF文件。</p>
<p>创建一个新的模板（template）文件在<em>orders</em>应用的<em>templates/orders/order/目录下命名为</em>pdf.html*。添加如下内容：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml"><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">html</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">body</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h1</span>&gt;</span></span>My Shop<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h1</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
       Invoice no. <span>{</span><span>{</span> order.id <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">br</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"secondary"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
         <span>{</span><span>{</span> order.created|date:"M d, Y" <span>}</span><span>}</span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>

     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Bill to<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>
       <span>{</span><span>{</span> order.first_name <span>}</span><span>}</span> <span>{</span><span>{</span> order.last_name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span></span>
       <span>{</span><span>{</span> order.email <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span></span>
       <span>{</span><span>{</span> order.address <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">br</span>&gt;</span></span>
       <span>{</span><span>{</span> order.postal_code <span>}</span><span>}</span>, <span>{</span><span>{</span> order.city <span>}</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">h3</span>&gt;</span></span>Items bought<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">h3</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">table</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">thead</span>&gt;</span></span> 
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Product<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Price<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Quantity<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">th</span>&gt;</span></span>Cost<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">th</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">thead</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tbody</span>&gt;</span></span>
         <span>{</span><span>%</span> for item in order.items.all <span>%</span><span>}</span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"row<span>{</span><span>%</span> cycle "</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">1"</span></span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag">"<span class="hljs-attr">2</span>"</span></span><span class="hljs-tag"> </span><span class="er"><span class="hljs-tag"><span>%</span><span>}</span>"</span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span>&gt;</span></span><span>{</span><span>{</span> item.product.name <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.price <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span><span>{</span><span>{</span> item.quantity <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
             <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> item.get_cost <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
         <span>{</span><span>%</span> endfor <span>%</span><span>}</span>
         <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">tr</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"total"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">colspan</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"3"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>Total<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
           <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">td</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"num"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>$<span>{</span><span>{</span> order.get_total_cost <span>}</span><span>}</span><span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">td</span>&gt;</span></span>
         <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tr</span>&gt;</span></span>
       <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">tbody</span>&gt;</span></span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">table</span>&gt;</span></span>
     
     <span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">span</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">class</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">"<span>{</span><span>%</span> if order.paid <span>%</span><span>}</span>paid<span>{</span><span>%</span> else <span>%</span><span>}</span>pending<span>{</span><span>%</span> endif <span>%</span><span>}</span>"</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span></span>
       <span>{</span><span>%</span> if order.paid <span>%</span><span>}</span>Paid<span>{</span><span>%</span> else <span>%</span><span>}</span>Pending payment<span>{</span><span>%</span> endif <span>%</span><span>}</span>
     <span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">span</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">body</span>&gt;</span></span>
<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">html</span>&gt;</span></span></code></pre></div>
<p>这个模板（template）就是PDF发票。在这个模板（template）中，我们展示所有订单详情以及一个HTML <code>&lt;table&gt;</code> 元素包含所有商品。我们还包含了一条消息来展示如果该订单已经支付或者支付还在进行中。</p>
<h2 id="渲染pdf文件">渲染PDF文件</h2>
<p>我们将要创建一个视图（view）来生成PDF发票给存在的订单通过使用管理平台站点。编辑<em>order</em>应用的<em>views.py</em>文件添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">from</span></span> django.http <span class="im"><span class="hljs-keyword">import</span></span> HttpResponse
<span class="im"><span class="hljs-keyword">from</span></span> django.template.loader <span class="im"><span class="hljs-keyword">import</span></span> render_to_string
<span class="im"><span class="hljs-keyword">import</span></span> weasyprint

<span class="at"><span class="hljs-meta">@staff_member_required</span></span>
<span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">admin_order_pdf</span><span class="hljs-params">(request, order_id)</span>:</span>
    order <span class="op">=</span> get_object_or_404(Order, <span class="bu">id</span><span class="op">=</span>order_id)
    html <span class="op">=</span> render_to_string(<span class="st"><span class="hljs-string">'orders/order/pdf.html'</span></span>,
                            {<span class="st"><span class="hljs-string">'order'</span></span>: order})
    response <span class="op">=</span> HttpResponse(content_type<span class="op">=</span><span class="st"><span class="hljs-string">'application/pdf'</span></span>)
    response[<span class="st"><span class="hljs-string">'Content-Disposition'</span></span>] <span class="op">=</span> <span class="st"><span class="hljs-string">'filename=\</span></span><span class="hljs-string">
</span><span class="st"><span class="hljs-string">           "order_{}.pdf"'</span></span>.<span class="bu">format</span>(order.<span class="bu">id</span>)
    weasyprint.HTML(string<span class="op">=</span>html).write_pdf(response,
        stylesheets<span class="op">=</span>[weasyprint.CSS(
            settings.STATIC_ROOT <span class="op">+</span> <span class="st"><span class="hljs-string">'css/pdf.css'</span></span>)])
    <span class="cf"><span class="hljs-keyword">return</span></span> response</code></pre></div>
<p>这个视图（view）用来生成一个PDF发票给一个订单。我们使用<code>staff_member_required</code>装饰器来确保只有管理人员能够访问这个视图（view）。我们获取<em>Order</em>对象通过给予的ID并且我们使用<code>rander_to_string()</code>函数提供自Django来渲染<em>orders/order/pdf.html</em>。这个渲染过的HTML会被保存到<code>html</code>变量中。之后，我们生成一个新的<code>HttpResponse</code>对象指定<code>application/pdf</code>的内容类型并且包含<code>Content-Disposition</code>头来指定这个文件名。我们使用WeasyPrint来生成一个PDF文件从渲染的HTML代码中并且将该文件写入<code>HttpResponse</code>对象中。我们加载它从本地路径通过使用<code>STATIC_ROOT</code>设置。最后，我们返回这个生成的响应。</p>
<p>由于我们需要使用<code>STATIC_ROOT</code>设置，我们需要添加它到我们的项目中。这个项目将会是静态文件的所在地。编辑<em>myshop</em>项目的<em>settings.py</em>文件，添加如下设置：</p>
<pre><code class="hljs ini"><span class="hljs-attr">STATIC_ROOT</span> = os.path.join(BASE_DIR, <span class="hljs-string">'static/'</span>)</code></pre>
<p>之后，运行命令<code>python manage.py collectstatic</code>。你会在输出末尾看到如下输出：</p>
<pre class="shell"><code class="hljs fsharp">You have requested <span class="hljs-keyword">to</span> collect <span class="hljs-keyword">static</span> files at the destination
location <span class="hljs-keyword">as</span> specified <span class="hljs-keyword">in</span> your settings:
       
    code/myshop/<span class="hljs-keyword">static</span>
This will overwrite existing files!
Are you sure you want <span class="hljs-keyword">to</span> <span class="hljs-keyword">do</span> this?</code></pre>
<p>输入<em>yes</em>然后回车。你会得到一条消息，告知那个静态文件已经复制到<code>STATIC_ROOT</code>目录中。</p>
<p><em>collectstatic</em>命令复制所有静态文件从你的应用到定义在<code>STATIC_ROOT</code>设置的目录中。这允许每个应用去提供它自己的静态文件通过使用一个<em>static/</em>目录来包含它们。你还可以提供额外的静态文件来源在<code>STATICFILES_DIRS</code>设置。所有的目录被指定在<code>STATICFILED_DIRS</code>列中的都将会被复制到<code>STATIC_ROOT</code>目录中当<em>collectstatic</em>被执行的时候。</p>
<p>编辑<em>orders</em>应用目录下的<em>urls.py</em>文件并且添加如下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^admin/order/(?P&lt;order_id&gt;\d+)/pdf/$'</span></span>,
    views.admin_order_pdf,
    name<span class="op">=</span><span class="st"><span class="hljs-string">'admin_order_pdf'</span></span>),</code></pre></div>
<p>现在，我们可以编辑管理列展示页面给<em>Order</em>模型（model）来添加一个链接给PDF文件给每一个结果。编辑<em>orders</em>应用的<em>admin.py</em>文件并且添加以下代码在<code>OrderAdmin</code>类上面：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">order_pdf</span><span class="hljs-params">(obj)</span>:</span>
    <span class="cf"><span class="hljs-keyword">return</span></span> <span class="st"><span class="hljs-string">'&lt;a href="{}"&gt;PDF&lt;/a&gt;'</span></span>.<span class="bu">format</span>(
        reverse(<span class="st"><span class="hljs-string">'orders:admin_order_pdf'</span></span>, args<span class="op">=</span>[obj.<span class="bu">id</span>]))
order_pdf.allow_tags <span class="op">=</span> <span class="va"><span class="hljs-keyword">True</span></span>
order_pdf.short_description <span class="op">=</span> <span class="st"><span class="hljs-string">'PDF bill'</span></span></code></pre></div>
<p>添加<code>order_pdf</code>给<code>OrderAdmin</code>类的<code>list_display</code>属性：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">OrderAdmin</span><span class="hljs-params">(admin.ModelAdmin)</span>:</span>
    list_display <span class="op">=</span> [<span class="st"><span class="hljs-string">'id'</span></span>,
                    <span class="co"><span class="hljs-comment"># ... </span></span>
                    order_detail, 
                    order_pdf]</code></pre></div>
<p>如果你指定一个<code>short_description</code>属性给你的调用，Django将会使用它给这个列命名。</p>
<p>打开 <a href="http://127.0.0.1:8000/admin/orders/order/" class="uri">http://127.0.0.1:8000/admin/orders/order/</a> 在你的浏览器中。每一行现在都包含一个PDF链接，如下所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-13.png" alt="django-8-13"></p>
<p>点击某一个订单的<strong>PDF</strong>。你会看到一个生成的PDF文件，如下所示一个订单还没有支付完成：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-14.png" alt="django-8-14"></p>
<p>对于支付完成的订单，你会看到如下所示的PDF文件：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-8-15.png" alt="django-8-15"></p>
<h2 id="通过e-mail发送pdf文件">通过e-mail发送PDF文件</h2>
<p>让我们发送一封e-mail给我们的顾客包含生成的PDF发表但一个支付被接收的时候。编辑<em>payment</em>应用下的<em>signals.py</em>文件并且添加如下导入：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.template.loader <span class="im"><span class="hljs-keyword">import</span></span> render_to_string
<span class="im"><span class="hljs-keyword">from</span></span> django.core.mail <span class="im"><span class="hljs-keyword">import</span></span> EmailMessage
<span class="im"><span class="hljs-keyword">from</span></span> django.conf <span class="im"><span class="hljs-keyword">import</span></span> settings
<span class="im"><span class="hljs-keyword">import</span></span> weasyprint
<span class="im"><span class="hljs-keyword">from</span></span> io <span class="im"><span class="hljs-keyword">import</span></span> BytesIO</code></pre></div>
<p>之后添加如下代码在<code>order.save()</code>行之后，需要同样的缩进等级：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="co"><span class="hljs-comment"># create invoice e-mail</span></span>
subject <span class="op">=</span> <span class="st"><span class="hljs-string">'My Shop - Invoice no. {}'</span></span>.<span class="bu">format</span>(order.<span class="bu">id</span>)
message <span class="op">=</span> <span class="st"><span class="hljs-string">'Please, find attached the invoice for your recent</span></span><span class="hljs-string">
</span><span class="st"><span class="hljs-string">purchase.'</span></span>
email <span class="op">=</span> EmailMessage(subject,
                    message,
                    <span class="st"><span class="hljs-string">'admin@myshop.com'</span></span>,
                    [order.email])
<span class="co"><span class="hljs-comment"># generate PDF</span></span>
html <span class="op">=</span> render_to_string(<span class="st"><span class="hljs-string">'orders/order/pdf.html'</span></span>, {<span class="st"><span class="hljs-string">'order'</span></span>: order})
out <span class="op">=</span> BytesIO()
stylesheets<span class="op">=</span>[weasyprint.CSS(settings.STATIC_ROOT <span class="op">+</span> <span class="st"><span class="hljs-string">'css/pdf.css'</span></span>)]
weasyprint.HTML(string<span class="op">=</span>html).write_pdf(out,
                                        stylesheets<span class="op">=</span>stylesheets)
<span class="co"><span class="hljs-comment"># attach PDF file</span></span>
email.attach(<span class="st"><span class="hljs-string">'order_{}.pdf'</span></span>.<span class="bu">format</span>(order.<span class="bu">id</span>),
            out.getvalue(),
            <span class="co"><span class="hljs-string">'application/pdf'</span></span>)
<span class="co"><span class="hljs-comment"># send e-mail</span></span>
email.send()</code></pre></div>
<p>在这个信号中，我们使用Django提供的<code>EmailMessage</code>类来创建一个e-mail对象。之后我们渲染这个模板（template）到<code>html</code>变量中。我们生成PDF文件从渲染的模板（template）中，并且我们输出它到一个<code>BytesIO</code>实例中，该实例是一个内容字节缓存。之后我们附加这个生成的PDF文件到<code>EmailMessage</code>对象通过使用它的<code>attach()</code>方法，包含这个<code>out</code>缓存的内容。</p>
<p>记住设置你的SMTP设置在项目的<em>settings.py</em>文件中来发送e-mail。你可以到<strong>第二章 通过高级特性扩展你的blog</strong>去看下一个SMTP配置的例子。</p>
<p>现在你可以打开Ngrok提供给你的应用的URL然后完成一个新的支付处理为了收到PDF发票到你的e-mail中。</p>
<h2 id="总结">总结</h2>
<p>在这一章中，你集成了一个支付网关到你的项目中。你定制了Django管理平台页面并且学习到了如何动态的生成CSV以及PDF文件。</p>
<p>在下一章中将会给你一个深刻理解关于国际化和本地化给Django项目。你还会学习到创建一个赠券系统已经构建一个产品推荐引擎。</p>
</div>
