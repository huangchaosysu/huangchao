---
layout: single
author_profile: true
title: "Django处理http请求的流程解析"
date: 2018-06-14 7:45:52
tags:
  - Python
  - Django
categories:
  - Python文章
  - Django
toc: true
---

### 1、Django的架构
    
![](/assets/images/django/struct.png)
核心是middleware（中间件），django所有的请求、返回都由中间件来完成。

中间件，就是处理HTTP的request和response的，类似插件，比如有Request中间件、view中间件、response中间件、exception中间件等，Middleware都需要在 “project/settings.py” 中 MIDDLEWARE_CLASSES 的定义。大致的程序流程图如下所示：

![](/assets/images/django/http_flow.png)

首先，Middleware都需要在 “project/settings.py” 中 MIDDLEWARE_CLASSES 的定义， 一个HTTP请求，将被这里指定的中间件从头到尾处理一遍，暂且称这些需要挨个处理的中间件为处理链，如果链中某个处理器处理后没有返回response，就把请求传递给下一个处理器；如果链中某个处理器返回了response，直接跳出处理链由response中间件处理后返回给客户端，可以称之为短路处理。

Django 处理一个 Request 的过程是首先通过中间件，然后再通过默认的 URL 方式进行的。我们可以在 Middleware 这个地方把所有Request 拦截住，用我们自己的方式完成处理以后直接返回 Response。因此了解中间件的构成是非常必要的。

#### Initializer: \_\_init\_\_(self)

出于性能的考虑，每个已启用的中间件在每个服务器进程中只初始化一次。也就是说__init__()仅在服务进程启动的时候调用，而在针对单个 request 处理时并不执行。

对一个 middleware 而言，定义__init__()方法的通常原因是检查自身的必要性。如果__init__() 抛出异常 django.core.exceptions.MiddlewareNotUsed ，则 Django 将从 middleware 栈中移出该 middleware。

在中间件中定义__init__() 方法时，除了标准的 self 参数之外，不应定义任何其它参数。

这个方法的调用时机在 Django 接收到 request 之后，但仍未解析URL以确定应当运行的 view 之前。Django 向它传入相应的 HttpRequest 对象，以便在方法中修改。

#### Request预处理函数: process_request(self, request)

process_request() 应当返回 None 或 HttpResponse 对象。  
如果返回 None ，Django 将继续处理这个 request，执行后续的中间件， 然后调用相应的 view。

如果返回 HttpResponse 对象，Django 将不再执行任何其它的中间件(无视其种类)以及相应的view。 Django将立即返回该 HttpResponse。

#### View预处理函数: process_view(self, request, callback, callback_args, callback_kwargs)

这个方法的调用时机在 Django 执行完 request 预处理函数并确定待执行的 view 之后，但在 view 函数实际执行之前。

#### request：HttpRequest 对象。

callback：Django将调用的处理request的python函数. 这是实际的函数对象本身, 而不是字符串表述的函数名。  
args：将传入view的位置参数列表，但不包括 request 参数(它通常是传入view的第一个参数)。

#### kwargs：将传入view的关键字参数字典。

如同 process_request() , process_view() 应当返回 None 或 HttpResponse 对象。如果返回 None ， Django将继续处理这个 request ，执行后续的中间件， 然后调用相应的view。

如果返回 HttpResponse 对象，Django 将不再执行 任何 其它的中间件(不论种类)以及相应的view，Django将立即返回。

#### Response后处理函数: process_response(self, request, response)
这个方法的调用时机在 Django 执行 view 函数并生成 response 之后。

该处理器能修改 response 的内容；一个常见的用途是内容压缩，如 gzip 所请求的 HTML 页面。

这个方法的参数相当直观： request 是 request 对象，而 response 则是从 view 中返回的 response 对象。

process_response() 必须返回 HttpResponse 对象. 这个 response 对象可以是传入函数的那一个原始对象（通常已被修改），也可以是全新生成的。

#### Exception后处理函数: process_exception(self, request, exception)

这个方法只有在 request 处理过程中出了问题并且 view 函数抛出了一个未捕获的异常时才会被调用。这个钩子可以用来发送错误通知，将现场相关信息输出到日志文件，或者甚至尝试从错误中自动恢复。

这个函数的参数除了一贯的 request 对象之外，还包括view函数抛出的实际的异常对象 exception 。

process_exception() 应当返回 None 或 HttpResponse 对象。

如果返回 None ， Django将用框架内置的异常处理机制继续处理相应request。

