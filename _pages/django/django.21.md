---
layout: single
permalink: /django/example12/
title: "Django By Example 第十二章"
sidebar:
  nav: "django"
# toc: true
---

<div><p>书籍出处：<a href="https://www.packtpub.com/web-development/django-example" class="uri">https://www.packtpub.com/web-development/django-example</a><br>
原作者：Antonio Melé</p>
<h1 id="第十二章">第十二章</h1>
<h2 id="构建一个api">构建一个API</h2>
<p>在上一章中，你构建了一个学生注册系统和课程报名。你创建了用来展示课程内容的视图以及如何使用Django的缓存框架。在这章中，你将学习如何做到以下几点：</p>
<ul>
<li>构建一个 RESTful API</li>
<li>用API视图操作认证和权限</li>
<li>创建API视图放置和路由</li>
</ul>
<h2 id="构建一个restful-api">构建一个RESTful API</h2>
<p>你可能想要创建一个接口给其他的服务来与你的web应用交互。通过构建一个API，你可以允许第三方来消费信息以及程序化的操作你的应用程序。</p>
<p>你可以通过很多方法构成你的API，但是我们最鼓励你遵循REST原则。REST体系结构来自Representational State Transfer。RESTful API是基于资源的（resource-based）。你的模型代表资源和HTTP方法例如GET,POST,PUT,以及DELETE是被用来取回，创建，更新，以及删除对象的。HTTP响应代码也可以在上下文中使用。不同的HTTP响应代码的返回用来指示HTTP请求的结果，例如，2XX响应代码用来表示成功，4XX表示错误，等等。</p>
<p>在RESTful API中最通用的交换数据是JSON和XML。我们将要为我们的项目构建一个JSON序列化的REST API。我们的API会提供以下功能：</p>
<ul>
<li>获取科目</li>
<li>获取可用的课程</li>
<li>获取课程内容</li>
<li>课程报名</li>
</ul>
<p>我们可以通过创建定制视图从Django开始构建一个API。当然，有很多第三方的模块可以给你的项目简单的创建一个API，其中最有名的就是Django Rest Framework。</p>
<h2 id="安装django-rest-framework">安装Django Rest Framework</h2>
<p>Django Rest Framework允许你为你的项目方便的构建REST APIs。你可以通过访问 <a href="http://www.django-rest-framework.org/" class="uri">http://www.django-rest-framework.org</a> 找到所有REST Framework信息。</p>
<p>打开shell然后通过以下命令安装这个框架：</p>
<pre><code class="hljs cmake">pip <span class="hljs-keyword">install</span> djangorestframework=<span class="hljs-number">3.2</span>.<span class="hljs-number">3</span></code></pre>
<p>编辑<em>educa</em>项目的<em>settings.py</em>文件，在<em>INSTALLED_APPS</em>设置中添加<em>rest_framework</em>来激活这个应用，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">INSTALLED_APPS <span class="op">=</span> (
       <span class="co"><span class="hljs-comment"># ...</span></span>
       <span class="st"><span class="hljs-string">'rest_framework'</span></span>,
   )</code></pre></div>
<p>之后，添加如下代码到<em>settings.py</em>文件中：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">REST_FRAMEWORK <span class="op">=</span> {
       <span class="st"><span class="hljs-string">'DEFAULT_PERMISSION_CLASSES'</span></span>: [
<span class="st"><span class="hljs-string">'rest_framework.permissions.DjangoModelPermissionsOrAnonReadOnly'</span></span>
       ]
}</code></pre></div>
<p>你可以使用<em>REST_FRAMEWORK</em>设置为你的API提供一个指定的配置。REST Framework提供了一个广泛的设置去配置默认的行为。<em>DEFAULT_PERMISSION_CLASSES</em>配置指定了去读取，创建，更新或者删除对象的默认权限。我们设置<em>DjangoModelPermissionsOrAnonReadOnly</em>作为唯一的默认权限类。这个类依赖与Django的权限系统允许用户去创建，更新，或者删除对象，同时提供只读的访问给陌生人用户。你会在之后学习更多关于权限的方面。</p>
<p>如果要找到一个完整的REST框架可用设置列表，你可以访问 <a href="http://www.django-rest-framework.org/api-guide/settings/" class="uri">http://www.django-rest-framework.org/api-guide/settings/</a> 。</p>
<h2 id="定义序列化器">定义序列化器</h2>
<p>设置好REST Framework之后，我们需要指定我们的数据将会如何序列化。输出的数据必须被序列化成指定的格式，并且输出的数据将会给进程去序列化。REST框架提供了以下类来给单个对象去构建序列化：</p>
<ul>
<li>Serializer：给一般的Python类实例提供序列化。</li>
<li>ModelSerializer：给模型实例提供序列化。</li>
<li>HyperlinkedModelSerializer：类似与<em>ModelSerializer</em>，但是代表与链接而不是主键的对象关系。</li>
</ul>
<p>让我们构建我们的第一个序列化器。在<em>courses</em>应用目录下创建以下文件结构：</p>
<pre class="shell"><code class="hljs">api/
    __init__.py
    serializers.py</code></pre>
