---
layout:     post
title:      "tornado——异步请求"
subtitle:   "使用gen.coroutine和concurrent.futures使请求异步"
date:       2017-03-10 12:00:00
author:     "ZWY"
header-img: "img/in-post/3-10/banner.jpg"
catalog: true
tags:
    - tornado
    - web框架
    - 服务器
---
# tornado框架

Tornado是使用Python编写的一个非阻塞Web服务器，也是一个轻量级框架。其基于EPOLL，所以可以非阻塞的就解决C10K的问题。
>Web服务器与Web框架、同步与异步、C10K问题 的相关概念，会在下一篇文章整理写出来

<!-- more -->

<br> 网上看到一张tornado的框架图，画得很好：
![img](/img/in-post/3-10/3-10-1.png)

**主要模块**
* **web** - FriendFeed 使用的基础 Web 框架，包含了 Tornado 的大多数重要的功能
* **escape** - XHTML, JSON, URL 的编码/解码方法
* **database** - 对 MySQLdb 的简单封装，使其更容易使用
* **template** - 基于 Python 的 web 模板系统
* **httpclient** - 非阻塞式 HTTP 客户端，它被设计用来和 web 及 httpserver 协同工作
* **auth** - 第三方认证的实现（包括 Google OpenID/OAuth、Facebook Platform、Yahoo BBAuth、FriendFeed OpenID/OAuth、Twitter OAuth）
* **locale** - 针对本地化和翻译的支持
* **options** - 命令行和配置文件解析工具，针对服务器环境做了优化
<br> **底层模块**
* **httpserver** - 服务于 web 模块的一个非常简单的 HTTP 服务器的实现
* **iostream** - 对非阻塞式的 socket 的简单封装，以方便常用读写操作
* **ioloop** - 核心的 I/O 循环

# tornado的同步
Tornado虽然是一个异步框架，但是如果使用不当很容易造成性能低下。首先实现一个同步的最小服务HTTP的服务器：
```
import tornado.httpserver                         
import tornado.ioloop
import tornado.web
import time

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello world!")

class SleepHandler(tornado.web.RequestHandler):
    def get(self):
        time.sleep(10)
        self.write("Sleep!")

requestHandlers=[(r'/',MainHandler),
                 (r'/sleep',SleepHandler)]

if __name__ == "__main__":
    app=tornado.web.Application(requestHandlers)
    app.listen(9998)
    tornado.ioloop.IOLoop.instance().start()
```
因为tornado本身是单线程的，所以当你调用 sleep 接口后会发现，整个服务会阻塞，直到这个耗时的任务处理完成才能响应接下来的请求，服务器的性能大幅度的降低。

# gen.coroutine的异步
在Tornado中有两个装饰器：
* tornado.web.asynchronous
* tornado.gen.coroutine
<br> asynchronous 装饰器是让请求变成长连接的方式，必须手动调用 self.finish() 才会响应。
<br> coroutine 装饰器是指定改请求为协程模式。
<br> 加入这两个装饰器：
```
class SleepHandler(tornado.web.RequestHandler):
    @tornado.web.asynchronous
    @tornado.gen.coroutine
    def get(self):
        yield tornado.gen.sleep(10)
        self.write("Sleep!")
        self.finish()
```

实际上gen.coroutine 在 Tornado 3.1 后会自动调用 self.finish() 结束请求，所以可以不使用 asynchronous 装饰器。

```
class SleepHandler(tornado.web.RequestHandler):
    @tornado.gen.coroutine
    def get(self):
        yield tornado.gen.sleep(10)
        self.write("Sleep!")
```
可以注意到此处我们将`time.sleep()` 修改为 `tornado.gen.sleep()`，因为使用 gen.coroutine 装饰器编写异步函数，如果库本身不支持异步，那么响应任然是阻塞的。所以这种实现异步非阻塞的方式需要依赖大量的基于 Tornado 协议的异步库，使用上比较局限。所以我们再使用另一个方法解决这个问题。

# concurrent.futures的异步
concurrent.futures 是python3新增加的一个库，用于并发处理。其中 ThreadPoolExecutor 是对标准库中的 threading 的高度封装，利用线程的方式让阻塞函数异步化，解决了很多库是不支持异步的问题。在 Tornado 中有个装饰器能使用 ThreadPoolExecutor 来让阻塞过程编程非阻塞，其原理是在 Tornado 本身这个线程之外另外启动一个线程来执行阻塞的程序。

```
import tornado.httpserver
import tornado.ioloop
import tornado.web
import tornado.gen
import time
import threading
from tornado.concurrent import run_on_executor
from concurrent.futures import ThreadPoolExecutor

class MainHandler(tornado.web.RequestHandler):
    def get(self):
        self.write("Hello world!")

class SleepHandler(tornado.web.RequestHandler):
    executor = ThreadPoolExecutor(2)

    @tornado.gen.coroutine
    def get(self):
        yield self.sleep(10)
        self.write("Sleep!")

    @run_on_executor
    def sleep(self,sec):
        time.sleep(sec)

def activeth():
    while True:
        print(threading.active_count())
        time.sleep(1)

requestHandlers=[(r'/',MainHandler),
                 (r'/sleep',SleepHandler)]

if __name__ == "__main__":
    acc = threading.Thread(target = activeth)
    acc.start()
    app=tornado.web.Application(requestHandlers)
    app.listen(9998)
    tornado.ioloop.IOLoop.instance().start()
```

# 总结
gen.coroutine用法方便，消耗小，但需要异步库的支持。concurrent.futures则是不需要异步库的支持，但是线程的消耗很大。
<br> 还有一种就是利用celery。使用线程和 Celery 的模式进行异步编程，轻量级的放在线程中执行，复杂的放在 Celery 中执行。当然如果有异步库使用那最好不过了。