如果返回 HttpResponse 对象，Django 将使用该response对象，而短路框架内置的异常处理机制。

### 2、Django HTTP请求的处理流程
Django 和其他 Web 框架的 HTTP 处理的流程大致相同，Django 处理一个 Request 的过程是首先通过中间件，然后再通过默认的 URL 方式进行的。我们可以在 Middleware 这个地方把所有 Request 拦截住，用我们自己的方式完成处理以后直接返回 Response。

#### 1. 加载配置
Django 的配置都在 “Project/settings.py” 中定义，可以是 Django 的配置，也可以是自定义的配置，并且都通过 django.conf.settings 访问，非常方便。

#### 2. 启动
最核心动作的是通过 django.core.management.commands.runfcgi 的 Command 来启动，它运行 django.core.servers.fastcgi 中的 runfastcgi ， runfastcgi 使用了 flup 的 WSGIServer 来启动 fastcgi 。而 WSGIServer 中携带了 django.core.handlers.wsgi 的 WSGIHandler 类的一个实例，通过 WSGIHandler 来处理由Web服务器（比如Apache，Lighttpd等）传过来的请求，此时才是真正进入 Django 的世界。

#### 3. 处理 Request
当有 HTTP 请求来时， WSGIHandler 就开始工作了，它从 BaseHandler 继承而来。 WSGIHandler 为每个请求创建一个 WSGIRequest 实例，而 WSGIRequest 是从 http.HttpRequest 继承而来。接下来就开始创建 Response 了。

#### 4. 创建Response
BaseHandler 的 get_response 方法就是根据 request 创建 response ， 而 具体生成 response 的动作就是执行 urls.py 中对应的view函数了，这也是 Django可以处理“友好URL”的关键步骤，每个这样的函数都要返回一个 Response 实例。此时一般的做法是通过 loader 加载 template 并生成页面内 容，其中重要的就是通过 ORM 技术从数据库中取出数据，并渲染到 Template 中，从而生成具体的页面了。

#### 5. 处理Response
Django 返回 Response 给 flup ， flup 就取出 Response 的内容返回给 Web 服务器，由后者返回给浏览器。

总之， Django 在 fastcgi 中主要做了两件事：处理 Request 和创建 Response ， 而它们对应的核心就是“urls分析”、“模板技术”和“ORM技术”。

![](/assets/images/django/http_flow1.png)

如图所示，一个 HTTP 请求，首先被转化成一个 HttpRequest 对象，然后该对象被传递给 Request 中间件处理，如果该中间件返回了Response，则直接传递给 Response 中间件做收尾处理。否则的话 Request 中间件将访问 URL 配置，确定哪个 view 来处理，在确定了哪个 view 要执行，但是还没有执行该 view 的时候，系统会把 request 传递给 View 中间件处理器进行处理，如果该中间件返回了Response，那么该 Response 直接被传递给 Response 中间件进行后续处理，否则将执行确定的 View 函数处理并返回 Response，在这个过程中如果引发了异常并抛出，会被 Exception 中间件处理器进行处理。

### 3、请求处理机制其一：进入Django前的准备
一个 Request 到达了！

首先发生的是一些和 Django 有关（前期准备）的其他事情，分别是：

如果是 Apache/mod_python 提供服务，request 由 mod_python 创建的 django.core.handlers.modpython.ModPythonHandler 实例传递给 Django。

如果是其他服务器，则必须兼容 WSGI，这样，服务器将创建一个django.core.handlers.wsgi.WsgiHandler 实例。

这两个类都继承自 django.core.handlers.base.BaseHandler，它包含对任何类型的 request 来说都需要的公共代码。

快准备处理器(Handler)  
当上面其中一个处理器实例化后，紧接着发生了一系列的事情：

这个处理器（handler）导入你的 Django 配置文件。

这个处理器导入 Django 的自定义异常类。

这个处理器调用它自己的 load_middleware 方法，加载所有列在 MIDDLEWARE_CLASSES 中的 middleware 类并且内省它们。

最后一条有点复杂，我们仔细瞧瞧。

一个 middleware 类可以渗入处理过程的四个阶段：request，view，response 和 exception。要做到这一点，只需要定义指定的、恰当的方 法：process_request，process_view， process_response 和 process_exception。middleware 可以定义其中任何一个或所有这些方法，这取决于你想要它提供什么样的功能。

当处理器内省 middleware 时，它查找上述名字的方法，并建立四个列表作为处理器的实例变量：

