---
title: Python协程光速入门
date: 2018-03-29 20:31:37
tags: [Python,协程]
author: 法狗狗 NLP 数据科学家 - 大头队长
---

Python协程光速入门
==================

`原创作者：法狗狗 NLP 数据科学家 - 大头队长`

### 前言

关于网络IO，同步，异步或者阻塞非阻塞永远是绕不开的话题。对于爬虫这种IO密集型任务，如果使用同步的方式进行网络请求，只要某个请求被阻塞了，则会造成整个流程时间的拖长，极大的降低了爬虫的速度。这篇文章将介绍Python实现异步的其中一种方式，协程。并通过一些实例来说明协程的用法，以及好处。

### 基本知识

* 同步，异步

  上文提到了同步与异步两个术语，那么这两者到底有什么区别呢？拿爬虫场景举个例子，比如说现在爬虫需要点开十个链接，IO过程就是打开这十个链接的过程，CPU负责点击链接的事件。显而易见，点击事件是非常快速的，而链接显示的过程是比较缓慢的。同步IO就是爬虫点击了一个网址，等到获得了彻底响应，才去点击下一个网址。而异步IO就是爬虫点击了一个网址，不等获得响应，立刻点击下一个网址，最后等待响应。两者对比，异步的效率会更加高一些。

* IO密集与计算密集

  IO密集型任务指的是磁盘IO或者网络IO占主要的任务，计算量很小，比如请求网页，读写文件等。计算密集型任务指的是CPU计算占主要的任务，比如图形渲染中矩阵的运算（当然现在都用GPU来完成）。


* 为什么不使用多线程

  一般来说，解决并行事件的传统思路可能是使用多线程。但是多线程有几个劣势，第一是资源的开销，第二是由于Python GIL（全局解释锁）的存在，多线程并非并行执行，而是交替执行，造成多线程在计算密集型任务的效率并不高。

* 协程是什么

  协程的概念比较容易理解，它在单线程中，允许一个执行过程A中断，然后转到执行过程B，在合适的时间又可以转回来，从而在单线程中实现了类似多线程的效果。而它有以下几点优势：

  * 数量理论上可以是无限个，因为是在单线程上进行，没有线程间的切换操作，效率比较高。
  * 不需要“锁”的机制，所有的协程都在一个线程中。
  * 比较容易Debug，因为代码是顺序执行。

上面的内容解释了使用协程的背景，与传统多线程的区别以及协程的概念。接下来用一段代码来详细说明协程工作的过程。



### Python中的协程实现

