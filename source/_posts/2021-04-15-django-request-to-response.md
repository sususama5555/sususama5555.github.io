---
title: Django - 从请求到响应
date: 2021-04-15 00:16:39
tags: 
- Django
- 源码分析
- 请求与响应
- 编程框架
categories: 
- Django
copyright: true
---

`Django` 版本：`2.2.6`

本文并不局限于某一个 `Django` 版本但都会尽量讨论版本 `2.0+`

## 流程总览

![Django 处理请求流程][1]

### 概述：

`Django` 和其他 `Web` 框架的 `HTTP` 处理的流程大致相同：先通过 `Request Middleware` 对请求对象做定义处理，然后再通过默认的 `URL` 指向的方法，最后再通过 `Response Middleware` 对响应对象做自定义处理。

<!--more-->

### 细则：

1. [启动->WSGI]通过任意方式启动 `Django` 创建 `WSGIServer` 类的实例
2. 用户通过浏览器请求某个 `Django` 页面
3. [WSGI]`Django WSGIServer` 接收客户端（浏览器）请求初始化 `WSGIHandler` 实例
4. [WSGI->加载配置]导入 `setting` 配置和 `Django` 异常类
5. [WSGI->中间件]加载 `setting` 中设置的中间件
6. [中间件]创建 `_request_middleware,_view_middleware,_response_middleware,_exception_middleware` 四个列表
7. [中间件]遍历执行 `_request_middleware`，对 `request` 进行处理：若返回 `None` 进入到 8；若直接返回 `HttpResponse` 对象进入到 12
8. [URL Resolver]解析 `url` 并进行匹配（假设匹配成功）
9. [中间件]遍历执行 `_view_middleware`，对 `request` 进行处理：若返回 `None` 进入到 10；若直接返回 `HttpResponse` 对象进入到 12。
10. [中间件]实现 `url` 匹配的 `view` 逻辑：若引发异常进入到 11;若正常返回 `HttpResponse` 对象进入到 12
11. [中间件]遍历执行 `_exception_middleware`
12. [中间件]遍历执行 `_response_middleware`，对 `HttpResponse` 进行处理并最终返回 `response`

## 启动

在开发环境中，我们一般是通过命令行执行 `runserver` 命令，`ruserver` 命令是使用 `Django` 自带的的 `Web Server`，而在正式的环境中，一般会使用 `Nginx+uWSGI` 模式。

无论通过哪种方式，启动一个项目时，都会做两件事：

1. 创建一个 `WSGIServer` 类的实例，来接受用户的请求。
2. 当一个用户的 `HTTP` 请求到达的时，为用户指定一个 `WSGIHandler`，用于处理用户请求与响应，这个 `Handler` 是处理整个 `Request` 的核心。

## WSGI

`WSGI`：全称 `Web Server Gateway Interface`。

`WSGI` 不是服务器，`Python` 模块，框架，`API` 或者任何软件，只是一种规范，描述 `Web Server` 如何与 `Web Application` 通信的规范。

`WSGI` 协议主要包括 `server` 和 `application` 两部分：

1. `WSGI Server` 负责从客户端接收请求，将 `request` 转发给 `application`，将`application` 返回的 `response` 返回给客户端；
2. `WSGI Application` 接收由 `server` 转发的 `request`，处理请求，并将处理结果返回给 `server`。`application` 中可以包括多个栈式的中间件(middlewares)，这些中间件需要同时实现 `server` 与 `application`，因此可以在 `WSGI` 服务器与 `WSGI` 应用之间起调节作用：对服务器来说，中间件扮演应用程序，对应用程序来说，中间件扮演服务器。

### Django WSGI Application

`WSGI Application` 应该实现为一个可调用对象，例如：函数、方法、类（包含 `call` 方法）。

需要接收两个参数：
1. 包含客户端请求的信息以及其他信息的字典。可以认为是请求上下文，一般叫做`environment`（编码中多简写为 `environ`、`env`）；
2. 用于发送 `HTTP` 响应状态（`HTTP Status`）、响应头（`HTTP Headers`）的回调函数；

通过回调函数将响应状态和响应头返回给 `WSGI Server`，同时返回响应正文，响应正文是可迭代的、并包含了多个字符串。下面是 `Django WSGI Application` 的具体实现：

```python
class WSGIHandler(base.BaseHandler):
    request_class = WSGIRequest

    def __init__(self, *args, **kwargs):
        super().__init__(*args, **kwargs)
        # 注：加载中间件
        self.load_middleware()

    def __call__(self, environ, start_response):
        set_script_prefix(get_script_name(environ))
        # 注：处理请求前发送信号
        signals.request_started.send(sender=self.__class__, environ=environ)
        request = self.request_class(environ)
        response = self.get_response(request)

        response._handler_class = self.__class__

        status = '%d %s' % (response.status_code, response.reason_phrase)
        response_headers = list(response.items())
        for c in response.cookies.values():
            response_headers.append(('Set-Cookie', c.output(header='')))
        # 注：将响应的 header 和 status 返回给 server
        start_response(status, response_headers)
        if getattr(response, 'file_to_stream', None) is not None and environ.get('wsgi.file_wrapper'):
            response = environ['wsgi.file_wrapper'](response.file_to_stream)
        return response
```