_request_middleware 是一个保存 process_request 方法的列表（在每一 种情况下，它们是真正的方法，可以直接调用），这些方法来自于任一个定义了它们的 middleware 类。

_view_middleware 是一个保存 process_view 方法的列表，这些方法来自于任一个定义了它们的 middleware 类。

_response_middleware 是一个保存 process_response 方法的列表，这些方法来自于任一个定义了它们的 middleware 类。

_exception_middleware 是一个保存 process_exception 方法的列表，这些方法来自于任一个定义了它们的 middleware 类。

HttpRequest 准备好了就可以进入 Django

现在处理器已经准备好真正开始处理了，因此它给调度程序发送一个信号 request_started（Django 内部的调度程序允许各种不同的组件声明它们正在干 什么，并可以写一些代码监听特定的事件。关于这一点目前还没有官方的文档， 但在 wiki 上有一些注释。）。接下来它实例化一个 django.http.HttpRequest 的子类。

根据不同的处理器，可能是 django.core.handlers.modpython.ModPythonRequest 的一个实例，也可能是 django.core.handlers.wsgi.WSGIRequest 的一个实例。需要两个不同的类是因 为 mod_python 和 WSGI APIs 以不同的格式传入 request 信息，这个信息需要 解析为 Django 能够处理的一个单独的标准格式。

一旦一个 HttpRequest 或者类似的东西存在了，处理器就调用它自己的 get_response 方法，传入这个 HttpRequest 作为唯一的参数。这里就是几乎所有真正的活动发生的地方。

#### 4、请求处理机制其二：Django中间件的解析

Middleware 开始工作了

get_response 做的第一件事就是遍历处理器的 _request_middleware 实例变量并调用其中的每一个方法，传入 HttpRequest 的实例作为参数。
````
for middleware_method in self._request_middleware:
    response = middleware_method(request)
    if response:
        break
````
这些方法可以选择短路剩下的处理并立即让 get_response 返回，通过返回自身的一个值（如果它们这样做，返回值必须是 django.http.HttpResponse 的一个实例，后面会讨 论到）。如果其中之一这样做了，我们会立即回到主处理器代码，get_response 不会等着看其它 middleware 类想要做什么，它直接返回，然后处理器进入 response 阶段。

然而，更一般的情况是，这里应用的 middleware 方法简单地做一些处理并决定是否增加，删除或补充 request 的属性。

URL resolver 的解析

假设没有一个作用于 request 的 middleware 直接返回 response，处理器下一 步会尝试解析请求的 URL。它在配置文件中寻找一个叫做 ROOT_URLCONF 的配 置，用这个配置加上根 URL /，作为参数来创建 django.core.urlresolvers.RegexURLResolver 的一个实例，然后调用它的 resolve 方法来解析请求的 URL 路径。

URL resolver 遵循一个相当简单的模式。对于在 URL 配置文件中根据 ROOT_URLCONF 的配置产生的每一个在 urlpatterns 列表中的条目，它会检查请 求的 URL 路径是否与这个条目的正则表达式相匹配，如果是的话，有两种选择：

如果这个条目有一个可以调用的 include，resolver 截取匹配的 URL，转到 include 指定的 URL 配置文件并开始遍历其中 urlpatterns 列表中的 每一个条目。根据你 URL 的深度和模块性，这可能重复好几次。

否则，resolver 返回三个条目：

匹配的条目指定的 view function；

一个 从 URL 得到的未命名匹配组（被用来作为 view 的位置参数）；

一个关键 字参数字典，它由从 URL 得到的任意命名匹配组和从 URLConf 中得到的任 意其它关键字参数组合而成。

注意这一过程会在匹配到第一个指定了 view 的条目时停止，因此最好让你的 URL 配置从复杂的正则过渡到简单的正则，这样能确保 resolver 不会首先匹配 到简单的那一个而返回错误的 view function。

如果没有找到匹配的条目，resolver 会产生 django.core.urlresolvers.Resolver404 异常，它是 django.http.Http404 例外的子类。后面我们会知道它是如何处理的。
```
# Apply view middleware
for middleware_method in self._view_middleware:
    response = middleware_method(request, callback, allback_args, callback_kwargs)
    if response:
        break
```
一旦知道了所需的 view function 和相关的参数，处理器就会查看它的 _view_middleware 列表，并调用其中的方法，传入 HttpRequst，view function，针对这个 view 的位置参数列表和关键字参数字典。

还有，Middleware 有可能介入这一阶段并强迫处理器立即返回。

### 5、 请求处理机制其三：view层与模板解析
进入 View 了

