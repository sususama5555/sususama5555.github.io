---
title: Python之全局解释器锁GIL
date: 2020-06-11 00:54:11
tags: 
- Python
- GIL
- 全局解释器锁
- 多线程
categories: 
- Python
- GIL
---

> ## 前言

GIL(Global Interpreter Lock)，也称为全局解释器，官方解释为：

> In CPython, the global interpreter lock, or GIL, is a mutex that prevents multiple native threads from executing Python bytecodes at once. This lock is necessary mainly because CPython’s memory management is not thread-safe. (However, since the GIL exists, other features have grown to depend on the guarantees that it enforces.)

主要意思为：

> GIL是一个互斥锁，它防止多个线程同时执行Python字节码。这个锁是必要的，主要是因为CPython的内存管理不是线程安全的

<!-- more -->

### Python解释器有哪些

1. CPython: 官方默认版本，使用C语言开发，是Python使用最广泛的解释器，有GIL.
2. IPython: IPython是基于CPython之上的交互式解释器，其它方面和CPython相同.
3. PyPy: PyPy采用JIT(Just In Time)也就是即时编译编译器，对Python代码执行动态编译，目的是加快执行速度，有GIL.
4. Jython: 运行在Java平台上的解释器，把Python代码编译为Java字节码执行，没有GIL.
5. IronPython: IronPython和Jython类似，只不过IronPython是运行在微软.Net平台上的Python解释器，可以直接把Python代码编译成.Net的字节码，没有GIL.

## GIL的背景

由于物理上得限制，各CPU厂商在核心频率上的比赛已经被多核所取代。为了更有效的利用多核处理器的性能，就出现了多线程的编程方式，而随之带来的就是线程间数据一致性和状态同步的困难。

Python当然也逃不开，为了利用多核，Python开始支持多线程。而解决多线程之间数据完整性和状态同步的最简单方法自然就是加锁。于是有了GIL这把超级大锁，而当越来越多的代码库开发者接受了这种设定后，他们开始大量依赖这种特性（即默认python内部对象是thread-safe的，无需在实现时考虑额外的内存锁和同步操作）。

## GIL为什么会存在？

GIL的问题其实是由于近十几年来应用程序和操作系统逐步从多任务单核心演进到多任务多核心导致的 , 在一个古老的单核CPU上调度多个线程任务，大家相互共享一个全局锁，谁在CPU执行，谁就占有这把锁，直到这个线程因为IO操作或者Timer Tick到期让出CPU，没有在执行的线程就安静的等待着这把锁（除了等待之外，他们应该也无事可做）。下面这个图演示了一个单核CPU的线程调度方式：

{% asset_img python-gil-single.jpg %}

在一个现代多核心的处理器上，上面的模型就有很大优化空间了，原来只能等待的线程任务，现在可以在其它空闲的核心上调度并发执行。由于古老GIL机制，如果线程2需要在CPU 2 上执行，它需要先等待在CPU 1 上执行的线程1释放GIL（记住：GIL是全局的）。如果线程1是因为 i/o 阻塞让出的GIL，那么线程2必定拿到Gil。但如果线程1是因为timer ticks计数满100让出GIL，那么这个时候线程1和线程2公平竞争。但要命的是，在Python 2.x, 线程1不会动态的调整自身的优先级，所以很大概率下次被选中执行的还是线程1，在很多个这样的选举周期内，线程2只能安静的看着线程1拿着GIL在CPU 1上欢快的执行。

{% asset_img python-gil-muti.jpg %}

在稍微极端一点的情况下，比如线程1使用了while True在CPU 1 上执行，那就真是“一核有难，八核围观”了，如下图所示：

{% asset_img python-gil-octo.jpg %}

## GIL的影响

从上文的介绍和官方的定义来看，GIL无疑就是一把全局排他锁。毫无疑问全局锁的存在会对多线程的效率有不小影响。甚至就几乎等于Python是个单线程的程序。

### GIL对多线程Python程序的影响

程序的性能受到计算密集型(CPU)的程序限制和I/O密集型的程序限制影响，那什么是计算密集型和I/O密集型程序?

#### 计算密集型(CPU)

高度使用CPU的程序，例如: 进行数学计算，矩阵运算，搜索，图像处理等.

#### I/O密集型

I/0(Input/Output)程序是进行数据传输，例如: 文件操作，数据库，网络数据等

### 顺序执行单线程和并发执行多线程的效率对比

#### - 顺序执行单线程(single_thread.py)

```python
import threading
import time

def test_counter():
    i = 0
    for _ in range(100000000):
        i += 1
    return True

def main():
    start_time = time.time()
    for tid in range(2):
        t1 = threading.Thread(target=test_counter)
        t1.start()
        t1.join()
    end_time = time.time()
    print("Total time:{}".format(end_time-start_time))


if __name__ == "__main__":
    main()
```

执行结果:

```css
Total time: 11.299654722213745
```



#### - 并发执行两个线程(multi_thread.py)

```python
import threading
import time

def test_counter():
    i = 0
    for _ in range(100000000):
        i += 1
    return True

def main():
    thread_array = {}
    start_time = time.time()
    for tid in range(2):
        t = threading.Thread(target=test_counter)
        t.start()
        thread_array[tid] = t
    for i in range(2):
        thread_array[i].join()
    end_time = time.time()
    print("Total time:{}".format(end_time-start_time))


if __name__ == "__main__":
    main()
```

执行结果:

```css
Total time:13.7098388671875
```

GIL对I/O绑定多线程程序的性能影响不大，因为线程在等待I/O时共享锁.

GIL对计算型绑定多线程程序有影响，例如: 使用线程处理部分图像的程序，不仅会因锁定而成为单线程，而且还会看到执行时间的增加，这种增加是由锁的获取和释放开销的结果.

## 如何处理Python中的GIL

- 计算密集型程序
  - 使用多进程(什么是多进程呢，后续道来)
  - 使用其它语言(将计算密集程序放到其它语言中执行)
  - 替换解释器(可以自己尝试)
  - 等大神解决GIL
- I/O密集型程序
  - 使用多线程
  - 使用多进程
  - 使用多进程 + 多线程



******

参考链接：

[Understanding the Python GIL](http://www.dabeaz.com/python/UnderstandingGIL.pdf)  

[What Is the Python Global Interpreter Lock (GIL)?](https://realpython.com/python-gil/)

[深入理解Python中的GIL（全局解释器锁）](https://zhuanlan.zhihu.com/p/75780308)