可以看出 `Django WSGI Application` 的流程包括:

1. 加载所有中间件，以及执行框架相关的操作，设置当前线程脚本前缀，发送请求开始信号；
2. 处理请求，调用 `get_response()` 方法处理当前请求，该方法的的主要逻辑是通过`urlconf` 找到对应的 `view` 和 `callback`，按顺序执行各种 `middleware` 和 `callback`；
3. 调用由 `WSGI Server` 传入的 `start_response()` 方法将响应 `header` 与 `status` 返回给 `WSGI Server`；
4. 返回响应正文。

### Django WSGI Server

负责获取 `HTTP` 请求，将请求传递给 `Django WSGI Application`，由 `Django WSGI Application` 处理请求后返回 `response`。以 `Django` 内建 `server` 为例看一下具体实现。

通过 `runserver` 命令运行 `Django` 项目，在启动时都会调用下面的 `run` 方法，创建一个 `WSGIServer` 的实例，之后再调用其 `serve_forever()` 方法启动服务。

```python
def run(addr, port, wsgi_handler, ipv6=False, threading=False, server_cls=WSGIServer):
    server_address = (addr, port)
    if threading:
        httpd_cls = type('WSGIServer', (socketserver.ThreadingMixIn, server_cls), {})
    else:
        httpd_cls = server_cls
    httpd = httpd_cls(server_address, WSGIRequestHandler, ipv6=ipv6)
    if threading:
        httpd.daemon_threads = True
    httpd.set_app(wsgi_handler)
    httpd.serve_forever()
```

下图表示 `WSGI Server` 服务器处理流程中关键的类和方法（来自：[参考引用_1][2]）

![WSGI Server 服务器处理流程中关键的类和方法][5]

#### 1. WSGIServer

`run()` 方法会创建 `WSGIServer` 实例，主要作用是接收客户端请求，将请求传递给`WSGI Application`，然后将 `WSGI Application` 返回的 `response` 返回给客户端。

* 创建实例时会指定 `HTTP` 请求的 `handler` ：`WSGIRequestHandler` 类;
* 通过 `set_app` 和 `get_app` 方法设置和获取 `WSGIApplication` 实例`wsgi_handler`;
* 处理 `HTTP` 请求时，调用 `handler_request` 方法，会创建 `WSGIRequestHandler` 实例处理 `HTTP` 请求;
* `WSGIServer` 中 `get_request` 方法通过 `socket` 接受请求数据;

#### 2. WSGIRequestHandler

* 由 `WSGIServer` 在调用 `handle_request` 时创建实例，传入 `request,cient_address,WSGIServer` 三个参数，`__init__` 方法在实例化同时还会调用自身的 `handle` 方法；
* `handle` 方法会创建 `ServerHandler` 实例，然后调用其 `run` 方法处理请求；

#### 3. ServerHandler

* `WSGIRequestHandler` 在其 `handle` 方法中调用 `run` 方法，传入`self.server.get_app()` 参数，获取 `WSGIApplication`，然后调用实例(__call__)，获取 `response`，其中会传入 `start_response` 回调，用来处理返回的 `header` 和 `status`；
* 通过 `application` 获取 `response` 以后，通过 `finish_response` 返回 `response`；

#### 4. WSGIHandler（即 Django WSGI Application）

* `WSGI` 协议中的 `application`，接收两个参数，`environ` 字典包含了客户端请求的信息以及其他信息，可以认为是请求上下文，`start_response` 用于发送返回 `status` 和 `header` 的回调函数

虽然上面一个 `Django WSGI Server` 涉及到多个类实现以及相互引用，但其实原理还是调用`WSGIHandler`，传入请求参数以及回调方法 `start_response()`，并将响应返回给客户端。

### Python wsgiref simple_server

在 `Python3.7` 的源码中给出了一个 `simple_server` 案例位于 `python3.7/wsgiref/simple_server.py`。

模块实现了一个简单的 `HTTP` 服务器，并给出了一个简单的 `demo`，可以直接运行，运行结果会将请求中涉及到的环境变量在浏览器中展示出来。其中包括上述描述的整个 `HTTP` 请求的所有组件：`ServerHandler,WSGIServer,WSGIRequestHandler` 以及 `demo_app` 表示的简易版的 `WSGIApplication`。

