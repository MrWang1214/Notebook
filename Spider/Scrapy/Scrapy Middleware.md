# Scrapy Middleware

## Scrapy Downloader Middleware

下载器中间件是介于Scrapy的`request/response`处理的钩子框架。 是用于`全局`修改Scrapy request和response的一个轻量、底层的系统

### 激活下载器中间件

要激活下载器中间件组件，将其加入到 `DOWNLOADER_MIDDLEWARES` 设置中。 该设置是一个字典(dict)，键为中间件类的路径，值为其中间件的顺序(order)

`DOWNLOADER_MIDDLEWARES` 设置会与Scrapy定义的 `DOWNLOADER_MIDDLEWARES_BASE` 设置合并(`但不是覆盖`)， 而后根据顺序(order)进行排序

* 第一个中间件是最靠近引擎的，最后一个中间件是最靠近下载器的

* 中间件可能会依赖于之前(或者之后)执行的中间件，因此顺序是很重要的

* 禁止内置的(在 `DOWNLOADER_MIDDLEWARES_BASE` 中设置并默认启用的)中间件，必须在项目的 `DOWNLOADER_MIDDLEWARES` 设置中定义该中间件，并将其值赋为 None

  ```python
  DOWNLOADER_MIDDLEWARES = {
    'myproject.middlewares.CustomDownloaderMiddleware': 543,
    'scrapy.downloadermiddlewares.useragent.UserAgentMiddleware': None,
    }
  ```

### 编写Downloader Middleware

每个中间件组件是一个定义了`以下一个或多个方法`的Python类:

`class scrapy.downloadermiddlewares.DownloaderMiddleware`

* `process_request(request, spider)`

  * 当每个request通过下载中间件时，该方法被调用

  * `process_request()` 必须返回其中之一: 返回 `None` 、返回一个 `Response` 对象、返回一个 `Request` 对象或`raise IgnoreRequest`

    * 如果其返回 `None` ，Scrapy将继续处理该request，执行其他的中间件的相应方法，直到合适的下载器处理函数(`download handler`)被调用， 该request被执行(其response被下载)

    * 如果其返回 `Response` 对象，Scrapy将不会调用任何其他的 `process_request()` 或 `process_exception()` 方法，或相应地下载函数；其将返回该response。 已安装的中间件的 `process_response()` 方法则会在每个response返回时被调用

    * 如果其返回 `Request` 对象，Scrapy则停止调用`process_request`方法并重新调度返回的request。当新返回的request被执行后， 相应地中间件链将会根据下载的response被调用

    * 如果其raise一个`IgnoreRequest`异常，则安装的下载中间件的`process_exception()`方法会被调用。如果没有任何一个方法处理该异常， 则request的`errback(Request.errback)`方法会被调用。如果没有代码处理抛出的异常，则该异常被忽略且不记录(不同于其他异常那样)

    参数:
      * `request` (Request 对象) – 处理的request
      * `spider` (Spider 对象) – 该request对应的spider

* `process_response(request, response, spider)`

  * `process_request()` 必须返回以下之一: 返回一个`Response`对象、 返回一个`Request`对象或raise一个 `IgnoreRequest` 异常

    * 如果其返回一个 `Response` (可以与传入的response相同，也可以是全新的对象)， 该response会被在链中的其他中间件的 process_response() 方法处理

    * 如果其返回一个 `Request` 对象，则中间件链停止， 返回的request会被重新调度下载。处理类似于 `process_request()` 返回request所做的那样

    * 如果其抛出一个 `IgnoreRequest` 异常，则调用request的`errback(Request.errback)`。如果没有代码处理抛出的异常，则该异常被忽略且不记录(不同于其他异常那样)

    参数:
      * `request` (Request 对象) – response所对应的request
      * `response` (Response 对象) – 被处理的response
      * `spider` (Spider 对象) – response所对应的spider

