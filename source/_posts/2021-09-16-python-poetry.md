---
title: Poetry - 更好的Python依赖管理工具
date: 2021-09-16 23:53:32
tags: 
- Python
- Poetry
- pip
- pipenv
- 依赖管理
categories: 
- Python
- 依赖管理
---

## Poetry是什么

Poetry 和 Pipenv 类似，是一个 Python 虚拟环境和依赖管理工具，另外它还提供了包管理功能，比如打包和发布。你可以把它看做是 Pipenv 和 Flit 这些工具的超集。它可以让你用 Poetry 来同时管理 Python 库和 Python 程序。

相比于`pip`、 `pipenv`和`requirements.txt`，`Poetry`更加强大、对依赖维护更友好

Pipenv 描绘了一个美梦，让我们以为 Python 也有了其他语言那样完善的包管理器，不过这一切却在后来者 Poetry 这里得到了更好的实现。

<!-- more -->

## 安装 Poetry

官方推荐的安装命令是使用自带的 get-poetry.py 脚本，使用 curl：

```shell
$ curl -sSL https://raw.githubusercontent.com/sdispater/poetry/master/get-poetry.py | python
```

或者直接下载这个安装脚本 [get-poetry.py](https://github.com/sdispater/poetry/blob/master/get-poetry.py)，然后在本地执行。

也可以用 pip 安装（不过 Poetry 官方文档不建议这么做，因为有可能会造成依赖冲突，可以考虑用 [pipx](https://github.com/pipxproject/pipx) 或 [pipsi](https://github.com/mitsuhiko/pipsi)）：

```shell
$ pip install --user poetry
```

安装后可以使用下面的命令确认安装成功：

```shell
$ poetry --version
Poetry 0.12.17
```

*附注：在 Mac 上安装报错 ~/.bash_profile 权限错误，临时没管，也能正常运行。后续可以考虑手动把 ~/.poetry/bin 加进去。*

## Poetry 的基本用法

Poetry 的用法很简单，大部分命令和 Pipenv 接近。我们需要先了解一些基本概念和 Tips：

- 使用 [PEP 518](https://www.python.org/dev/peps/pep-0518/) 引入的新标准 pyproject.toml 文件管理依赖列表和项目的各种 meta 信息，用来替代 Pipfile、requirements.txt、setup.py、setup.cfg、MANIFEST.in 等等各种配置文件。
- 依赖分为两种，普通依赖（生产环境）和开发依赖。
- 安装某个包，会在 pyproject.toml 文件中默认使用 upper bound版本限定，比如 Flask^1.1。这被叫做 Caret requirements，比如某个依赖的版本限定是 ^2.9.0，当你执行 poetry update 的时候，它或许会更新到 2.14.0，但不会更新到 3.0.0；假如固定的版本是 ^0.1.11，它可能会更新到 0.1.19，但不会更新到 0.2.0。总之，在更新依赖的时候不会修改最左边非零的数字号版本（对于 SemVer 版本号而言），这样的默认设定可以确保你在更新依赖的时候不会更新到具有不兼容变动的版本。另外也支持更多依赖[版本限定符号](https://github.com/sdispater/poetry#dependencies-and-dev-dependencies)。
- 不会像 Pipenv 那样随时更新你的锁定依赖版本，锁定依赖存储在`poetry.lock` 文件里（这个文件会自动生成）。所以，记得把你的 poetry.lock 文件纳入版本控制。
- 执行 poetry 或 poetry list 命令查看所有可用的命令。

如果你想了解更多进阶的内容，比如设置命令行补全、打包和发布等等，请阅读 [Poetry 文档](https://poetry.eustace.io/docs/)。

### 准备工作

如果你是在一个已有的项目里使用 Poetry，你只需要执行 poetry init 命令来创建一个 pyproject.toml 文件：

```shell
$ poetry init
```

根据它的提示输入你的项目信息，不确定的内容就按下 Enter 使用默认值，后续也可以手动更新。指定依赖的环节可以跳过，手动安装会更高效一点。

如果你想创建一个新的 Python 项目，使用 poetry new <文件夹名称> 命令可以创建一个项目模板：

```shell
$ poetry new foo
```

这会创建一个这样的项目结构：

```
foo
├── pyproject.toml
├── README.rst
├── foo
│   └── __init__.py
└── tests
    ├── __init__.py
    └── test_foo.py
```

如果你想使用 src 文件夹，可以添加 –src 选项，这会把程序包嵌套在 src 文件夹里。

### 创建虚拟环境

使用 poetry install 命令创建虚拟环境（确保当前目录有 pyproject.toml 文件）：

```shell
$ poetry install
```

这个命令会读取 pyproject.toml 中的所有依赖（包括开发依赖）并安装，如果不想安装开发依赖，可以附加 –no-dev 选项。如果项目根目录有 poetry.lock 文件，会安装这个文件中列出的锁定版本的依赖。如果执行 add/remove 命令的时候没有检测到虚拟环境，也会为当前目录自动创建虚拟环境。

### 激活虚拟环境

执行 poetry 开头的命令并不需要激活虚拟环境，因为它会自动检测到当前虚拟环境。如果你想快速在当前目录对应的虚拟环境中执行命令，可以使用 poetry run <你的命令> 命令，比如：

```shell
$ poetry run python app.py
```

如果你想显式的激活虚拟环境，使用 poetry shell 命令：

```shell
$ poetry shell
```

### 安装包

使用 poetry add 命令来安装一个包：

```shell
$ poetry add flask
```

添加 –dev 参数可以指定为开发依赖：

```shell
$ poetry add pytest --dev
```

### 追踪 & 更新包

使用 poetry show 命令可以查看所有安装的依赖（可以传递包名称作为参数查看具体某个包的信息）：

```shell
$ poetry show
```

添加 –tree 选项可以查看依赖关系：

```shell
$ poetry show --tree
```

添加 –outdated 可以查看可以更新的依赖：

```shell
$ poetry show --outdated
```

执行 poetry update 命令可以更新所有锁定版本的依赖：

```shell
$ poetry update
```

如果你想更新某个指定的依赖，传递包名作为参数：

```shell
$ poetry update foo
```

### 卸载包

使用 poetry remove <包名称> 卸载一个包：

```shell
$ poetry remove foo
```

## 常用配置

Poetry 的配置存储在单独的文件中，比 Pipenv 设置环境变量的方式要方便一点。配置通过 poetry config 命令设置，比如下面的命令可以写入 PyPI 的账号密码信息：

```shell
$ poetry config http-basic.pypi username password
```

下面的命令设置在项目内创建虚拟环境文件夹：

```shell
$ poetry config settings.virtualenvs.in-project true
```

另一个常用的配置是设置 PyPI 镜像源，以使用豆瓣提供的 PyPI 镜像源为例，你需要在 pyproject.toml 文件里加入这部分内容：

```
[[tool.poetry.source]]
name = "douban"
url = "https://pypi.doubanio.com/simple/"
```

[这篇文章](http://greyli.com/set-custom-pypi-mirror-url-for-pip-pipenv-poetry-and-flit/)列出了一些其他国内的 PyPI 源。

## 联动python脚本

Poetry有一点深得我爱，就是可以吧isort、black、pytest等等配置接入到pyproject.toml，然后如下使用：

```shell
poetry run isort backend
poetry run black src
poetry run pytest tests
```

就可以实现统一python脚本的入口，实现代码规范、格式化与开启测试。

一下举例isort和black的配置

### isort

```shell
[tool.isort]
force_grid_wrap = 0
include_trailing_comma = true
line_length = 119
multi_line_output = 3
use_parentheses = true
```



## 总结

总的来说，我愿意深入尝试和使用 Poetry。当然，经过使用 Pipenv 的痛苦经历，我对推荐工具这种事情变得更保守了。所以我不推荐 Python 初学者使用，不推荐直接在生产环境使用，不推荐没法正常访问国际互联网的人使用。

列一些我了解到的优缺点：

优点

- 使用标准的 pyproject.toml 标准，不用写多个配置文件
- 同时支持管理 Python 程序和 Python 库
- 更符合直觉的默认设计，比如不会随便更新锁定版本的依赖
- 干净简洁的命令行输出，没有星星和蛋糕
- 安装包的时候，使用 upper bound 版本限定，而不是 Pipenv 默认的通配符
- 卸载包的时候，直接卸载孤立的子依赖，不需要像 Pipenv 那样需要再执行 pipenv clean

缺点

- 引入新的 [pyproject.toml](https://poetry.eustace.io/docs/pyproject/) 标准，旧项目需要一点迁移成本和学习成本
- 解析依赖的过程有时候会久一点
- 对虚拟环境的管理控制有些弱，没有 Pipenv 那样的删除虚拟环境和清空依赖的操作

总的来说，现在的poetry已经是一个足够强大的python依赖管理工具，最新的项目用它准没错。



------

参考资料：

[Poetry官方文档](https://python-poetry.org/docs)

[相比 Pipenv，Poetry 是一个更好的选择](https://greyli.com/set-pypi-mirror/)