感兴趣的话可以自己去看一下源码。

## 加载配置

`Django` 的配置都在 `{project_name}/settings.py` 中定义，可以是 `Django` 的配置，也可以是自定义的配置，并且都通过 `django.conf.settings` 访问。

## 中间件-Middleware

### 概述：

`Django` 中的 `Middleware` 类似底层中一个轻量级的插件系统，它能够介入 `Django` 的请求和响应过程，在全局修改 `Django` 的输入和输出内容。从流程总览图中可以看出 `Django` 请求处理过程的核心在于 `Middleware`，`Django` 中所有的请求和响应都有 `Middleware` 的参与。

### 细则：

![中间件流程图][7]

一个 `HTTP` 请求，首先被转化成一个 `HttpRequest` 对象，然后该对象被传递给 `Request Middleware` 处理，如果它返回了 `HttpResponse` 对象，则直接传递给 `Response Middleware` 做收尾处理。否则的话 `Request Middleware` 将访问 `URL` 配置，确定目标 `view` 来处理 `HttpRequest` 对象，在确定了 `view`，但是还没有执行时候，系统会把 `HttpRequest` 对象传递给 `View Middleware` 进行处理，如果它返回了 `HttpResponse` 对象，那么该 `HttpResponse` 对象将被传递给 `Response Middleware` 进行后续处理，否则将执行确定的 `view` 函数处理并返回 `HttpResponse` 对象，在整个过程中如果引发了异常并抛出，会被 `Exception Middleware` 进行处理。

### 中间件执行顺序

![中间件流程图][6]

在请求阶段，调用视图之前，`Django` 按照 `setting.py` 设置的顺序，自顶向下应用遍历执行 `Request Middleware`。你可以把它想象成一个洋葱：每个中间件类都是一个“层”，它覆盖了洋葱的核心。如果请求通过洋葱的所有层（每一个调用 `get_response`）以将请求传递到下一层，一直到内核的视图，那么响应将在返回的过程中通过每个层（以相反的顺序）。

### 如何编写自己的中间件即中间件的深入了解

编写一个自己的中间件是很容易的，每个中间件组件都是一个独立的 `Python Class`，你可以在自定义的 `Class` 下编写一个或多个下面的方法：

#### process_request

函数样式：`process_request(request)`；

参数解析：`request` 是一个 `HTTPRequest` 对象；

调用时间：在 `Django` 决定执行哪个 `view` 之前，`process_request()` 会被请求调用；

产生响应：它应该返回一个 `None` 或一个 `HttpResponse` 对象，如果返回 `None`，`Django` 会继续处理这个请求；如果它返回一个 `HTTPResponse` 对象，`Django` 会直接跳转到 `Response Middleware`；

#### process_view

函数样式：`process_view(request, view_func, view_args, view_kwargs)`；

参数解析：`request` 是一个 `HTTPRequest` 对象，`view_func` 是 `Django` 会调用的一个函数（准确的说是一个函数对象而非一个表示函数名的字符串），`view_args` 是一个会被传递到视图的 `*args`，`view_kwargs` 是一个会被传递到视图的 `**kwargs`，`view_args` 和 `view_kwargs` 都不包括 `request`；

调用时间：`process_view()` 会在 `Django` 调用 `view` 前被调用；

产生响应：它应该返回一个 `None` 或一个 `HttpResponse` 对象，如果返回 `None`，`Django` 会继续处理这个请求；如果它返回一个 `HTTPResponse` 对象，`Django` 会直接跳转到 `Response Middleware`；

PS：除 `CsrfViewMiddleware ` 外中间件运行时在视图运行前或在 `process_view()` 中访问 `request.POST` 会使得之后的所有视图无法修改 `request`，所以应该尽量避免。

#### process_template_response

函数样式：`process_template_response(request, response)`；

参数解析：`request` 是一个 `HttpRequest` 对象，`response` 是一个 `TemplateResponse` 对象（或类似对象），由 `Django` 视图或中间件返回；

调用时间：如果 `response` 的实例有 `render()` 方法，`process_template_response()` 在视图刚好执行完毕之后被调用，这表明他是一个 `TemplateResponse` 对象（或类似对象）；

产生响应：这个方法必须返回一个实现了 `render()` 方法的 `TemplateResponse` 对象（或类似对象），它可以修改给定的 `response` 对象，也可以创建一个全新的 `TemplateResponse` 对象（或类似对象）；

PS：在响应处理阶段，中间件以相反的顺序运行，包括 `process_template_response`；

#### process_response

函数样式：`process_response(request, response)`；

参数解析：`request` 是一个 `HttpRequest` 对象，`response` 是一个 `HttpResponse` 对象，由 `Django` 视图或中间件返回；

调用时间：`process_request` 在所有响应返回客户端前被调用;