* `process_exception(request, exception, spider)`

  当下载处理器(download handler)或 `process_request()` (下载中间件)抛出异常(包括 `IgnoreRequest` 异常)时， Scrapy调用 process_exception() 。

  * `process_exception()` 应该返回以下之一: 返回 `None` 、 一个 `Response` 对象、或者一个 `Request` 对象。

    * 如果其返回 `None`，Scrapy将会继续处理该异常，接着调用已安装的其他中间件的 `process_exception()` 方法，直到所有中间件都被调用完毕，则调用默认的异常处理。

    * 如果其返回一个 `Response` 对象，则已安装的中间件链的 `process_response()` 方法被调用。Scrapy将不会调用任何其他中间件的 `process_exception()` 方法。

    * 如果其返回一个 `Request` 对象， 则返回的request将会被重新调用下载。这将停止中间件的 `process_exception()` 方法执行，就如返回一个response的那样。

    参数:
      * `request` (是 Request 对象) – 产生异常的request
      * `exception` (Exception 对象) – 抛出的异常
      * `spider` (Spider 对象) – request对应的spider

## Spider中间件(Middleware)

`Spider中间件`是介入到Scrapy的spider处理机制的钩子框架，可以添加代码来处理发送给 `Spiders` 的response及spider产生的`item`和`request`

### 激活spider中间件

要启用`spider中间件`，可以将其加入到 `SPIDER_MIDDLEWARES` 设置中。 该设置是一个字典，键位中间件的路径，值为中间件的顺序(order)。具体设置的方法和`Downloader Middleware`基本上相同

### 编写spider Middleware

* `class scrapy.contrib.spidermiddleware.SpiderMiddleware`

  * `process_spider_input(response, spider)`

  当response通过spider中间件时，该方法被调用，处理该`response`。`process_spider_input()` 应该返回 None 或者抛出一个异常。

  * 如果其返回 `None` ，Scrapy将会继续处理该response，调用所有其他的中间件直到spider处理该response

  * 如果其跑出一个异常(exception)，Scrapy将不会调用任何其他中间件的 `process_spider_input()` 方法，并调用request的errback。 errback的输出将会以另一个方向被重新输入到中间件链中，使用 process_spider_output() 方法来处理，当其抛出异常时则带调用 `process_spider_exception()`

    参数:
    * `response` (Response 对象) – 被处理的response
    * `spider` (Spider 对象) – 该response对应的spider

* `process_spider_output(response, result, spider)`

  当Spider处理response返回result时，该方法被调用。`process_spider_output()` 必须返回包含 Request 、dict 或 Item 对象的可迭代对象(iterable)。

  参数:
  * `response` (Response 对象) – 生成该输出的response
  * `result` (包含 Request 、dict 或 Item 对象的可迭代对象(iterable)) – spider返回的result
  * `spider` (Spider 对象) – 其结果被处理的spider

* `process_spider_exception(response, exception, spider)`

  当spider或(其他spider中间件的) process_spider_input() 跑出异常时， 该方法被调用。`process_spider_exception()` 必须要么返回 `None` ， 返回一个包含 `Response` 、`dict`或 `Item` 对象的可迭代对象(iterable)。

  * 如果其返回 `None` ，Scrapy将继续处理该异常，调用中间件链中的其他中间件的 `process_spider_exception()` 方法，直到所有中间件都被调用，该异常到达引擎(异常将被记录并被忽略)。

  * 如果其返回一个可迭代对象，则中间件链的 `process_spider_output()` 方法被调用， 其他的 `process_spider_exception()` 将不会被调用。

    参数:
    * response (Response 对象) – 异常被抛出时被处理的response
    * exception (Exception 对象) – 被跑出的异常
    * spider (Spider 对象) – 抛出该异常的spider

* process_start_requests(start_requests, spider)

  该方法以spider 启动的request为参数被调用，执行的过程类似于 `process_spider_output()` ，只不过其没有相关联的response并且必须返回request(不是item)。其接受一个可迭代的对象(`start_requests` 参数)且必须返回另一个包含 `Request` 对象的可迭代对象

  参数:
  * `start_requests` (包含 Request 的可迭代对象) – start requests
  * `spider` (Spider 对象) – start requests所属的spider