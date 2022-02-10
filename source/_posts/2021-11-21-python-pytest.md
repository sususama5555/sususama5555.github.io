---
title: Pytest自动化测试框架
date: 2021-11-21 01:07:42
tags: 
- Python
- 测试
- TDD
- Pytest
categories: 
- Python
- 测试
- Pytest
---

## Pytest

> The `pytest` framework makes it easy to write small, readable tests, and can scale to support complex functional testing for applications and libraries.

这是Pytest官网对其的描述，毋庸置疑，pytest已经是最好的python测试框架。

现代软件开发推崇`TDD`(Test-Driven Development) —— 测试驱动开发，所以pytest可以说是python后端的必要技术栈。

由于最近的项目对代码质量要求较高，当然对测试代码也需要很高的完成度，首先来讲讲如何使用pytest开展工作，后面再补上Pytest在Django和DRF中的最佳实践。

<!-- more -->

## Pytest和Unittest测试框架的区别？

如何区分这两者：很简单，unittest作为官方的测试框架，在测试方面更加基础，并且可以再次基础上进行二次开发，同时在用法上格式会更加复杂；而pytest框架作为第三方框架，方便的地方就在于使用更加灵活，并且能够对原有unittest风格的测试用例有很好的兼容性，同时在扩展上更加丰富，可通过扩展的插件增加使用的场景，比如一些并发测试等；

## Pytest 安装

pip安装：

```shell
pip install pytest
```

pip是个好文明

## Pytest 示例

Pytest编写规则:

- 测试文件以test_开头（以_test为结尾）
- 测试的类以Test开头；
- 测试的方法以test_开头
- 断言使用基本的assert

test_example.py

```python
def count_num(a: list) -> int:
    return len(a)


def test_count():
    assert count_num([1, 2, 3]) != 3
```

执行测试：

```bash
pytest test_example.py
```

执行结果：

```bash
C:\Users\libuliduobuqiuqiu\Desktop\GitProjects\PythonDemo\pytest>pytest test_example.py -v
================================================================= test session starts =================================================================
platform win32 -- Python 3.6.8, pytest-6.2.5, py-1.10.0, pluggy-1.0.0 -- d:\coding\python3.6\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\libuliduobuqiuqiu\Desktop\GitProjects\PythonDemo\pytest
plugins: Faker-8.11.0
collected 1 item                                                                                                                                       

test_example.py::test_count FAILED                                                                                                               [100%]

====================================================================== FAILURES =======================================================================
_____________________________________________________________________ test_count ______________________________________________________________________

    def test_count():
>       assert count_num([1, 2, 3]) != 3
E       assert 3 != 3
E        +  where 3 = count_num([1, 2, 3])

test_example.py:11: AssertionError
=============================================================== short test summary info ===============================================================
FAILED test_example.py::test_count - assert 3 != 3
================================================================== 1 failed in 0.16s ==================================================================
复制代码
```

备注：

- .代表测试通过，F代表测试失败；
- -v显示详细的测试信息， -h显示pytest命令详细的帮助信息；

## 标记

> 默认情况下，pytest会在当前目录下寻找以test_为开头（以_test结尾）的测试文件，并且执行文件内所有以test_为开头（以_test为结尾）的所有函数和方法；

1.指定运行测试用例，可以通过::显示标记（文件名::类名::方法名）（文件名::函数名）

```shell
pytest test_example3.py::test_odd
```

2.指定一些测试用例测试运行，可以使用-k模糊匹配

```
pytest -k example
```

3.通过pytest.mark.skip()或者pytest.makr.skipif()条件表达式，跳过指定的测试用例

```python
import pytest

test_flag = False

@pytest.mark.skip()
def test_odd():
    num = random.randint(0, 100)
    assert num % 2 == 1


@pytest.mark.skipif(test_flag is False, reason="test_flag is False")
def test_even():
    num = random.randint(0, 1000)
    assert num % 2 == 0
```

4.通过pytest.raises()捕获测试用例可能抛出的异常

```python
def test_zero():
    num = 0
    with pytest.raises(ZeroDivisionError) as e:
        num = 1/0
    exc_msg = e.value.args[0]
    print(exc_msg)
    assert num == 0
```