<p>我们将会在<em>api</em>目录中构建所有的API功能为了保持一切都有良好的组织。编辑<em>serializeers.py</em>文件，然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework <span class="im"><span class="hljs-keyword">import</span></span> serializers
<span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Subject

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">SubjectSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Subject
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>)</code></pre></div>
<p>以上是给<em>Subject</em>模型使用的序列化器。序列化器以一种类似的方式被定义给Django<br>
的<em>From</em>和<em>ModelForm</em>类。<em>Meta</em>类允许你去指定模型序列化以及给序列化包含的字段。所有的模型字段都会被包含如果你没有设置一个<em>fields</em>属性。</p>
<p>让我们尝试我们的序列化器。打开命令行通过`python manage.py shell*开始Django shell。运行以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> courses.models <span class="im"><span class="hljs-keyword">import</span></span> Subject
<span class="im"><span class="hljs-keyword">from</span></span> courses.api.serializers <span class="im"><span class="hljs-keyword">import</span></span> SubjectSerializer
subject <span class="op">=</span> Subject.objects.latest(<span class="st"><span class="hljs-string">'id'</span></span>)
serializer <span class="op">=</span> SubjectSerializer(subject)
serializer.data</code></pre></div>
<p>在上面的例子中，我们拿到了一个<em>Subject</em>对象，创建了一个<em>SubjectSerializer</em>的实例，并且访问序列化的数据。你会得到以下输出：</p>
<pre class="shell"><code class="hljs python">{<span class="hljs-string">'slug'</span>: <span class="hljs-string">'music'</span>, <span class="hljs-string">'id'</span>: <span class="hljs-number">4</span>, <span class="hljs-string">'title'</span>: <span class="hljs-string">'Music'</span>}</code></pre>
<p>如你所见，模型数据被转换成了Python的数据类型。</p>
<h2 id="了解解析器和渲染器">了解解析器和渲染器</h2>
<p>在你在一个HTTP响应中返回序列化数据之前，这个序列化数据必须使用指定的格式进行渲染。同样的，当你拿到一个HTTP请求，在你使用这个数据操作之前你必须解析传入的数据并且反序列化这个数据。REST Framework包含渲染器和解析器来执行以上操作。</p>
<p>让我们看下如何解析传入的数据。给予一个JSON字符串输入，你可以使用REST康佳提供的<em>JSONParser</em>类来转变它成为一个Python对象。在Python shell中执行以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> io <span class="im"><span class="hljs-keyword">import</span></span> BytesIO
<span class="im"><span class="hljs-keyword">from</span></span> rest_framework.parsers <span class="im"><span class="hljs-keyword">import</span></span> JSONParser
data <span class="op">=</span> <span class="hljs-string">b</span><span class="st"><span class="hljs-string">'{"id":4,"title":"Music","slug":"music"}'</span></span>
JSONParser().parse(BytesIO(data))</code></pre></div>
<p>你将会拿到以下输出：</p>
<pre><code class="hljs python">{<span class="hljs-string">'id'</span>: <span class="hljs-number">4</span>, <span class="hljs-string">'title'</span>: <span class="hljs-string">'Music'</span>, <span class="hljs-string">'slug'</span>: <span class="hljs-string">'music'</span>}</code></pre>
<p>REST Framework还包含<em>Renderer</em>类，该类允许你去格式化API响应。框架会查明通过的内容使用的是哪种渲染器。它对响应进行检查，根据请求的<em>Accept</em>头去预判内容的类型。除此以外，渲染器可以通过URL的格式后缀进行预判。举个例子，访问将会出发<em>JSONRenderer</em>为了返回一个JSON响应。</p>
<p>回到shell中，然后执行以下代码去从提供的序列化器例子中渲染<em>serializer</em>对象：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.renderers <span class="im"><span class="hljs-keyword">import</span></span> JSONRenderer
JSONRenderer().render(serializer.data)</code></pre></div>
<p>你会看到以下输出：</p>
<p>b'{"id":4,"title":"Music","slug":"music"}'</p>
<p>我们使用<em>JSONRenderer</em>去渲染序列化数据为JSON。默认的，REST Framework使用两种不同的渲染器：<em>JSONRenderer</em>和<em>BrowsableAPIRenderer</em>。后者提供一个web接口可以方便的浏览你的API。你可以通过<em>REST_FRAMEWORK</em>设置的<em>DEFAULT_RENDERER_CLASSES</em>选项改变默认的渲染器类。</p>
<p>你可以找到更多关于渲染器和解析器的信息通过访问 <a href="http://www.django-rest-framework.org/api-guide/renderers/" class="uri">http://www.django-rest-framework.org/api-guide/renderers/</a> 以及 <a href="http://www.django-rest-/" class="uri">http://www.django-rest-</a> framework.org/api-guide/parsers/ 。</p>
<h2 id="构建列表和详情视图">构建列表和详情视图</h2>
<p>REST Framework自带一组通用视图和mixins，你可以用来构建你自己的API。它们提供了获取，创建，更新以及删除模型对象的功能。你可以看到所有REST Framework提供的通用mixins和视图，通过访问 <a href="http://www.django-rest-framework.org/api-guide/generic-views/" class="uri">http://www.django-rest-framework.org/api-guide/generic-views/</a> 。</p>
<p>让我们创建列表和详情视图去取回<em>Subject</em>对象们。在<em>courses/api/</em>目录下创建一个新的文件并命名为<em>views.py</em>。添加如下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework <span class="im"><span class="hljs-keyword">import</span></span> generics
<span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Subject
<span class="im"><span class="hljs-keyword">from</span></span> .serializers <span class="im"><span class="hljs-keyword">import</span></span> SubjectSerializer

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">SubjectListView</span><span class="hljs-params">(generics.ListAPIView)</span>:</span>
    queryset <span class="op">=</span> Subject.objects.<span class="bu">all</span>()
    serializer_class <span class="op">=</span> SubjectSerializer

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">SubjectDetailView</span><span class="hljs-params">(generics.RetrieveAPIView)</span>:</span>
    queryset <span class="op">=</span> Subject.objects.<span class="bu">all</span>()
    serializer_class <span class="op">=</span> SubjectSerializer</code></pre></div>
<p>在这串代码中，我们使用REST Framework提供的<em>ListAPIView</em>和<em>RetrieveAPIView</em>视图。我们给给予的关键值包含了一个<em>pk</em> URL参数给详情视图去取回对象。两个视图都有以下属性：</p>
<ul>
<li>queryset：基础查询集用来取回对象。</li>
<li>serializer_class：这个类用来序列化对象。</li>
</ul>
<p>让我们给我们的视图添加URL模式。在<em>courses/api/</em>目录下创建新的文件并命名为<em>urls.py</em>并使之看上去如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

urlpatterns <span class="op">=</span> [
       url(<span class="vs"><span class="hljs-string">r'^subjects/$'</span></span>,
           views.SubjectListView.as_view(),
           name<span class="op">=</span><span class="st"><span class="hljs-string">'subject_list'</span></span>),
       url(<span class="vs"><span class="hljs-string">r'^subjects/(?P&lt;pk&gt;\d+)/$'</span></span>,
           views.SubjectDetailView.as_view(),
           name<span class="op">=</span><span class="st"><span class="hljs-string">'subject_detail'</span></span>),
]</code></pre></div>
<p>编辑<em>educa</em>项目的主<em>urls.py</em>文件并且包含以下API模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># ...</span></span>
    url(<span class="vs"><span class="hljs-string">r'^api/'</span></span>, include(<span class="st"><span class="hljs-string">'courses.api.urls'</span></span>, namespace<span class="op">=</span><span class="st"><span class="hljs-string">'api'</span></span>)),
]</code></pre></div>
<p>我们给我们的API URLs使用<em>api</em>命名空间。确保你的服务器已经通过命令<code>python manange.py runserver</code>启动。打开shell然后通过<em>cURL</em>获取URL <a href="http://127.0.0.1:8000/api/subjects/" class="uri">http://127.0.0.1:8000/api/subjects/</a> 如下所示：</p>
<pre><code class="hljs ruby">$ curl <span class="hljs-symbol">http:</span>/<span class="hljs-regexp">/127.0.0.1:8000/api</span><span class="hljs-regexp">/subjects/</span></code></pre>
<p>你会获取类似以下的响应：</p>
<pre class="shell"><code class="hljs json">[{"<span class="hljs-attr">id</span>":<span class="hljs-number">2</span>,"<span class="hljs-attr">title</span>":<span class="hljs-string">"Mathematics"</span>,"<span class="hljs-attr">slug</span>":<span class="hljs-string">"mathematics"</span>},{"<span class="hljs-attr">id</span>":<span class="hljs-number">4</span>,"<span class="hljs-attr">title</span>":<span class="hljs-string">"Music"</span>,"<span class="hljs-attr">slug</span>":<span class="hljs-string">"music"</span>},{"<span class="hljs-attr">id</span>":<span class="hljs-number">3</span>,"<span class="hljs-attr">title</span>":<span class="hljs-string">"Physics"</span>,"<span class="hljs-attr">slug</span>":<span class="hljs-string">"physics"</span>},{"<span class="hljs-attr">id</span>":<span class="hljs-number">1</span>,"<span class="hljs-attr">title</span>":<span class="hljs-string">"Programming"</span>,"<span class="hljs-attr">slug</span>":<span class="hljs-string">"programming"</span>}]</code></pre>
<p>这个HTTP响应包含JSON格式的一个<em>Subject</em>对象列。如果你的操作系统没有安装过<em>cURL</em>，你可还可以使用其他的工具去发送定制HTTP请求例如一个浏览器扩展 Postman ，这个扩展你可以在 <a href="https://www.getpostman.com/" class="uri">https://www.getpostman.com</a> 找到。</p>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/api/subjects/" class="uri">http://127.0.0.1:8000/api/subjects/</a> 。你会看到如下所示的REST Framework的可浏览API：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-12-1.png" alt="django-12-1"></p>
<p>这个HTML界面由<em>BrowsableAPIRenderer</em>渲染器提供。它展示了结果头和内容并且允许执行请求。你还可以在URL包含一个<em>Subject</em>对象的id来访问该对象的API详情视图。在你的浏览器中打开 <a href="http://127.0.0.1:8000/api/subjects/1/" class="uri">http://127.0.0.1:8000/api/subjects/1/</a> 。你将会看到一个单独的渲染成JSON格式的<em>Subject</em>对象。</p>
<h2 id="创建嵌套的序列化">创建嵌套的序列化</h2>
<p>我们将要给<em>Course</em>模型创建一个序列化。编辑<em>api/serializers.py</em>文件并添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Course
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'voerview'</span></span>,
                  <span class="co"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'owner'</span></span>, <span class="st"><span class="hljs-string">'modules'</span></span>)</code></pre></div>
<p>让我们看下一个<em>Course</em>对象是如何被序列化的。打开shell，运行<code>python manage.py shell</code>，然后运行以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.renderers <span class="im"><span class="hljs-keyword">import</span></span> JSONRenderer
<span class="im"><span class="hljs-keyword">from</span></span> courses.models <span class="im"><span class="hljs-keyword">import</span></span> Course
<span class="im"><span class="hljs-keyword">from</span></span> courses.api.serializers <span class="im"><span class="hljs-keyword">import</span></span> CourseSerializer
course <span class="op">=</span> Course.objects.latest(<span class="st"><span class="hljs-string">'id'</span></span>)
serializer <span class="op">=</span> CourseSerializer(course)
JSONRenderer().render(serializer.data)</code></pre></div>
<p>你将会通过我们包含在<em>CourseSerializer</em>中的字段获取到一个JSON对象。你可以看到<em>modules</em>管理器的被关联对象呗序列化成一列关键值，如下所示：</p>
<pre><code class="hljs prolog"><span class="hljs-string">"modules"</span>: [<span class="hljs-number">17</span>, <span class="hljs-number">18</span>, <span class="hljs-number">19</span>, <span class="hljs-number">20</span>, <span class="hljs-number">21</span>, <span class="hljs-number">22</span>]</code></pre>
<p>我们想要包含个多的信息关于每一个模块，所以我们需要序列化<em>Module</em>对象以及嵌套它们。修改<em>api/serializers.py</em>文件提供的代码，使之看上去如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework <span class="im"><span class="hljs-keyword">import</span></span> serializers
<span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Course, Module
   
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ModuleSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Module
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'order'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'description'</span></span>)

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
    modules <span class="op">=</span> ModuleSerializer(many<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>, read_only<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Course
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>, <span class="st"><span class="hljs-string">'overview'</span></span>,
                    <span class="co"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'owner'</span></span>, <span class="st"><span class="hljs-string">'modules'</span></span>)</code></pre></div>
<p>我们给<em>Module</em>模型定义了一个<em>ModuleSerializer</em>去提供序列化。之后我们添加一个<em>modules</em>属性给<em>CourseSerializer</em>去嵌套<em>ModuleSerializer</em>序列化器。我们设置<em>many=True</em>去表明我们正在序列化多个对象。<em>read_only</em>参数表明这个字段是只读的并且不可以被包含在任何输入中去创建或者升级对象。</p>
<p>打开shell并且再次创建一个<em>CourseSerializer</em>的实例。使用<em>JSONRenderer</em>渲染序列化器的<em>data</em>属性。这一次，被排列的模块会被通过嵌套的<em>ModuleSerializer</em>序列化器给序列化，如下所示：</p>
<pre class="shell"><code class="hljs smalltalk"><span class="hljs-comment">"modules"</span>: [
        {
           <span class="hljs-comment">"order"</span>: <span class="hljs-number">0</span>,
           <span class="hljs-comment">"title"</span>: <span class="hljs-comment">"Django overview"</span>,
           <span class="hljs-comment">"description"</span>: <span class="hljs-comment">"A brief overview about the Web Framework."</span>
        }, 
        {
            <span class="hljs-comment">"order"</span>: <span class="hljs-number">1</span>,
            <span class="hljs-comment">"title"</span>: <span class="hljs-comment">"Installing Django"</span>,
            <span class="hljs-comment">"description"</span>: <span class="hljs-comment">"How to install Django."</span>
        },
        ... 
]</code></pre>
<p>你可以找到更多关于序列化器的内容，通过访问 <a href="http://www.django-rest-framework.org/api-guide/serializers/" class="uri">http://www.django-rest-framework.org/api-guide/serializers/</a>。</p>
<h2 id="构建定制视图">构建定制视图</h2>
<p>REST Framework提供一个<em>APIView</em>类，这个类基于Django的<em>View</em>类构建API功能。<em>APIView</em>类与<em>View</em>在使用REST Framework的定制<em>Request</em>以及<em>Response</em>对象时不同，并且操作<em>APIException</em>例外的返回合适的HTTP响应。它还有一个内建的验证和认证系统去管理视图的访问。</p>
<p>我们将要创建一个视图给用户去对课程进行报名。编辑<em>api/views.py</em>文件并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.shortcuts <span class="im"><span class="hljs-keyword">import</span></span> get_object_or_404
<span class="im"><span class="hljs-keyword">from</span></span> rest_framework.views <span class="im"><span class="hljs-keyword">import</span></span> APIView
<span class="im"><span class="hljs-keyword">from</span></span> rest_framework.response <span class="im"><span class="hljs-keyword">import</span></span> Response
<span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Course

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseEnrollView</span><span class="hljs-params">(APIView)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">post</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, pk, </span></span><span class="bu"><span class="hljs-function"><span class="hljs-params">format</span></span></span><span class="op"><span class="hljs-function"><span class="hljs-params">=</span></span></span><span class="va"><span class="hljs-function"><span class="hljs-params">None</span></span></span><span class="hljs-function"><span class="hljs-params">)</span>:</span>
        course <span class="op">=</span> get_object_or_404(Course, pk<span class="op">=</span>pk)
        course.students.add(request.user)
        <span class="cf"><span class="hljs-keyword">return</span></span> Response({<span class="st"><span class="hljs-string">'enrolled'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>})</code></pre></div>
<p><em>CourseEnrollView</em>视图操纵用户对课程进行报名。以上代码解释如下：</p>
<ul>
<li>我们创建了一个定制视图，是<em>APIView</em>的子类。</li>
<li>我们给POST操作定义了一个<em>post()</em>方法。其他的HTTP方法都不允许放这个这个视图。</li>
<li>我们预计一个<em>pk</em>URL参数会博涵一个课程的ID。我们通过给予的<em>pk</em>参数获取这个课程，并且如果这个不存在的话就抛出一个404异常。</li>
<li>我们添加当前用户给<em>Course</em>对象的<em>students</em>多对多关系并放回一个成功响应。</li>
</ul>
<p>编辑<em>api/urls.py</em>文件并且给<em>CourseEnrollView</em>视图添加以下URL模式：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^courses/(?P&lt;pk&gt;\d+)/enroll/$'</span></span>,
       views.CourseEnrollView.as_view(),
       name<span class="op">=</span><span class="st"><span class="hljs-string">'course_enroll'</span></span>),</code></pre></div>
<p>理论上，我们现在可以执行一个POST请求去给当前用户对一个课程进行报名。但是，我们需要辨认这个用户并且阻止为认证的用户来访问这个视图。让我们看下API认证和权限是如何工作的。</p>
<h2 id="操纵认证">操纵认证</h2>
<p>REST Framework提供认证类去辨别用户执行的请求。如果认证成功，这个框架会在<em>request.user</em>中设置认证的<em>User</em>对象。如果没有用户被认证，一个Django的<em>AnonymousUser</em>实例会被代替。</p>
<p>REST Framework提供以下认证后台：</p>
<ul>
<li>BasicAuthentication：HTTP基础认证。用户和密码会被编译为Base64并被客户端设置在<em>Authorization</em> HTTP头中。你可以学习更多关于它的内容，通过访问 <a href="https://en.wikipedia.org/wiki/Basic_access_authentication" class="uri">https://en.wikipedia.org/wiki/Basic_access_authentication</a> 。</li>
<li>TokenAuthentication：基于token的认证。一个<em>Token</em>模型被用来存储用户tokens。用来认证的<em>Authorization</em> HTTP头里面拥有包含token的用户。</li>
<li>SessionAuthentication：使用Djnago的会话后台（session backend）来认证。这个后台从你的网站前端来执行认证AJAX请求给API是非常有用的。</li>
</ul>
<p>你可以创建一个通过继承REST Framework提供的<em>BaseAuthentication</em>类的子类以及重写<em>authenticate()</em>方法来构建一个定制的认证后台。</p>
<p>你可以在每个视图的基础上设置认证，或者通过<em>DEFAULT_AUTHENTICATION_CLASSES</em>设置为全局认证。</p>
<blockquote>
<p>认证只能失败用户正在执行的请求。它无法允许或者组织视图的访问。你必须使用权限去限制视图的访问。</p>
</blockquote>
<p>你可以找到关于认证的所有信息，通过访问 <a href="http://www.django-rest-/" class="uri">http://www.django-rest-</a> framework.org/api-guide/authentication/ 。</p>
<p>让我们给我们的视图添加<em>BasicAuthentication</em>。编辑<em>courses</em>应用的<em>api/views.py</em>文件，然后给<em>CourseEnrollView</em>添加一个<em>authentication_classes</em>属性，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.authentication <span class="im"><span class="hljs-keyword">import</span></span> BasicAuthentication

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseEnrollView</span><span class="hljs-params">(APIView)</span>:</span>
    authentication_classes <span class="op">=</span> (BasicAuthentication,)
    <span class="co"><span class="hljs-comment"># ...</span></span></code></pre></div>
<p>用户将会被设置在HTTP请求中的<em>Authorization</em>头里面的证书进行识别。</p>
<h2 id="给视图添加权限">给视图添加权限</h2>
<p>REST Framework包含一个权限系统去限制视图的访问。一些REST Framework的内置权限如下所示：</p>
<ul>
<li>AllowAny：无限制的访问，无论当前用户是否通过认证。</li>
<li>IsAuthenticated：只允许通过认证的用户。</li>
<li>IsAuthenticatedOrReadOnly：通过认证的用户拥有完整的权限。陌生用户只允许去还行可读的方法，例如GET, HEAD或者OPETIONS。</li>
<li>DjangoModelPermissions：权限与<em>django.contrib.auth</em>进行了捆绑。视图需要一个<em>queryset</em>属性。只有分配了模型权限的并经过认证的用户才能获得权限。</li>
<li>DjangoObjectPermissions：基于每个对象基础上的Django权限。</li>
</ul>
<p>如果用户没有权限，他们通常会获得以下某个HTTP错误：</p>
<ul>
<li>HTTP 401：无认证。</li>
<li>HTTP 403：没有权限。</li>
</ul>
<p>你可以获得更多的关于权限的信息，通过访问 <a href="http://www.django-rest-/" class="uri">http://www.django-rest-</a> framework.org/api-guide/permissions/ 。</p>
<p>编辑<em>courses</em>应用的<em>api/views.py</em>文件然后给<em>CourseEnrollView</em>添加一个<em>permission_classes</em>属性，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.authentication <span class="im"><span class="hljs-keyword">import</span></span> BasicAuthentication
<span class="im"><span class="hljs-keyword">from</span></span> rest_framework.permissions <span class="im"><span class="hljs-keyword">import</span></span> IsAuthenticated

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseEnrollView</span><span class="hljs-params">(APIView)</span>:</span>
    authentication_classes <span class="op">=</span> (BasicAuthentication,)
    permission_classes <span class="op">=</span> (IsAuthenticated,)
    <span class="co"><span class="hljs-comment"># ...</span></span></code></pre></div>
<p>我们包含了<em>IsAuthenticated</em>权限。这个权限将会组织陌生用户访问这个视图。现在，我们可以之sing一个POST请求给我们的新的API方法。</p>
<p>确保开发服务器正在运行。打开shell然后运行以下命令：</p>
<pre><code class="hljs groovy">curl -i –X POST <span class="hljs-string">http:</span><span class="hljs-comment">//127.0.0.1:8000/api/courses/1/enroll/</span></code></pre>
<p>你将会得到以下响应：</p>
<pre class="shell"><code class="hljs r">HTTP/<span class="hljs-number">1.0</span> <span class="hljs-number">401</span> UNAUTHORIZED
<span class="hljs-keyword">...</span>
{<span class="hljs-string">"detail"</span>: <span class="hljs-string">"Authentication credentials were not provided."</span>}</code></pre>
<p>如我们所预料的，我们得到了一个401 HTTP code，因为我们没有认证过。让我们带上我们的一个用户进行下基础认证。运行以下命令：</p>
<pre class="shell"><code class="hljs groovy">curl -i -X POST -u <span class="hljs-string">student:</span>password <span class="hljs-string">http:</span><span class="hljs-comment">//127.0.0.1:8000/api/courses/1/enroll/</span></code></pre>
<p>使用一个已经存在的用户的证书替换<em>student:password</em>。你会得到以下响应：</p>
<pre class="shell"><code class="hljs cpp">HTTP/<span class="hljs-number">1.0</span> <span class="hljs-number">200</span> OK
...
{<span class="hljs-string">"enrolled"</span>: <span class="hljs-literal">true</span>}</code></pre>
<p>你可以额访问管理站点然后检查到上面命令中的用户已经完成了课程的报名。</p>
<h3 id="创建视图设置和路由">创建视图设置和路由</h3>
<p><em>ViewSets</em>允许你去定义你的API的交互并且让REST Framework通过一个<em>Router</em>对象动态的构建URLs。通过使用视图设置，你可以避免给多个视图重复编写相同的逻辑。视图设置包含典型的创建，获取，更新，删除选项操作，它们是 <em>list()</em>,<em>create()</em>,<em>retrieve()</em>,<em>update()</em>,<em>partial_update()</em>以及<em>destroy()</em>。</p>
<p>让我们给<em>Course</em>模型创建一个视图设置。编辑<em>api/views.py</em>文件然后添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework <span class="im"><span class="hljs-keyword">import</span></span> viewsets
<span class="im"><span class="hljs-keyword">from</span></span> .serializers <span class="im"><span class="hljs-keyword">import</span></span> CourseSerializer

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseViewSet</span><span class="hljs-params">(viewsets.ReadOnlyModelViewSet)</span>:</span>
    queryset <span class="op">=</span> Course.objects.<span class="bu">all</span>()
    serializer_class <span class="op">=</span> CourseSerializer</code></pre></div>
<p>我们创建了一个继承<em>ReadOnlyModelViewSet</em>类的子类，被继承的类提供了只读的操作 <em>list()</em>和<em>retrieve()</em>，前者用来排列对象，后者用来取回一个单独的对象。编辑<em>api/urls.py</em>文件并且给我们的视图设置创建一个路由，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> django.conf.urls <span class="im"><span class="hljs-keyword">import</span></span> url, include
<span class="im"><span class="hljs-keyword">from</span></span> rest_framework <span class="im"><span class="hljs-keyword">import</span></span> routers
<span class="im"><span class="hljs-keyword">from</span></span> . <span class="im"><span class="hljs-keyword">import</span></span> views

router <span class="op">=</span> routers.DefaultRouter()
router.register(<span class="st"><span class="hljs-string">'courses'</span></span>, views.CourseViewSet)

urlpatterns <span class="op">=</span> [
    <span class="co"><span class="hljs-comment"># ...</span></span>
    url(<span class="vs"><span class="hljs-string">r'^'</span></span>, include(router.urls)),
]</code></pre></div>
<p>我们创建两个一个<em>DefaultRouter</em>对象并且通过<em>courses</em>前缀注册了我们的视图设置。这个路由负责给我们的视图动态的生成URLs。</p>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/api/" class="uri">http://127.0.0.1:8000/api/</a> 。你会看到路由排列除了所有的视图设置在它的基础URL中，如下图所示：</p>
<p><img src="http://ohqrvqrlb.bkt.clouddn.com/django-12-2.png" alt="django-12-2"></p>
<p>你可以访问 <a href="http://127.0.0.1:8000/api/courses/" class="uri">http://127.0.0.1:8000/api/courses/</a> 去获取课程的列表。</p>
<p>你可以学习到跟多关于视图设置的内容，通过访问 <a href="http://www.django-rest-framework.org/api-guide/viewsets/" class="uri">http://www.django-rest-framework.org/api-guide/viewsets/</a> 。你也可以找到更多关于路由的信息，通过访问 <a href="http://www.django-rest-framework.org/api-guide/routers/" class="uri">http://www.django-rest-framework.org/api-guide/routers/</a> 。</p>
<h3 id="给视图设置添加额外的操作">给视图设置添加额外的操作</h3>
<p>你可以给视图设置添加额外的操作。让我们修改我们之前的<em>CourseEnrollView</em>视图成为一个定制的视图设置操作。编辑<em>api/views.py</em>文件然后修改<em>CourseViewSet</em>类如下所示：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.decorators <span class="im"><span class="hljs-keyword">import</span></span> detail_route

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseViewSet</span><span class="hljs-params">(viewsets.ReadOnlyModelViewSet)</span>:</span>
    queryset <span class="op">=</span> Course.objects.<span class="bu">all</span>()
    serializer_class <span class="op">=</span> CourseSerializer
<span class="hljs-meta">    </span><span class="at"><span class="hljs-meta">@detail_route</span></span><span class="hljs-meta">(methods</span><span class="op"><span class="hljs-meta">=</span></span><span class="hljs-meta">[</span><span class="st"><span class="hljs-meta">'post'</span></span><span class="hljs-meta">],</span>
                authentication_classes<span class="op">=</span>[BasicAuthentication],
                permission_classes<span class="op">=</span>[IsAuthenticated])
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">enroll</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
        course <span class="op">=</span> <span class="va">self</span>.get_object()
        course.students.add(request.user)
        <span class="cf"><span class="hljs-keyword">return</span></span> Response({<span class="st"><span class="hljs-string">'enrolled'</span></span>: <span class="va"><span class="hljs-keyword">True</span></span>})</code></pre></div>
<p>我们添加了一个定制<em>enroll()</em>方法相当于给这个视图设置的一个额外的操作。以上的代码解释如下：</p>
<ul>
<li>我们使用框架的<em>detail_route</em>装饰器去指定这个类是一个在一个单独对象上被执行的操作。</li>
<li>这个装饰器允许我们给这个操作添加定制属性。我们指定这个视图只允许POST方法，并且设置了认证和权限类。</li>
<li>我们使用<em>self.get_object()</em>去获取<em>Course</em>对象。</li>
<li>我们给<em>students</em>添加当前用户的多对多关系并且返回一个定制的成功响应。</li>
</ul>
<p>编辑<em>api/urls.py</em>文件并移除以下URL，因为我们不再需要它们：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">url(<span class="vs"><span class="hljs-string">r'^courses/(?P&lt;pk&gt;[\d]+)/enroll/$'</span></span>,
       views.CourseEnrollView.as_view(),
       name<span class="op">=</span><span class="st"><span class="hljs-string">'course_enroll'</span></span>),</code></pre></div>
<p>之后编辑<em>api/views.py</em>文件并且移除<em>CourseEnrollView</em>类。</p>
<p>这个用来在课程中报名的URL现在已经是由路由动态的生成。这个URL保持不变，因为它使用我们的操作名<em>enroll</em>动态的进行构建。</p>
<h3 id="创建定制权限">创建定制权限</h3>
<p>我们想要学生可以访问他们报名过的课程的内容。只有在这个课程中报名过的学生才能访问这个课程的内容。最好的方法就是通过一个定制的权限类。Django提供了一个<em>BasePermission</em>类允许你去定制以下功能：</p>
<ul>
<li>has_permission()：视图级的权限检查。</li>
<li>has_object_permission()：实例级的权限检查。</li>
</ul>
<p>以上方法会返回<em>True</em>来允许访问，相反就会返回<em>False</em>。在<em>courses/api/</em>中创建一个新的文件并命名为<em>permissions.py</em>。添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> rest_framework.permissions <span class="im"><span class="hljs-keyword">import</span></span> BasePermission

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">IsEnrolled</span><span class="hljs-params">(BasePermission)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">has_object_permission</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, view, obj)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> obj.students.<span class="bu">filter</span>(<span class="bu">id</span><span class="op">=</span>request.user.<span class="bu">id</span>).exists()</code></pre></div>
<p>我们创建了一个继承<em>BasePermission</em>类的子类，并且重写了<em>has_object_permission()</em>。我们检查执行请求的用户是否存在<em>Course</em>对象的<em>students</em>关系。我们下一步将要使用<em>IsEnrolled</em>权限。</p>
<h3 id="序列化课程内容">序列化课程内容</h3>
<p>我们需要序列化课程内容。<em>Content</em>模型包含一个通用的外键允许我们去关联不同的内容模型对象。然而，我们给上一章中给所有的内容模型添加了一个公用的<em>render()</em>方法。我们可以使用这个方法去提供渲染过的内容给我们的API。</p>
<p>编辑<em>courses</em>应用的<em>api/serializers.py</em>文件并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs"><span class="im"><span class="hljs-keyword">from</span></span> ..models <span class="im"><span class="hljs-keyword">import</span></span> Content

<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ItemRelatedField</span><span class="hljs-params">(serializers.RelatedField)</span>:</span>
    <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">to_representation</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, value)</span>:</span>
        <span class="cf"><span class="hljs-keyword">return</span></span> value.render()
        
<span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ContentSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
    item <span class="op">=</span> ItemRelatedField(read_only<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
    <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
        model <span class="op">=</span> Content
        fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'order'</span></span>, <span class="st"><span class="hljs-string">'item'</span></span>)</code></pre></div>
<p>在以上代码中，我们通过子类化REST Framework提供的<em>RealtedField</em>序列化器字段定义了一个定制字段并且重写了<em>to_representation()</em>方法。我们给<em>Content</em>模型定义了<em>ContentSerializer</em>序列化器并且使用定制字段给<em>item</em>生成外键。</p>
<p>我们需要一个替代序列化器给<em>Module</em>模型来包含它的内容，以及一个扩展的<em>Course</em>序列化器。编辑<em>api/serializers.py</em>文件并且添加以下代码：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">   <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">ModuleWithContentsSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
       contents <span class="op">=</span> ContentSerializer(many<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
       <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
           model <span class="op">=</span> Module
           fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'order'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'description'</span></span>, <span class="st"><span class="hljs-string">'contents'</span></span>)
           
   <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseWithContentsSerializer</span><span class="hljs-params">(serializers.ModelSerializer)</span>:</span>
       modules <span class="op">=</span> ModuleWithContentsSerializer(many<span class="op">=</span><span class="va"><span class="hljs-keyword">True</span></span>)
       <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">Meta</span>:</span>
           model <span class="op">=</span> Course
           fields <span class="op">=</span> (<span class="st"><span class="hljs-string">'id'</span></span>, <span class="st"><span class="hljs-string">'subject'</span></span>, <span class="st"><span class="hljs-string">'title'</span></span>, <span class="st"><span class="hljs-string">'slug'</span></span>,
                     <span class="co"><span class="hljs-string">'overview'</span></span>, <span class="st"><span class="hljs-string">'created'</span></span>, <span class="st"><span class="hljs-string">'owner'</span></span>, <span class="st"><span class="hljs-string">'modules'</span></span>)           </code></pre></div>
<p>让我们创建一个视图来模仿<em>retrieve()</em>操作的行为但是包含课程内容。编辑<em>api/views.py</em>文件添加以下方法给<em>CourseViewSet</em>类：</p>
<div class="sourceCode"><pre class="sourceCode python"><code class="sourceCode python hljs">   <span class="im"><span class="hljs-keyword">from</span></span> .permissions <span class="im"><span class="hljs-keyword">import</span></span> IsEnrolled
   <span class="im"><span class="hljs-keyword">from</span></span> .serializers <span class="im"><span class="hljs-keyword">import</span></span> CourseWithContentsSerializer
   
   <span class="kw"><span class="hljs-class"><span class="hljs-keyword">class</span></span></span><span class="hljs-class"> <span class="hljs-title">CourseViewSet</span><span class="hljs-params">(viewsets.ReadOnlyModelViewSet)</span>:</span>
       <span class="co"><span class="hljs-comment"># ...</span></span>
<span class="hljs-meta">       </span><span class="at"><span class="hljs-meta">@detail_route</span></span><span class="hljs-meta">(methods</span><span class="op"><span class="hljs-meta">=</span></span><span class="hljs-meta">[</span><span class="st"><span class="hljs-meta">'get'</span></span><span class="hljs-meta">],</span>
                     serializer_class<span class="op">=</span>CourseWithContentsSerializer,
                     authentication_classes<span class="op">=</span>[BasicAuthentication],
                     permission_classes<span class="op">=</span>[IsAuthenticated,
                                         IsEnrolled])
       <span class="kw"><span class="hljs-function"><span class="hljs-keyword">def</span></span></span><span class="hljs-function"> <span class="hljs-title">contents</span><span class="hljs-params">(</span></span><span class="va"><span class="hljs-function"><span class="hljs-params">self</span></span></span><span class="hljs-function"><span class="hljs-params">, request, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">*</span></span></span><span class="hljs-function"><span class="hljs-params">args, </span></span><span class="op"><span class="hljs-function"><span class="hljs-params">**</span></span></span><span class="hljs-function"><span class="hljs-params">kwargs)</span>:</span>
           <span class="cf"><span class="hljs-keyword">return</span></span> <span class="va">self</span>.retrieve(request, <span class="op">*</span>args, <span class="op">**</span>kwargs)</code></pre></div>
<p>以上的方法解释如下：</p>
<ul>
<li>我们使用<em>detail_route</em>装饰器去指定这个操作是在一个单独的对象上进行执行。</li>
<li>我们指定只有GET方法允许访问这个操作。</li>
<li>我们使用新的<em>CourseWithContentsSerializer</em>序列化器类来包含渲染过的课程内容。</li>
<li>我们使用<em>IsAuthenticated</em>和我们的定制<em>IsEnrolled</em>权限。只要做到了这点，我们可以确保只有在这个课程中报名的用户才能访问这个课程的内容。</li>
<li>我们使用存在的<em>retrieve()</em>操作去返回课程对象。</li>
</ul>
<p>在你的浏览器中打开 <a href="http://127.0.0.1:8000/api/courses/1/contents/" class="uri">http://127.0.0.1:8000/api/courses/1/contents/</a> 。如果你使用正确的证书访问这个视图，你会看到这个课程的每一个模块都包含给渲染过的课程内容的HTML，如下所示：</p>
<div class="sourceCode"><pre class="sourceCode html"><code class="sourceCode html hljs xml">   {
   "order": 0,
   "title": "Installing Django",
   "description": "",
   "contents": [
        {
        "order": 0,
        "item": "<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">p</span>&gt;</span></span>Take a look at the following video for installing Django:<span class="kw"><span class="hljs-tag">&lt;/<span class="hljs-name">p</span>&gt;</span></span>\n"
        }, 
        {
        "order": 1,
        "item": "\n<span class="kw"><span class="hljs-tag">&lt;<span class="hljs-name">iframe</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">width</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">\</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">"480\"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">height</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">\</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">"360\"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">src</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">\</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">"http:</span>//<span class="hljs-attr">www.youtube.com</span>/<span class="hljs-attr">embed</span>/<span class="hljs-attr">bgV39DlmZ2U</span>?<span class="hljs-attr">wmode</span></span></span><span class="ot"><span class="hljs-tag">=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">opaque\</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">frameborder</span>=</span></span><span class="st"><span class="hljs-tag"><span class="hljs-string">\</span></span></span><span class="er"><span class="hljs-tag"><span class="hljs-string">"0\"</span></span></span><span class="ot"><span class="hljs-tag"> <span class="hljs-attr">allowfullscreen</span></span></span><span class="kw"><span class="hljs-tag">&gt;</span><span class="hljs-tag">&lt;/<span class="hljs-name">iframe</span>&gt;</span></span>\n\n"
        } 
    ]
    }   </code></pre></div>
<p>你已经构建了一个简单的API允许其他服务器来程序化的访问课程应用。REST Framework还允许你通过<em>ModelViewSet</em>视图设置去管理创建以及编辑对象。我们已经覆盖了Django Rest Framework的主要部分，但是你可以找到该框架更多的特性，通过查看它的文档，地址在 <a href="http://www.django-rest-framework.org/" class="uri">http://www.django-rest-framework.org/</a> 。</p>
<h3 id="总结">总结</h3>
<p>在这章中，你创建了一个RESTful API给其他的服务器去与你的web应用交互。</p>
<p>一个额外的章节<strong>第十三章，Going Live</strong>需要在线下载：<a href="https://www/" class="uri">https://www</a>. packtpub.com/sites/default/files/downloads/Django_By_Example_ GoingLive.pdf 。第十三章将会教你如何使用uWSGI以及NGINX去构建一个生产环境。你还将学习到如何去导入一个定制中间件以及创建定制管理命令。</p>
<p>你已经到达这本书的结尾。恭喜你！你已经学习到了通过Django构建一个成功的web应用所需的技能。这本书指导你通过其他的技术与Django集合去开发了几个现实生活能用到的项目。现在你已经准备好去创建你自己的Django项目了，无论是一个简单的样品还是一个一个强大的wen应用。</p>
<p>祝你下一次Django的冒险好运！</p>
</div>
