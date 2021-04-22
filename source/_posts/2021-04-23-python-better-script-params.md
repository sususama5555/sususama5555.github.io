---
title: python - 脚本传参最佳实践
date: 2021-04-23 00:57:06
tags: 
- Python 
- 编码优化
- 最佳实践
- 脚本
categories: 
- Python 
- 脚本
---
## 前言

命令行模式在Python开发中并不陌生，简单的说就是python hello_world.py这种使用命令的模式运行Python程序。目前比较主流的命令行工具主要有以下几项，

- 内置的sys
- argparse
- tensorflow的Flags

当然，还有例如Fire、Docopt等工具，这里不一一展开。

本文的主角是**Click**，因为它真的很强大、很好用。以下概括性的讲解一下上述三款命令行工具，以及重头戏**Click**。

<!--more-->

## sys

sys和os一样，是Python自带且非常常用的模块，该模块主要作用就是获取和Python解释器相关的一些信息，它的一个常见用途就是获取命令行参数，

```python
import sys

def hello_world():
    print(sys.argv[1], sys.argv[2])

hello_world()

# 输出
$ python test.py 1 2
>>> 1 2
```

sys通过argv来获取命令行参数，其中argv[0]获取的是脚本的名称，从argv[1]开始获取的是命令行传入的参数。

## argparse
argparse是用的非常多的一种命令行工具，它支持选项命名，指定数据类型，添加帮助信息，设置默认参数，功能非常全面而强大，因此非常受欢迎，

```python
import argparse

arg = argparse.ArgumentParser("This is a test!")

def main(arg):
    print(arg.name)
    print(arg.age)

if __name__ == '__main__':
    arg.add_argument("--name", "-n", default="China", type=str, help="the name of your country.")
    arg.add_argument("--age", "-a", default=25, type=int, help="your age.")
    args = arg.parse_args()
    main(args)
```

查看帮助，

```shell
# 输出
$ python test.py --help
usage: This is a test! [-h] [--name NAME] [--age AGE]
​
optional arguments:
  -h, --help            show this help message and exit
  --name NAME, -n NAME  the name of your country.
  --age AGE, -a AGE     your age.
```
可以输入对应的参数，

```shell
$ python test.py --name Chinese --age 26
Chinese
26
```

也可以使用简写在命令行传入参数，
```
$ python test.py -n Chinese -a 26
Chinese
26
```

## FLAGS
做深度学习相关方向，尤其经常使用tensorflow的应该对这个命令行工具比较熟悉，FLAGS是tensorflow提供的一款命令行工具，和大多数命令行工具大同小异，

```python
import tensorflow as tf

FLAGS = tf.flags.FLAGS

tf.flags.DEFINE_string("train_path", '/voc2012/JPEG', "training file path")
tf.flags.DEFINE_integer("batch_size", 64, "training batch size")
tf.flags.DEFINE_float("lr", 0.01, "learning rate")

def main(_):
    print(FLAGS.train_path)
    print(FLAGS.batch_size)
    print(FLAGS.lr)


if __name__ == '__main__':
    tf.app.run()
```

查看帮助，
```shell
$ python test.py --help

# 输出
       USAGE: test.py [flags]
flags:

test.py:
  --batch_size: training batch size
    (default: '64')
    (an integer)
  --lr: learning rate
    (default: '0.01')
    (a number)
  --train_path: training file path
    (default: '/voc2012/JPEG')

Try --helpfull to get a list of all flags.
```

命令行传入参数，
```shell
$ python test.py --train_path /home/li/images --batch_size 128 --lr 0.001

# 输出
/home/li/images
128
0.001
```

## Click
下面要介绍的就是本文的主角Click，这款工具是用flask的开发团队pallets进行开发，目前在github已经7.6k+star，受欢迎程度可见一斑，

Click的开发初衷就是使用最少的代码，以一种可组合的方式创建漂亮的命令行接口。它的目的是使编写命令行工具的过程快速而有趣，同时防止由于无法实现预期的CLI API而导致的任何问题。

Click主要有以下3个亮点：

- 命令的任意嵌套
- 自动帮助页面生成
- 支持在运行时延迟加载子命令

### 安装

`$ pip install click`
Click支持Python 3.4及以上版本、Python 2.7和PyPy。
首先先来一个例子，

```python
import click


@click.command()
@click.option("--name", default="li", help="your name")
@click.option("--age", default=26, help="your age")
def hello_world(name, age):
    click.echo(name)
    print(age)
    
hello_world()
```


命令行调用，

```shell
$ python test.py --name ll --age 27

# 输出
ll
27
```

可以看出，上述主要用了click的3个方法，分别是，
- command
- option
- echo

这3个方法在Click工具中至关重要，除此之外还有其他的方法，它们的功能分别是，

### 方法功能

- command：用于装饰一个函数，使得该函数作为命令行的接口，例如上述装饰hello_world
- option：用于装饰一个函数，主要功能是为命令行添加选项
- echo：用于输出结果，由于print函数在2.x和3.x之间存在不同之处，为了更好的兼容性，因此提供了echo输出方法
- Choice：输入为一个列表，列表中为选项可选择的值

把上述程序的帮助信息输出，

```shell
$ python test.py --help

Usage: test.py [OPTIONS]

Options:
  --name TEXT    your name
  --age INTEGER  your age
  --help         Show this message and exit.
```

在示例程序中，对于option只使用了default、help两个属性，除此之外option还有其他的属性选项，它们的功能如下，

### 属性描述

- default：给命令行选项添加默认值
- help：给命令行选项添加帮助信息
- type：指定参数的数据类型，例如int、str、float
- required：是否为必填选项，True为必填，False为非必填
- prompt：在命令行提示用户输入对应选项的信息
- nargs：指定命令行选项接收参数的个数，如果超过则会报错
除此之外，Click还提供了group方法，该方法可以添加多个子命令，

```python
import click

@click.group()
def first():
    print("hello world")

@click.command()
@click.option("--name", help="the name")
def second(name):
    print("this is second: {}".format(name))

@click.command()
def third():
    print("this is third")

first.add_command(second)
first.add_command(third)

first()
```

调用子命令second，
```shell
$ python test.py second --name hh

# 输出
hello world
this is second: hh
```

### 一个简单的click例子



```python
import click


@click.command()
@click.option('--count', default=1, help='Number of greetings.')
@click.option('--name', prompt='Your name',
              help='The person to greet.')
@click.option('--life', prompt='life leave',
              help='how many your life.')
def hello(count, name, life):
    """Simple program that greets NAME for a total of COUNT times."""
    for x in range(count):
        click.echo('Hello %s!' % name)
        click.echo('life %s!' % life)


if __name__ == '__main__':
    hello()
```

