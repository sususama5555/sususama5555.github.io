---
title: Pytest在Django、DRF中的最佳实践
date: 2022-01-02 00:32:36
tags: 
- Python
- 测试
- TDD
- Pytest
- pytest-django
categories: 
- Python
- 测试
- Pytest
---

## 前言

上一篇文章写了Pytest的基础用法

{% post_link python-pytest 回顾上一篇《Pytest自动化测试框架》 %}

由于最新启动的Django项目，最终选型Pytest作为测试框架，就想着分享下测试代码的最佳实践。

> 为什么不是unittest
>
> unittest作为官方的测试框架，在测试方面更加基础，并且可以再次基础上进行二次开发，同时在用法上格式会更加复杂；而pytest框架作为第三方框架，方便的地方就在于使用更加灵活，并且能够对原有unittest风格的测试用例有很好的兼容性，同时在扩展上更加丰富，可通过扩展的插件增加使用的场景，比如一些并发测试等；

<!-- more -->

## 依赖

这里笔者使用了poetry，主要依赖：

```
pytest = "6.2.4"
pytest-django = "3.9.0"
mock = "3.0.5"
requests-mock = "1.9.3"
```

是pytest、pytest-django（django的插件包）和mock模块

> pytest-django is a plugin for [pytest](https://pytest.org/) that provides a set of useful tools for testing [Django](https://www.djangoproject.com/)applications and projects.

## 配置

一般在项目根目录创建`pytest.ini`文件，对项目的pytest进行配置，更多配置可参考官网

以下是我的配置：

```
[pytest]
DJANGO_SETTINGS_MODULE = backend.settings.dev
testpaths = backend/tests
addopts = --reuse-db
```

稍微解释一下：

- DJANGO_SETTINGS_MODULE为指定Django项目的settings文件，这里可以指定dev环境，区分测试、生产的配置，对开发人员友好
- testpaths为该项目的测试代码根目录，好像不写也问题不大，因为上一篇讲到，Pytest框架会自动识别测试文件和代码
- addopts为额外的操作选项，--reuse-db的意思为，重复使用测试的数据库，不然的话每次执行测试时，都会重新创建数据库，然后执行`migrate`操作，比较费时

配置完成后，然后命令行运行，就开始测试啦

```shell
pytest
```

## 目录结构

一切就绪，来看一下是测试框架的目录结构：

> 可以用tree命令生成目录结构

```
backend/tests
├── __init__.py
├── conftest.py
├── database
│   ├── __init__.py
│		├── conftest.py
│   └── mysql
│       ├── __init__.py
│       └── test_mysql_connect.py
└── km
    ├── __init__.py
    ├── conftest.py
    └── test_article.py
```

我们拿km目录当作知识库的测试目录，以下代码围绕这个来讲解

## 测试代码demo

先来看点测试代码：

> 以下为笔者改造的知识库的文章接口测试，如有雷同，纯属巧合

test_article.py

```python
# -*- coding: utf-8 -*-
import pytest
from rest_framework import status

from backend.km.models import Article


class TestArticleViewSet:
    def test_create_article(self, api_client, create_article_params):
        for params in create_article_params:
            response = api_client.post("/apis/km/articles/", params)
            assert response.status_code == status.HTTP_201_CREATED
            assert "id" in response.data
        assert Article.objects.count() == len(create_article_params)

    @pytest.mark.usefixtures("init_article_data")
    def test_list_article(self, api_client):
        response = api_client.get("/apis/km/articles/")
        data = response.data
        assert data["count"] == len(data["results"]) == 2

    @pytest.mark.usefixtures("init_article_data")
    def test_get_article(self, api_client, article_id):
        response = api_client.get(f"/apis/km/articles/{article_id}/")
        data = response.data
        assert data["id"] == article_id

    @pytest.mark.usefixtures("init_article_data")
    def test_article_comment(self, api_client, article_id):
        response = api_client.get(f"/apis/km/articles/{article_id}/comments/")
        data = response.data
        assert len(data) == 1
        assert data[0]["id"] == article_id
        assert len(data[0]["likes"])

```

大体思路就是，请求DRF的接口，然后将返回的数据与期望的数据做对比

一般用`assert`关键字来验证数据符合性，不然就抛出异常，表示该单元测试失败

> 所以我的理解里，测试代码的正确性，应该以能否获取该接口期望的数据格式为判断条件，也就是说，前端、客户端需要什么，测试的代码就应该验证什么。这似乎决定了测试的assert并没有严格的标准。

这里的fixture固件用法疑点重重，听我慢慢分析

### 固件fixtures

#### api_client

**api_client**是我们用来调用接口的client，十分通用，我们把它放在测试根目录的conftest.py里：

```python
# -*- coding: utf-8 -*-
import mock
import pytest
from django.contrib.auth import get_user_model
from django.utils.crypto import get_random_string
from rest_framework.test import APIClient


@pytest.fixture
def test_user():
    User = get_user_model()
    user = User.objects.create(username=get_random_string(6))

    user.token = mock.MagicMock()
    user.token.access_token = get_random_string(12)
    user.token.expires_soon = lambda: False

    return user


@pytest.fixture
def api_client(test_user):
    client = APIClient()
    client.force_authenticate(user=test_user)
    return client

```

这里我们定义了api_client这个fixture，同时他依赖test_user这个fixture

- test_user用来生成一个随机的Django内置用户，并且为其token赋值，这等于是mock了一个用于测试的用户
- APIClient是`Django REST framework`测试模块中，自带的API请求客户端，继承了DjangoClient，为了更好地适配DRF，下面会写到DRF对测试的支持，client用法和requests包很类似。

- api_client在拿到test_user这个随机用户后，把用户赋予给这个client，是对token的强制认证，不然通不过Django的认证中间件

然后使用`api_client.post("/apis/articles/", params)`就可以请求到restful API风格的接口

```
<!-- ATTENTION!
Pytest中，所有对数据库有操作的测试代码，都必须使用@pytest.mark.django_db装饰器
例如：
@pytest.mark.django_db
@pytest.fixture()
def init_ticket_data():
    for params in create_ticket_params:
        instance = Ticket.objects.create()
        TicketFields.objects.create(ticket=instance, **params)
改fixture需要用到Ticket等数据库模型来查询数据库，就必须加上此装饰器，不然会被数据库拒绝连接

还有一种方法，在测试代码文件开头，或者__init__.py文件中声明
import pytest

pytestmark = pytest.mark.django_db
-->
```

#### create_article_params和article_id

```python
# -*- coding: utf-8 -*-
import random

import pytest

from backend.km.models import Article

CREATE_ARTICLE_PARAMS = [
    {
        "title": "pytest基础用法",
        "content": "..........",
        "expire_time": "30",
        "category": "程序开发"
    },
    {
        "title": "pytest最佳实践",
        "content": "..........",
        "expire_time": "180",
        "category": "程序开发",
    },
]


@pytest.fixture
def article_id():
    """生成随机单据ID"""
    return random.choice(list(Article.objects.values_list("id", flat=True)))


@pytest.fixture
def create_article_params():
    return CREATE_ARTICLE_PARAMS


@pytest.fixture
def init_article_data(test_username):
    for params in CREATE_ARTICLE_PARAMS:
        params.update(creator=test_username)
        instance = Article.objects.create(**params)
        instance.create_comments()
        instance.create_likes()
```

article_id：从Article这张表中，随机一个主键id，我们在测试update、detail等接口会用到

init_article_data：给数据库创建几条数据，提供给list、detail等接口查询

但是这二者的用法也不同：

```python
@pytest.mark.usefixtures("init_article_data")
def test_get_article(self, api_client, article_id):
    response = api_client.get(f"/apis/km/articles/{article_id}/")
    data = response.data
    assert data["id"] == article_id
```

由此可见，api_client和article_id是直接使用的fixture方法，是有返回值的

而init_article_data，是没有返回值的，我们只是用来初始化数据，它类似于测试代码的前置运行时。

> **usefixtures与传fixture区别**：
>
> 如果fixture有返回值，那么usefixture就无法获取到返回值，这个是装饰器usefixture与用例直接传fixture参数的区别。
>
> 当fixture需要用到return出来的参数时，只能讲参数名称直接当参数传入，不需要用到return出来的参数时，两种方式都可以。

## DRF对测试的支持

> REST framework 包含一些扩展 Django 现有测试框架的助手类，并改进对 API 请求的支持。

这里强烈推荐去看源码，以下列举了三种请求接口的方式，重点是APIClient

### APIRequestFactory

扩展了 Django 现有的 `RequestFactory` 类。

`APIRequestFactory` 类支持与 Django 的标准 `RequestFactory` 类几乎完全相同的 API。这意味着标准的 `.get()`, `.post()`, `.put()`, `.patch()`, `.delete()`, `.head()` 和 `.options()` 方法都可用。

```python
from rest_framework.test import APIRequestFactory

factory = APIRequestFactory()
request = factory.post('/notes/', {'title': 'new idea'})
```

### APIClient

APIClient继承了APIRequestFactory和DjangoClient，扩展了 Django 现有的 `Client` 类。

`APIClient` 类支持与 Django 标准 `Client` 类相同的请求接口。这意味着标准的 `.get()`, `.post()`, `.put()`, `.patch()`, `.delete()`, `.head()` 和 `.options()` 方法都可用。例如：

```python
from rest_framework.test import APIClient

client = APIClient()
client.post('/notes/', {'title': 'new idea'}, format='json')
```

#### 认证.login

`login` 方法的功能与 Django 的常规 `Client` 类一样。这使你可以对任何包含 `SessionAuthentication` 的视图进行身份验证。

```python
client = APIClient()
client.login(username='sapphire', password='123456')
```

要登出，请照常调用 `logout` 方法。

```python
client.logout()
```

#### .credentials(**kwargs)

`credentials` 方法可用于设置 header，这些 header 将包含在测试客户端的所有后续请求中。

```python
from rest_framework.authtoken.models import Token
from rest_framework.test import APIClient

token = Token.objects.get(user__username='sapphire')
client = APIClient()
client.credentials(HTTP_AUTHORIZATION='Token ' + token.key)
```

请注意，第二次调用 `credentials` 会覆盖任何现有凭证。你可以通过调用没有参数的方法来取消任何现有的凭证。

```python
client.credentials()
```

`credentials` 方法适用于测试需要验证 header 的 API，例如 basic 验证，OAuth1 和 OAuth2 验证以及简单令牌验证方案。

#### .force_authenticate(user=None, token=None)

有时你可能想完全绕过认证，强制测试客户端的所有请求被自动视为已认证。

如果你正在测试 API 但是不想构建有效的身份验证凭据以进行测试请求，则这可能是一个有用的捷径。

```python
user = User.objects.get(username='sapphire')
client = APIClient()
client.force_authenticate(user=user)
```

要对后续请求进行身份验证，请调用 `force_authenticate` 将 user 和(或) token 设置为 `None`。

```python
client.force_authenticate(user=None)
```

#### CSRF 验证

默认情况下，使用 `APIClient` 时不应用 CSRF 验证。如果你需要明确启用 CSRF 验证，则可以通过在实例化客户端时设置 `enforce_csrf_checks` 标志来实现。

```python
client = APIClient(enforce_csrf_checks=True)
```

像往常一样，CSRF 验证将仅适用于任何会话验证视图。这意味着 CSRF 验证只有在客户端通过调用 `login()` 登录后才会发生。

### RequestsClient

REST framework 还包含一个客户端，用于使用流行的 Python 库 `requests` 与应用程序进行交互。 这可能是有用的，如果：

- 你期望主要从另一个 Python 服务与 API 进行交互，并且希望在与客户端相同的级别测试该服务。
- 你希望以这样的方式编写测试，以便它们也可以在分段或实时环境中运行。

它暴露了与直接使用请求会话完全相同的接口。

```python
client = RequestsClient()
response = client.get('http://testserver/users/')
assert response.status_code == 200
```



------

参考资料：

[pytest-django Documentation](https://pytest-django.readthedocs.io/en/latest/index.html)

[Django REST framework API 指南：测试](https://blog.csdn.net/weixin_34080951/article/details/87985073)

