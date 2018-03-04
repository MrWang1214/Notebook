# Scrapy架构

下图为`Scrapy`的架构，包括组件及在系统中发生的数据流的概览

![Scrapy架构](http://scrapy-chs.readthedocs.io/zh_CN/stable/_images/scrapy_architecture.png)

* `Scrapy Engine` 引擎是`scarpy`架构的核心，负责控制数据流在系统中所有组件中流动，并在相应动作发生时触发事件

* `调度器(Scheduler)` 调度器从引擎接受request并将他们入队，以便之后请求时提供给引擎

* `下载器(Downloader)` 下载器负责获取页面数据并提供给引擎，而后提供给`spider`

* `Spider` 用于分析response并提取`item`或`额外跟进URL`的类。每个spider负责处理一个特定(或一些)网站

* `Item Pipeline` 负责处理被spider提取出来的item。典型的处理有去重、清理、验证及持久化(例如存取到数据库中)，相当于数据的过滤器。

* `下载器中间件(Downloader middlewares)` 在引擎及下载器之间的`specific hook`，处理`Downloader`传递给引擎的response。

* `Spider中间件(Spider middlewares)` 在引擎及Spider之间的`specific hook`，处理spider的`response`和`items`及`requests`。

## 数据流(Data flow)

Scrapy中的数据流由执行引擎控制，其过程如下:

1. 引擎打开一个网站，找到处理该网站的Spider并向该spider请求第一个要爬取的`URL`(s)
2. 引擎从Spider中获取到第一个要爬取的`URL`并在调度器(Scheduler)以`Request`调度
3. 引擎向调度器请求下一个要爬取的`URL`
4. 调度器返回下一个要爬取的`URL`给引擎，引擎将URL通过下载中间件(请求(request)方向)转发给下载器(Downloader)
5. 一旦页面下载完毕，下载器生成一个该页面的Response，并将其通过下载中间件(返回(`response`)方向)发送给引擎。
6. 引擎从下载器中接收到`Response`并通过Spider中间件(输入方向)发送给Spider处理。
7. Spider处理`Response`并返回爬取到的Item及(跟进的)新的Request给引擎。
8. 引擎将(Spider返回的)爬取到的Item给`Item Pipeline`，将(Spider返回的)`Request`给调度器。
9. (返回第二步)重复直到调度器中没有更多地request，引擎关闭该网站。

## 事件驱动网络(Event-driven networking)

Scrapy基于事件驱动网络框架 `Twisted` 编写。因此，Scrapy基于并发性考虑由非阻塞(即`异步`)的实现