产生响应：这个方法必须返回一个 `HttpRequest` 对象，它可以修改给定的 `response` 对象，也可以创建一个全新的 `HttpRequest` 对象；

PS：`process_response` 总是被调用，这意味着你的 `process_response` 不能依赖 `process_request`

#### process_exception

函数样式：`process_exception(request, exception)`；

参数解析：`request` 是一个 `HttpRequest` 对象，`exception` 是一个被视图抛出 `Exception` 对象；

调用时间：当一个视图抛出异常，`Django` 会调用 `process_exception` 来处理;

产生响应：它应该返回一个 `None` 或一个 `HttpResponse` 对象，如果返回 `None`，`Django` 会继续处理这个请求；如果它返回一个 `HTTPResponse` 对象，模板对象和 `Response Middleware` 会被直接返回给客户端；否则，将启动默认异常处理。；

## URL Resolver

### 概述：

假设：中间件便利执行完 `_request_middleware,_view_middleware` 后都返回 `None`。

当 `Django` 遍历执行完 `_request_middleware` 后会得到一个经过处理的 `request` 对象，此时 `Django` 将按顺序进行对 `url` 进行正则匹配，如果匹配不成功，就会抛出异常；如果匹配成功，`Django` 会继续循环执行 `_view_middleware` 并在执行后继续执行刚刚匹配成功的 `view`。

在 `setting` 中有一个 `ROOT_URLCONF`，它指向 `urls.py` 文件，根据这个文件可以生产一个 `urlconf`，本质上，他就是 `url` 与视图函数之间的映射表，然后通过 `resolver` 解析用户的 `url`，找到第一个匹配的 `view`。

### 细则：

![URL Resolver流程图][9]

重要函数源码位置：

```
_path: django/urls/conf.py
URLPattern: django/urls/resolvers.py
ResolverMatch: django/urls/resolvers.py
URLResolver: django/urls/resolvers.py
```

源码比较长，就不放出来了，感兴趣的话自己去看吧。

1. 通过 `urlpatterns` 的配置执行 `_path` 函数；
2. `_path` 函数进行判断：如果是一个 `list` 或者 `tuple`，就用 `URLResolver` 处理，跳至 4；如果是一个正常的可调用的 `view` 函数，则用 `URLPattern` 处理，跳至；如果匹配失败，抛出异常；
3. `URLPattern` 初始化相应值后执行 `resolve` 方法：如果匹配成功，返回 `ResolverMatch`；如果匹配失败，抛出异常；
4. `URLResolver` 匹配 `path` 如果匹配成功，则继续匹配它的 `url_patterns`，跳至 5；匹配失败，抛出异常；
5. 匹配 `url_patterns`：若为 `urlpattern` 匹配成功，返回 `ResolverMatch`；若为 `URLResolver` 递归调用 `URLResolver` 跳至 4；若匹配失败，抛出异常；

可以发现，整个过程的关键就是 `ResolverMatch`，`URLPattern` 和 `URLResolver` 三个类，其中：
`ResolverMatch` 是匹配结果，包含匹配成功后需要的信息；
`URLPattern` 是一个 `url` 映射信息的对象，包含了 `url` 映射对应的可调用对象等信息；
`URLResolver` 是实现 `url` 路由，解析 `url` 的关键的地方，它的 `url_patterns`既可以是`URLPattern` 也可以是 `URLResolver`，正是因为这种设计, 实现了对 `url` 的层级解析。

## 总述

真实的请求响应过程肯定是比我提到的这些还要复杂的多，但是我的能力实在有限，目前仅能理解到这个层面了，如果错误欢迎指正。


参考引用：
1. [简书：做Python Web开发你要理解：WSGI & uWSGI 作者：rainybowe][2]
2. [掘金：Django从请求到响应的过程 作者：__奇犽犽][3]
3. [现代魔法学院：Python 与 Django 篇-Django 架构流程分析][4]
4. [简书：django源码分析之url路由(URLResolver) 作者：2hanson][8]
5. [Django 官方文档][10]
6. [Django 笔记-1-从请求到响应][11]

[1]:https://img.blanc.site/wiki/img/30.png
[2]:https://www.jianshu.com/p/679dee0a4193
[3]:https://juejin.im/post/5a6c4cc2f265da3e4c080605
[4]:http://www.nowamagic.net/academy/detail/13281808
[5]:https://img.blanc.site/wiki/img/31.png
[6]:https://img.blanc.site/wiki/img/32.png
[7]:https://img.blanc.site/wiki/img/33.png
[8]:https://www.jianshu.com/p/5b7bb930b170
[9]:https://img.blanc.site/wiki/img/34.png
[10]:https://docs.djangoproject.com/en/2.1/
[11]:https://wiki.blanc.site/archives/953c2552.html