如果处理过程这时候还在继续的话，处理器会调用 view function。Django 中的 Views 不很严格因为它只需要满足几个条件：

* 必须可以被调用。
* 必须接受 django.http.HttpRequest 的实例作为第一位值参数。
* 必须能产生一个异常或返回 django.http.HttpResponse 的一个实例。

除了这些，你就可以天马行空了。尽管如此，一般来说，views 会使用 Django 的 database API 来创建，检索，更新和删除数据库的某些东西，还会加载并渲染一个模板来呈现一些东西给最终用户。

额……模板

Django 的模板系统有两个部分：一部分是给设计师使用的混入少量其它东西的 HTML，另一部分是给程序员使用纯 Python。

从一个 HTML 作者的角度，Django 的模板系统非常简单，需要知道的仅有三个结构：

变量引用。在模板中是这样： {{ foo }}。

模板过滤。在上面的例子中使用过滤竖线是这样：{{ foo|bar }}。通常这 用来格式化输出（比如：运行 Textile，格式化日期等等）。

模板标签。是这样：{% baz %}。这是模板的“逻辑”实现的地方，你可以 {% if foo %}，{% for bar in foo %}，等等，if 和 for 都是模板标签。

变量引用以一种非常简单的方式工作。如果你只是要打印变量，只要 {{ foo }}，模板系统就会输出它。这里唯一的复杂情况是 {{ foo.bar }}，这时模板系 统按顺序尝试几件事：

首先它尝试一个字典方式的查找，看看 foo['bar'] 是否存在。如果存在， 则它的值被输出，这个过程也随之结束。

如果字典查找失败，模板系统尝试属性查找，看看 foo.bar 是否存在。同 时它还检查这个属性是否可以被调用，如果可以，调用之。

如果属性查找失败，模板系统尝试把它作为列表索引进行查找。

如果所有这些都失败了，模板系统输出配置 TEMPLATE_STRING_IF_INVALID 的值，默认是空字符串。

模板过滤就是简单的 Python functions，它接受一个值和一个参数，返回一个新的值。比如，date 过滤用一个 Python datetime 对象作为它的值，一个标准的 strftime 格式化字符串作为它的参数，返回对 datetime 对象应用了格式化字符 串之后的结果。

模板标签用在事情有一点点复杂的地方，它是你了解 Django 的模板系统是如何真正工作的地方。

Django 模板的结构

在内部，一个 Django 模板体现为一个 “nodes” 集合，它们都是从基本的 django.template.Node 类继承而来。Nodes 可以做各种处理，但有一个共同点： 每一个 Node 必须有一个叫做 render 的方法，它接受的第二个参数（第一个参 数，显然是 Node 实例）是 django.template.Context 的一个实例，这是一个类似于字典的对象，包含所有模板可以获得的变量。Node 的 render 方法必须返回 一个字符串，但如果 Node 的工作不是输出（比如，它是要通过增加，删除或修 改传入的 Context 实例变量中的变量来修改模板上下文），可以返回空字符串。

Django 包含许多 Node 的子类来提供有用的功能。比如，每个内置的模板标签都 被一个 Node 的子类处理（比如，IfNode 实现了 if 标签，ForNode 实现了 for 标签，等等）。所有内置标签可以在 django.template.defaulttags 找到。

![](/assets/images/django/http_flow2.png)

实际上，上面介绍的所有模板结构都是某种形式的 Nodes，纯文本也不异常。变 量查找由 VariableNode 处理，出于自然，过滤也应用在 VariableNode 上，标 签是各种类型的 Nodes，纯文本是一个 TextNode。

一般来说，一个 view 渲染一个模板要经过下面的步骤，依次是：

加载需要渲染的模板。这是由 django.template.loader.get_template 完成的，它能利用这许多方法中的任意一个来定位需要的模板文件。 get_template 函数返回一个 django.template.Template 实例，其中包含经过解析的模板和用到的方法。

实例化一个 Context 用来渲染模板。如果用的是 Context 的子类 django.template.RequestContext，那么附带的上下文处理函数就会自动添加在 view 中没有定义的变量。Context 的构建器方法用一个键/值对的字 典（对于模板，它将变为名/值变量）作为它唯一的参数，RequestContext 则用 HttpRequest 的一个实例和一个字典。

调用 Template 实例的 render 方法，Context 对象作为第一个位置参数。

Template 的 render 方法的返回值是一个字符串，它由 Template 中所有 Nodes 的 render 方法返回的值连接而成，调用顺序为它们出现在 Template 中的顺序。

