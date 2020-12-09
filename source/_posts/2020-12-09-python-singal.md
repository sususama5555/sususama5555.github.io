---
title: Django | 信号
date: 2020-12-09 23:28:55
tags: 
- Django
- 信号
- signals
categories: 
- Django
- 信号
---

> # 信号

Django 包含一个**信号调度器**，它帮助已解藕的应用程序在框架中的其它地方发生操作时可以得到通知。简而言之，信号允许某些 ***发送器*** 通知一组 ***接收器*** 某些操作已经发生。当许多代码段可能对同一事件感兴趣时，它们特别有用。

Django 提供了 [内置信号集](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/) 使用户代码能够获得 Django 自身某些操作的通知。其中包括一些有用的通知：

- [`django.db.models.signals.pre_save`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.pre_save) & [`django.db.models.signals.post_save`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.post_save)

  一个模型的 [`save()`](https://docs.djangoproject.com/zh-hans/3.1/ref/models/instances/#django.db.models.Model.save) 方法被调用之前或之后发出。

- [`django.db.models.signals.pre_delete`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.pre_delete) & [`django.db.models.signals.post_delete`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.post_delete)

  一个模型的 [`delete()`](https://docs.djangoproject.com/zh-hans/3.1/ref/models/instances/#django.db.models.Model.delete) 方法或查询结果集的 [`delete()`](https://docs.djangoproject.com/zh-hans/3.1/ref/models/querysets/#django.db.models.query.QuerySet.delete) 方法被调用之前或之后发出。

- [`django.db.models.signals.m2m_changed`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.m2m_changed)

  一个模型的 [`ManyToManyField`](https://docs.djangoproject.com/zh-hans/3.1/ref/models/fields/#django.db.models.ManyToManyField) 更改后发出。

- [`django.core.signals.request_started`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.core.signals.request_started) & [`django.core.signals.request_finished`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.core.signals.request_finished)

  Django 发起或结束一个 HTTP 请求后发出。

查看 [内置信号文档](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/) 以获取每个信号的完整列表和说明。

你还可以 [定义和发送自定义信号](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#defining-and-sending-signals)，详细实践见下文。

<!-- more -->

## 监听信号

要接收信号，使用 [`Signal.connect()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal.connect) 方法注册一个 *接收器* 函数。当发送信号时调用接收器。信号的所有接收器函数都按照注册时的顺序一个接一个调用。

**Signal.connect(*receiver*, *sender=None*, *weak=True*, *dispatch_uid=None*)**

**参数:**

- **receiver** -- 将连接到此信号的回调函数。查看 接收器函数 获取更多信息。
- **sender** -- 指定要从其接收信号的特定发送方。查看 连接到特定信号 获取更多信息。
- **weak** -- Django 默认将信号处理程序存储为弱引用。因此，如果你的接收器是本地函数，则可能会对其进行垃圾回收。要防止这种情况发生，当你要调用 `connect()` 方法时请传入 `weak=False`。
- **dispatch_uid** -- 在可能发送重复信号的情况下，信号接收器的唯一标识符。查看 [防止重复信号](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#preventing-duplicate-signals) 获取更多信息。

让我们通过注册一个在每个HTTP请求完成后被调用的信号来看看这是如何工作的。我们将连接到 [`request_finished`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.core.signals.request_finished) 信号。



### 接收器函数

首先，我们需要定义一个接收器函数。一个接收器可以是任何 Python 函数或方法：

```python
def my_callback(sender, **kwargs):
    print("Request finished!")
```

注意，该函数接收一个 `sender` 参数以及关键字参数 (`**kwargs`)；所有信号处理程序都必须接受这些参数。

[稍后](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#connecting-to-signals-sent-by-specific-senders) 我们将讨论发送器，但现在来看下 `**kwargs` 参数。所有信号都发送关键字参数，并且可以随时更改这些关键字参数。在接收 [`request_finished`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.core.signals.request_finished) 信号时，该信号没有发送任何参数，这意味着我们可能会试图将我们的信号处理程序写成 `my_callback(sender)`。

这是错误的——事实上，如果这样做，Django 将抛出一个错误。这是因为在任何时候，参数都可能被添加到信号中，而你的接收器必须能够处理这些新的参数。



### 连接接收器函数

有两种方法可以将接收器连接到信号。你可以选择手动连接线路：

```python
from django.core.signals import request_finished

request_finished.connect(my_callback)
```

或者，你可以使用一个 [`receiver()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.receiver) 装饰器：

- `receiver`(*signal*)[¶](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.receiver)

  参数:**signal** -- 一个用于连接函数的信号或包含多个信号的列表。

以下是你如何使用装饰器连接：

```python
from django.core.signals import request_finished
from django.dispatch import receiver

@receiver(request_finished)
def my_callback(sender, **kwargs):
    print("Request finished!")
```

现在，我们的 `my_callback` 函数将在每次请求完成时被调用。

我的代码该放在哪？

严格来说，信号处理和注册的代码可以放在任何你喜欢的地方，但是推荐避免放在应用程序的根目录和 `models` 模块内以尽量减少导入代码的副作用。

在实践中，信号处理程序通常定义在与之相关的应用程序的 `signals` 子模块中。信号接收器在你应用程序配置类的 [`ready()`](https://docs.djangoproject.com/zh-hans/3.1/ref/applications/#django.apps.AppConfig.ready) 方法中连接。如果你使用 [`receiver()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.receiver) 装饰器，在 [`ready()`](https://docs.djangoproject.com/zh-hans/3.1/ref/applications/#django.apps.AppConfig.ready) 中导入 `signals` 子模块。

注解

[`ready()`](https://docs.djangoproject.com/zh-hans/3.1/ref/applications/#django.apps.AppConfig.ready) 方法在测试过程中可能会多次执行，因此你可能需要 [防止重复信号](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#preventing-duplicate-signals)，尤其是当您计划在测试中发送信号时。



### 连接到特定发送器发送的信号

有些信号被多次发送，但你只对接收这些信号的某个子集感兴趣。例如，仔细考虑 [`django.db.models.signals.pre_save`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.pre_save) 在模型保存之前发送的信号。大多数时候，你不需要知道 *任何* 模型何时被保存——只需要知道某个 *特定* 模型何时被保存。

在这些情况下，您可以注册以接收仅由特定发送者发送的信号。在接收 [`django.db.models.signals.pre_save`](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/#django.db.models.signals.pre_save) 信号时 ，发送器会是要保存的模型类，因此你就可以表明你想要某个模型发送的信号：

```python
from django.db.models.signals import pre_save
from django.dispatch import receiver
from myapp.models import MyModel


@receiver(pre_save, sender=MyModel)
def my_handler(sender, **kwargs):
    ...
```

`my_handler` 函数将仅在 `MyModel` 实例保存后被调用。

不同的信号使用不同的对象作为它们的发送者；你需要查阅 [内置信号文档](https://docs.djangoproject.com/zh-hans/3.1/ref/signals/) 了解每个特定信号的详细信息。



### 防止重复信号

在某些情况下，连接接收器到信号的代码可能被执行多次。这可能会导致接收器函数被注册多次，因此对于一个信号事件调用同样多次。例如，[`ready()`](https://docs.djangoproject.com/zh-hans/3.1/ref/applications/#django.apps.AppConfig.ready) 方法在测试期间可能被多次执行。更普遍的是，在项目的任何地方导入定义信号的模块都会发生这种情况，因为信号注册的运行次数与导入的次数相同。

如果此行为会产生问题（例如在保存模型时使用信号发送电子邮件），则传递一个唯一标识符作为 `dispatch_uid` 参数来标识接收方函数。这个标识符通常是一个字符串，尽管任何可散列对象都可以。最终的结果是，对于每个唯一的 `dispatch_uid` 值，接收器函数只与信号绑定一次：

```python
from django.core.signals import request_finished

request_finished.connect(my_callback, dispatch_uid="my_unique_identifier")
```



## 定义和发送信号

您的应用程序可以利用信号基础设施并提供自己的信号。

何时使用自定义信号

信号是隐式函数调用，这使得调试更加困难。如果你的自定义信号的发送器和接收器都在你的项目内，最好使用显式函数调用。



### 定义信号

- *class* `Signal`

所有的信号都是 [`django.dispatch.Signal`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal) 的实例。

例如:

```python
import django.dispatch

pizza_done = django.dispatch.Signal()
```

这声明了一个 `pizza_done` 信号。



### 发送信号

在 Django 中有两种发送信号的方法。

- `Signal.send`(*sender*, ***kwargs*)

  

- `Signal.send_robust`(*sender*, ***kwargs*)

  

要发送信号，调用 [`Signal.send()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal.send) （所有内置信号使用这个）或者 [`Signal.send_robust()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal.send_robust)。你必须提供 `sender` 参数（大多数情况下是一个类），还可以根据需要提供任意多个其他关键字参数。

例如，发送 `pizza_done` 信号可能看起来如下：

```python
class PizzaStore:
    ...

    def send_pizza(self, toppings, size):
        pizza_done.send(sender=self.__class__, toppings=toppings, size=size)
        ...
```

`send()` 和 `send_robust()` 都返回一个元组对列表 `[(receiver, response), ... ]`，表示被调用的接收器函数及其响应值的列表。

`send()` 和 `send_robust()` 在处理接收器函数所引发异常的方式上有所不同。 `send()` *不* 捕获接收器引起的任何异常；它只是允许错误传播。因此，并非所有的接收器都会在出现错误时被通知信号。

`send_robust()` 捕获从 Python的 `Exception` 类派生的所有错误，并确保所有接收器都收到信号通知。如果发生错误，将在引发错误的接收器的元组对中返回错误实例。

回溯出现在调用 `send_robust()` 时返回的错误中的 `__traceback__` 属性中。



## 断开信号

- `Signal.disconnect`(*receiver=None*, *sender=None*, *dispatch_uid=None*)

要断开接收器与信号的连接，调用 [`Signal.disconnect()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal.disconnect)。参数已在 [`Signal.connect()`](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/#django.dispatch.Signal.connect) 描述。该方法成功断开连接返回 `True` 否则返回 `False` 。

`receiver` 参数表明要断开的接收器。它可以是 `None` 如果 `dispatch_uid` 已经被用来标识接收器。



******

参考资料：

[django官方文档 - 信号](https://docs.djangoproject.com/zh-hans/3.1/topics/signals/)