5.预先知道测试用例会失败，但是不想跳过，需要显示提示信息，使用pytest.mark.xfail()

```python
@pytest.mark.xfail()
def test_sum():
    random_list = [random.randint(0, 100)  for x in range(10)]
    num = sum(random_list)
    assert num < 20
```

6.对测试用例进行多组数据测试，每组参数都能够独立执行一次（可以避免测试用例内部执行单组数据测试不通过后停止测试）

```python
@pytest.mark.parametrize('num,num2', [(1,2),(3,4)])
def test_many_odd(num: int, num2: int):
    assert num % 2 == 1
    assert num2 % 2 == 0
```

## 固件（Fixture)

> 固件就是一些预处理的函数，pytest会在执行测试函数前（或者执行后）加载运行这些固件，常见的应用场景就有数据库的连接和关闭（设备连接和关闭）

简单使用

```python
import pytest


@pytest.fixture()
def postcode():
    return "hello"


def test_count(postcode):
    assert postcode == "hello"
```

> 按照官方的解释就是当运行测试函数，会首先检测运行函数的参数，搜索与参数同名的fixture，一旦pytest找到，就会运行这些固件，获取这些固件的返回值（如果有），并将这些返回值作为参数传递给测试函数；

## conftest文件

### 为了复用固件

我们可以定义`conftest.py`文件，这个可以理解成框架的固定写法，把`hook`或者`fixture`写在这个文件里，就会自动去调用。

`conftest.py`通常可以用来做2个事情：

- 存放你的fixture函数

- 在里面写自己的本地插件

另外，可以根据`conftest.py`的文件存放位置，来决定它的适用范围。

有了conftest，我们对固件的复用就方便了许多，当一个`fixture`函数在很多case模块中都要用到，这时候你要怎么引用呢？

没关系，我们可以把这个`fixture`函数放到`conftest.py`中去。

### 作用域

总结来说，`conftest.py`文件可以作用于`同级`以及 `以下`的模块。
**但是**，当以下层级中也存在了`另一个conftest.py`，那么以下层级将由`另一个conftest.py`文件接管。

下面这个例子中，conftest与test_case1、test_case2在同级目录，所以这两个文件可以直接复用其中的fixture函数

```
 -demo_project
    -test_case
        -test_module_01
            /* test_case1.py
            /* test_case2.py
            /* conftest.py     -----> conftest在这
        -test_module_02
            /* test_case3.py
    -utils
```

如果 test_module_02/test_case3 也想使用该conftest，那就可以把它在往外提一层。

### 预处理和后处理

接下来进一步验证关于官方的说法：

```python
import pytest


@pytest.fixture()
def connect_db():
    print("Connect Database in .......")
    yield
    print("Close Database out .......")


def read_database(key: str):
    p_info = {
        "name": "zhangsan",
        "address": "China Guangzhou",
        "age": 99
    }
    return p_info[key]


def test_count(connect_db):
    assert read_database("name") == "zhangsan"
```

执行测试函数结果：

```bash
============================= test session starts =============================
platform win32 -- Python 3.6.8, pytest-6.2.5, py-1.10.0, pluggy-1.0.0 -- D:\Coding\Python3.6\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\libuliduobuqiuqiu\Desktop\GitProjects\PythonDemo\pytest
plugins: Faker-8.11.0
collecting ... collected 1 item

test_example.py::test_count Connect Database in .......
PASSED                                       [100%]Close Database out .......


============================== 1 passed in 0.07s ==============================
复制代码
```

备注：

- 首先从结果上看验证了官方的解释，pytest执行测试函数前会寻找同名的固件加载运行；
- connect_db固件中有yield，这里pytest默认会判断yield关键词之前的代码属于预处理，会在测试前执行，yield之后的代码则是属于后处理，将在测试后执行；

### 作用域

> 从前面大致了解了固件的作用，抽离出一些重复的工作方便复用，同时pytest框架中为了更加精细化控制固件，会使用作用域来进行指定固件的使用范围，（比如在这一模块中的测试函数执行一次即可，不需要模块中的函数重复执行）更加具体的例子就是数据库的连接，这一连接的操作可能是耗时的，我只需要在这一模块的测试函数运行一次即可，不需要每次都运行。

