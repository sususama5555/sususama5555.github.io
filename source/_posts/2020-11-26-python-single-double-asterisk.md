---
title: Python函数中参数 * 和 ** 的区别
date: 2019-11-26 00:28:55
tags: 
- Python
- 参数
categories: 
- Python
- 基础原理
---

在 Python 的函数中经常能看到输入的参数前面有一个或者两个星号，例如：

```python
def foo(param1, *param2):
def bar(param1, **param2):
```

这两种用法其实都是用来将任意个数的参数导入到 Python 函数中。

<!-- more -->

### **单星号（\*）：\*agrs**

将所有参数以`元组(tuple)`的形式导入：

#### **实例**

```python
def foo(param1, *param2):
    print (param1)
    print (param2)
foo(1,2,3,4,5)
```

以上代码输出结果为：

```python
1
(2, 3, 4, 5)
```

### **双星号（\*）：\*kwargs**

双星号（**）将参数以`字典(dict)`的形式导入:

#### **实例**

```python
def bar(param1, **param2):
    print (param1)
    print (param2)
bar(1,a=2,b=3)
```

以上代码输出结果为：

```
1
{'a': 2, 'b': 3}
```

此外，单星号的另一个用法是解压参数列表：

#### **实例**

```python
def foo(runoob_1, runoob_2):
    print(runoob_1, runoob_2)
l = [1, 2]
foo(*l)
```

以上代码输出结果为：

```python
1 2
```

当然这两个用法可以同时出现在一个函数中：

#### **实例**

```python
def foo(a, b=10, *args, **kwargs):
    print (a)
    print (b)
    print (args)
    print (kwargs)
foo(1, 2, 3, 4, e=5, f=6, g=7)
```

以上代码输出结果为：

```python
1
2
(3, 4)
{'e': 5, 'f': 6, 'g': 7}
```