关于 Response，一点点

一旦一个模板完成渲染，或者产生了其它某些合适的输出，view 就会负责产生一 个 django.http.HttpResponse 实例，它的构建器接受两个可选的参数：

一个作为 response 主体的字符串（它应该是第一位置参数，或者是关键字 参数 content）。大部分时间，这将作为渲染一个模板的输出，但不是必须 这样，在这里你可以传入任何有效的 Python 字符串。

作为 response 的 Content-Type header 的值（它应该是第二位置参数， 或者是关键字参数 mine_type）。如果没有提供这个参数，Django 将会使 用配置中 DEFAULT_MIME_TYPE 的值和 DEFAULT_CHARSET 的值，如果你没有 在 Django 的全局配置文件中更改它们的话，分别是 “text/html” 和 “utf-8”。

异常

如果 view 函数，或者其中的什么东西，发生了异常，那么 get_response（我知 道我们已经花了些时间深入 views 和 templates，但是一旦 view 返回或产生异常，我们仍将重拾处理器中间的 get_response 方法）将遍历它的 _exception_middleware 实例变量并调用那里的每个方法，传入 HttpResponse 和这个 exception 作为参数。如果顺利，这些方法中的一个会实例化一个 HttpResponse 并返回它。

这时候有可能还是没有得到一个 HttpResponse，这可能有几个原因：

* view 可能没有返回值。
* view 可能产生了异常但没有一个 middleware 能处理它。
* 一个 middleware 方法试图处理一个异常时自己又产生了一个新的异常。

这时候，get_response 会回到自己的异常处理机制中，它们有几个层次：

如果 exception 是 Http404 并且 DEBUG 设置为 True，get_response 将 执行 view django.views.debug.technical_404_response，传入 HttpRequest 和 exception 作为参数。这个 view 会展示 URL resolver 试图匹配的模式信息。

如果 DEBUG 是 False 并且异常是 Http404，get_response 会调用 URL resolver 的 resolve_404 方法。这个方法查看 URL 配置以判断哪一个 view 被指定用来处理 404 错误。默认是 django.views.defaults.page_not_found，但可以在 URL 配置中给 handler404 变量赋值来更改。

对于任何其它类型的异常，如果 DEBUG 设置为 True，get_response 将执 行 view django.views.debug.technical_500_response，传入 HttpRequest 和 exception 作为参数。这个 view 提供了关于异常的详细 信息，包括 traceback，每一个层次 stack 中的本地变量，HttpRequest 对象的详细描述和所有无效配置的列表。

如果 DEBUG 是 False，get_response 会调用 URL resolver 的 resolve_500 方法，它和 resolve_404 方法非常相似，这时默认的 view 是 django.views.defaults.server_error，但可以在 URL 配置中给 handler500 变量赋值来更改。

此外，对于除了 django.http.Http404 或 Python 内置的 SystemExit 之外的任 何异常，处理器会给调度者发送信号 got_request_exception，在返回之前，构建一个关于异常的描述，把它发送给列在 Django 配置文件的 ADMINS 配置中的每一个人。

现在，无论 get_response 在哪一个层次上发生错误，它都会返回一个 HttpResponse 实例，因此我们回到处理器的主要部分。一旦它获得一个 HttpResponse 它做的第一件事就是遍历它的 _response_middleware 实例变量并 应用那里的方法，传入 HttpRequest 和 HttpResponse 作为参数。
```
finally:
# Reset URLconf for this thread on the way out for complete
# isolation of request.urlconf
urlresolvers.set_urlconf(None)

try:
# Apply response middleware, regardless of the response
for middleware_method in self._response_middleware:
    response = middleware_method(request, response)
    response = self.apply_response_fixes(request, response)
```
注意对于任何想改变点什么的 middleware 来说，这是它们的最后机会。

一旦 middleware 完成了最后环节，处理器将给调度者发送 信号 request_finished，对与想在当前的 request 中执行的任何东西来说，这是最后的调用。监听这个信号的处理者会清空并释放任何使用中的资源。比如，Django 的 request_finished 监听者会关闭所有数据库连接。

这件事发生以后，处理器会构建一个合适的返回值送返给实例化它的任何东西 （现在，是一个恰当的 mod_python response 或者一个 WSGI 兼容的 response，这取决于处理器）并返回。

这就是 Django 如何处理一个 request。

    

### 6、关于request与response
前面几个 Sections 介绍了关于 Django 请求（Request）处理的流程分析，我们也了解到，Django 是围绕着 Request 与 Response 进行处理，也就是无外乎“求”与“应”。