而定义固件是，一般通过scop参数来声明作用，常用的有：

- function: 函数级，每个测试函数都会执行一次固件；
- class: 类级别，每个测试类执行一次，所有方法都可以使用；
- module: 模块级，每个模块执行一次，模块内函数和方法都可使用；
- session: 会话级，一次测试只执行一次，所有被找到的函数和方法都可用。

```python
import pytest


@pytest.fixture(scope="function")
def func_scope():
    print("func_scope")


@pytest.fixture(scope="module")
def mod_scope():
    print("mod_scope")


@pytest.fixture(scope="session")
def sess_scope():
    print("session_scope")


def test_scope(sess_scope, mod_scope, func_scope):
    pass


def test_scope2(sess_scope, mod_scope, func_scope):
    pass
```

执行结果：

```bash
============================= test session starts =============================
platform win32 -- Python 3.6.8, pytest-6.2.5, py-1.10.0, pluggy-1.0.0 -- D:\Coding\Python3.6\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\libuliduobuqiuqiu\Desktop\GitProjects\PythonDemo\pytest
plugins: Faker-8.11.0
collecting ... collected 2 items

test_example2.py::test_scope session_scope
mod_scope
func_scope
PASSED                                      [ 50%]
test_example2.py::test_scope2 func_scope
PASSED                                     [100%]

============================== 2 passed in 0.07s ==============================
复制代码
```

> 从这里可以看出module，session作用域的固件只执行了一次，可以验证官方的使用介绍

### 自动执行

> 有人可能会说，这样子怎么那么麻烦，unittest框架中直接定义setUp就能自动执行预处理，同样的pytest框架也有类似的自动执行；  pytest框架中固件一般通过参数autouse控制自动运行。

```python
import pytest


@pytest.fixture(scope='session', autouse=True)
def connect_db():
   print("Connect Database in .......")
   yield
   print("Close Database out .......")


def test1():
   print("test1")


def test2():
   print("test")
复制代码
```

执行结果：

```bash
============================= test session starts =============================
platform win32 -- Python 3.6.8, pytest-6.2.5, py-1.10.0, pluggy-1.0.0 -- D:\Coding\Python3.6\python.exe
cachedir: .pytest_cache
rootdir: C:\Users\libuliduobuqiuqiu\Desktop\GitProjects\PythonDemo\pytest
plugins: Faker-8.11.0
collecting ... collected 2 items

test_example.py::test1 Connect Database in .......
PASSED                                            [ 50%]test1

test_example.py::test2 PASSED                                            [100%]test
Close Database out .......


============================== 2 passed in 0.07s ==============================
复制代码
```

> 从结果看到，测试函数运行前后自动执行了connect_db固件；

### 参数化

> 前面简单的提到过了@pytest.mark.parametrize通过参数化测试，而关于固件传入参数时则需要通过pytest框架中内置的固件request，并且通过request.param获取参数

```python
import pytest


@pytest.fixture(params=[
    ('redis', '6379'),
    ('elasticsearch', '9200')
])
def param(request):
    return request.param


@pytest.fixture(autouse=True)
def db(param):
    print('\nSucceed to connect %s:%s' % param)

    yield

    print('\nSucceed to close %s:%s' % param)


def test_api():
    assert 1 == 1
```

> 这里模拟连接redis和elasticsearch，加载固件自动执行连接然后执行测试函数再断开连接。

## 总结

 		对于开发来说，为什么也要学习自动化测试这一块，很重要的一点就是通过自动化测试节省一些重复工作的时间，同时对于优化代码结构，提高代码覆盖率，以及后续项目重构都是有着很重要的意义，同时理解pytest和unittest在基础上有何区别有助于不同的业务场景中选择适合自己的测试工具。
		 这篇文章只是简单的介绍了pytest的基本使用，有兴趣的可以去看看官方文档，官方文档中还提到了如内置固件的使用，常用测试的场景等等。



------

参考资料：

[pytest官方文档: helps you write better programs — pytest documentation](https://docs.pytest.org/en/latest/)

[Pytest 自动化测试框架 - 掘金 (juejin.cn)](https://juejin.cn/post/7013949685992259591)