StackOverflow上曾经有个人问，[如何使用Python以最快的方式发送10000个HTTP请求](https://stackoverflow.com/questions/2632520/what-is-the-fastest-way-to-send-100-000-http-requests-in-python)，其最佳答案是这么实现的：

```python
from urlparse import urlparse
from threading import Thread
import httplib, sys
from Queue import Queue

concurrent = 200

def doWork():
    while True:
        url = q.get()
        status, url = getStatus(url)
        doSomethingWithResult(status, url)
        q.task_done()

def getStatus(ourl):
    try:
        url = urlparse(ourl)
        conn = httplib.HTTPConnection(url.netloc)
        conn.request("HEAD", url.path)
        res = conn.getresponse()
        return res.status, ourl
    except:
        return "error", ourl

def doSomethingWithResult(status, url):
    print status, url

q = Queue(concurrent * 2)
for i in range(concurrent):
    t = Thread(target=doWork)
    t.daemon = True
    t.start()
try:
    for url in open('urllist.txt'):
        q.put(url.strip())
    q.join()
except KeyboardInterrupt:
    sys.exit(1)
```

这段代码的主要思想是，使用了多线程从队列中获取数据，利用IO阻塞等待的时间，执行其它线程，提升了效率。正如上文解释Python的多线程所述，多线程并非同时执行任务而是交替执行。那么是否可以利用协程的方式，只使用一个线程就能完成这个IO任务呢？

我们先来看看Python实现协程的基本原理，Python中的协程是通过生成器的概念实现的，我们来看一个例子：

```python
def consumer():
    print("[Consumer] Init Consumer ......")
    r = "init ok"
    while True:
        # Consumer通过yield拿到消息，又通过yield把结果返回
        n = yield r
        print("[Consumer] conusme n = %s, r = %s" % (n, r))
        r = "consume %s OK" % n

def produce(c):
    print("[Producer] Init Producer ......")
    # 调用c.send(None)启动生成器
    r = c.send(None)
    print("[Producer] Start Consumer, return %s" % r)
    n = 0
    while n < 5:
        n += 1
        print("[Producer] While, Producing %s ......" % n)
        # 一旦生产了东西，通过c.send(n)切换到consumer执行
        r = c.send(n)
        print("[Producer] Consumer return: %s" % r)
    c.close()
    print("[Producer] Close Producer ......")

produce(consumer())
```

上面的例子来自廖雪峰的Python教程，其执行结果如下:

> [Producer] Init Producer ......
> [Consumer] Init Consumer ......
> [Producer] Start Consumer, return init ok
> [Producer] While, Producing 1 ......
> [Consumer] conusme n = 1, r = init ok
> [Producer] Consumer return: consume 1 OK
> [Producer] While, Producing 2 ......
> [Consumer] conusme n = 2, r = consume 1 OK
> [Producer] Consumer return: consume 2 OK
> [Producer] While, Producing 3 ......
> [Consumer] conusme n = 3, r = consume 2 OK
> [Producer] Consumer return: consume 3 OK
> [Producer] While, Producing 4 ......
> [Consumer] conusme n = 4, r = consume 3 OK
> [Producer] Consumer return: consume 4 OK
> [Producer] While, Producing 5 ......
> [Consumer] conusme n = 5, r = consume 4 OK
> [Producer] Consumer return: consume 5 OK
> [Producer] Close Producer ......

可以看出生产者和消费者之间的任务是来回切换的，传统的生产者-消费者模式是一个线程写消息，一个线程取消息，通过锁机制控制队列和等待，有可能发生死锁的情况。而协程在生产者生产完消息后，直接通过`yield`跳转到消费者开始执行，待消费者执行完毕后，切换回生产者继续生产，效率很高。

针对网络请求IO的场景，我们可以做个实验来比较一下同步IO与异步IO的速度，

```python
import requests
import time


def consumer():
    r = 'https://baidu.com'
    while True:
        n = yield requests.get(r).status_code
        if not n:
            return
        print('[CONSUMER] Consuming %s...' % n)


def produce(c):
    c.send(None)
    for i in range(73000, 73100):
        request_url = "https://www.jb51.net/article/%d.htm" % i
        print('[PRODUCER] Producing %s...' % request_url)
        r = c.send(request_url)
        print('[PRODUCER] Consumer return: %s' % r)
    c.close()


async_start = time.time()
produce(consumer())
print(time.time() - async_start)

sync_start = time.time()
for i in range(73000, 73100):
    url = "https://www.jb51.net/article/%d.htm" % i
    response = requests.get(url)
print(time.time() - sync_start)

```

运行了一次的最后输出结果，通过协程实现的异步IO9.8秒就完成了请求任务，而同步IO使用了28.6秒。可以看出异步IO的效率确实有了很大提升。

### 写在之后

上面的文字简单的介绍了一下协程的基本特性以及如何使用Python实现协程从而达到异步IO的效果。关于网络IO还有很多可以讨论的东西，这里给出一个学习路线，

* 有兴趣可以去看看Python3对于异步IO的支持，比如aiohttp这些库，极大的降低了编码难度。
* 另外像Tornado这类支持异步非阻塞的Web框架也是特别有意思的，这次写问答系统的爬虫Web服务就是基于这个库。
* 如果还想继续深入了解，建议去看看《Linux/Unix系统编程手册》关于socket的几章，对于理解IO机制的底层原理（比如epoll，select）特别有帮助（我其实没看过，以后有时间会继续研究）。
