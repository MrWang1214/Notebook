# Python爬虫框架Scrapy (二)

1. 创建一个Scrapy项目
2. 定义提取的Item
3. 编写爬取网站的 spider 并提取 Item
4. 编写 Item Pipeline 来存储提取到的Item(即数据)

## 创建一个Scrapy项目

`scrapy startproject CapSpider`

* Scrapy项目结构

```python
CapSpider/
    ├── scrapy.cfg  //项目的配置信息，主要为Scrapy命令行工具提供一个基础的配置信息
    ├── tutorial/
            └── __init__.py
            ├── items.py       //设置数据存储模板，用于结构化数据
            ├── pipelines.py   //数据处理行为
            ├── settings.py    //配置文件
            └── spiders/       //爬虫目录
                    └── __init__.py
                    ├── ...
```

## 定义Item

* Item 是保存爬取到的数据的容器；其使用方法和python字典类似
* 类似在ORM中做的一样，您可以通过创建一个 `scrapy.Item`类， 并且定义类型为 `scrapy.Field` 的类属性来定义一个Item
* 首先根据需要从网站获取到的数据对`item`进行建模
* 通过定义`item`，可以很方便的使用Scrapy的其他方法。而这些方法需要知道`item`的定义

## 编写spider

* Spider是用于从单个网站(或者一些网站)爬取数据的`类`
* 包含了一个用于下载的`初始URL，如何跟进网页中的链接以及如何分析页面中的内容， 提取生成 item 的方法`
* `Spider`必须继承 `scrapy.Spider` 类， 且定义一些属性:

  * `name`: 用于区别Spider。 该名字必须是唯一的，您不可以为不同的Spider设定相同的名字
  * `start_urls`: 包含了Spider在启动时进行爬取的url列表。 因此，第一个被获取到的页面将是其中之一。 后续的URL则从初始的URL获取到的数据中提取。

    * Scrapy为Spider的 `start_urls` 属性中的每个URL创建了 `scrapy.Request` 对象，并将 `parse` 方法作为回调函数(callback)赋值给了`Request`

  * `parse()` 是spider的一个方法。 被调用时，每个初始URL完成下载后生成的 `Response` 对象将会作为唯一的参数传递给该函数。 该方法负责解析返回的数据(response data)，提取数据(生成item)以及生成需要进一步处理的URL的 `Request` 对象。

    * `Request`对象经过调度，执行生成 `scrapy.http.Response` 对象并送回给spider `parse()`方法

* 追踪链接 `Following links`

  您在回调函数中`yield`一个`Request`后, `Scrpay`将会调度,发送该请求,并且在该请求完成时,调用所注册的回调函数。基于此方法,您可以根据您所定义的跟进链接的规则,创建`复杂`的`crawler`,并且, 根据所访问的页面,提取不同的数据

  ```python
    import scrapy

    class CapSpider(scrapy.Spider):
        name = 'cap'
        start_urls = ['http://120.55.55.87']

        def parse(self, response):
            for href in response.xpath('//p/a/@href'):
                full_url = response.urljoin(href.extract())
                yield scrapy.Request(full_url, callback=self.parse_article)

        def parse_article(self, response):
            yield {
                'title': response.css('.entry-title ::text').extract(),
                'category': response.css('.post-category a::text').extract(),
                'link': response.url,
            }
  ```
  * `for href in response.xpath('//p/a/@href')`遍历所有满足`xpath`的链接
  * `response.urljoin` 方法构造一个绝对路径的URL， 产生`yield`一个请求， 该请求使用 `parse_article()`方法作为回调函数, 用于最终产生数据

## 爬取

执行命令：`scrapy runspider cap.py -o cap.json`

## 提取Ttem

提取`Item`，Scrapy使用了一种基于 `XPath` 和 `CSS` 表达式机制，XPath提供了更强大的功能。其不仅仅能指明数据所在的路径， 还能查看数据

XPath表达式的例子及对应的含义:

* `/html/head/title`: 选择HTML文档中 `<head>` 标签内的 `<title>` 元素
* `/html/head/title/text()`: 选择上面提到的 `<title>` 元素的文字
* `//td`: 选择所有的 `<td>` 元素
* `//div[@class="mine"]`: 选择所有具有 `class="mine"` 属性的 div 元素

Selector有四个基本的方法:

* `xpath()`: 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表 。
* `css()`: 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表.
* `extract()`: 序列化该节点为unicode字符串并返回list。
* `re()`: 根据传入的正则表达式对数据进行提取，返回unicode字符串list列表。

在Shell中尝试Selector选择器

在项目根目录下执行 `scrapy shell "http://120.55.55.87"`

* 得到一个包含response数据的本地 response 变量

  * `response.xpath('//title')` 提取标题标签
  * `response.xpath('//title').extract()` 序列化`标题`为unicode字符串并返回list
  * `response.xpath('//title/text()').extract()` 提取`标题文本`并序列化为unicode字符串并返回list
  * `......`

* 拼接更多的 `.xpath()` 来进一步获取某个节点

  ```python
   def parse(self, response):
        for sel in response.xpath('//ul/li'):
            title = sel.xpath('a/text()').extract()
            link = sel.xpath('a/@href').extract()
            desc = sel.xpath('text()').extract()
            print title, link, desc
  ```

## 保存爬取到的数据

根据程序执行命令`scrapy runspider cap.py -o cap.json`，将结果保存在新生成的 `cap.json`中，小规模的项目中，这种存储方式已经足够。 如果需要对爬取到的item做更多更为复杂的操作，将用到 `Item Pipeline`