当请求一个页面时，Django 把请求的 metadata 数据包装成一个 HttpRequest 对象，然后 Django 加载合适的 view 方法，把这个 HttpRequest 对象作为第一个参数传给 view 方法。任何 view 方法都应该返回一个 HttpResponse 对象。



HttpRequest

HttpRequest 对象表示来自某客户端的一个单独的 HTTP 请求。HttpRequest 对象是 Django 自动创建的。

它的属性有很多，可以参考 DjangoBook，比较常用的有以下几个：

1.method 请求方法，如：
```
if request.method == "POST":
    ......
elif request.mehtod =="GET":
    ......
```
2.类字典对象GET、POST  
3.COOKIES，字典形式

4.user：

一个django.contrib.auth.models.User 对象表示当前登录用户，若当前用户尚未登录，user会设为django.contrib.auth.models.AnonymousUser的一个实例。

可以将它们与is_authenticated()区分开：
```
if request.user.is_authenticated():
    ....
else:
    ....
```
5.session、字典形式
6.request.META

具体可以参考官方文档。

request.META 是一个 Python 字典，包含了所有本次 HTTP 请求的 Header 信息，比如用户 IP 地址和用户 Agent（通常是浏览器的名称和版本号）。注意，Header 信息的完整列表取决于用户所发送的 Header 信息和服务器端设置的 Header 信息。 这个字典中几个常见的键值有：

HTTP_REFERRER：进站前链接网页，如果有的话

HTTP_USER_AGENT，用户浏览器的user-agent字符串，如果有的话。 例如： "Mozilla/5.0 (X11; U; Linux i686; fr-FR; rv:1.8.1.17) Gecko/20080829 Firefox/2.0.0.17" .

REMOTE_ADDR 客户端IP，如："12.345.67.89" 。(如果申请是经过代理服务器的话，那么它可能是以逗号分割的多个IP地址，如："12.345.67.89,23.456.78.90" 。)

```
def request_test(request):
    context={}
    try:
        http_referer=request.META['HTTP_REFERRER']
        http_user_agent=request.META['HTTP_USER_AGENT']
        remote_addr=request.META['REMOTE_ADDR']
        return HttpResponse('[http_user_agent]:%s,[remote_addr]=%s' %(http_user_agent,remote_addr))
 	except Exception,e:
 	    return HttpResponse("Error:%s" %e)
```
注意：GET、POST属性都是django.http.QueryDict的实例，在DjangoBook可具体了解。

HttpResponse
Request 和 Response 对象起到了服务器与客户机之间的信息传递作用。Request 对象用于接收客户端浏览器提交的数据，而 Response 对象的功能则是将服务器端的数据发送到客户端浏览器。

比如在 view 层，一般都是以下列代码结束一个 def：
```
return HttpResponse(html)
return render_to_response('nowamagic.html', {'data': data})
```
对于 HttpRequest 对象来说，是由 Django 自动创建, 但是，HttpResponse 对象就必须我们自己创建。每个 View 方法必须返回一个 HttpResponse 对象。HttpResponse 类在 django.http.HttpResponse。

1.构造 HttpRequest

HttpResponse 类存在于 django.http.HttpResponse，以字符串的形式传递给页面。一般地，你可以通过给         HttpResponse 的构造函数传递字符串表示的页面内容来构造 HttpResponse 对象：

    >>> response = HttpResponse("Welcome to nowamagic.net.")
    >>> response = HttpResponse("Text only, please.", mimetype="text/plain")

想要增量添加内容, 你可以把response当作filelike对象使用：

    >>> response = HttpResponse()
    >>> response.write("<p>Welcome to nowamagic.net.</p>")
    >>> response.write("<p>Here's another paragraph.</p>")

也可以给 HttpResponse 传递一个 iterator 作为参数，而不用传递硬编码字符串。 如果你使用这种技术，下面是需要注意的一些事项：

iterator 应该返回字符串。

如果 HttpResponse 使用 iterator 进行初始化，就不能把 HttpResponse 实例作为 filelike 对象使用。这样做将会抛出异常。

最后，再说明一下，HttpResponse 实现了 write() 方法，可以在任何需要 filelike 对象的地方使用 HttpResponse 对象。

2.设置 Headers

你可以使用字典语法添加，删除 headers：

    >>> response = HttpResponse()
    >>> response['X-DJANGO'] = "It's the best."
    >>> del response['X-PHP']
    >>> response['X-DJANGO']
    "It's the best."
