# spider类

`Spider类`定义了如何爬取某个网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据`爬取item`

1. 以初始的URL初始化`Request`，并设置回调函数。 当该`request`下载完毕并返回时，将生成`response`，并作为参数传给该回调函数， spider中初始的request是通过调用 `start_requests()` 来获取的。 `start_requests()` 读取 `start_urls` 中的`URL`， 并以 `parse` 为回调函数生成 `Request`
2. 在回调函数内分析返回的(网页)内容，返回 `Item` 对象、`dict`、 `Request` 或者一个包括三者的可迭代容器。 返回的`Request`对象之后会经过`Scrapy`处理，下载相应的内容，并调用设置的`callback函数`(函数可相同)
3. 在回调函数内，您可以使用 `选择器(Selectors)` (您也可以使用`BeautifulSoup`, `lxml` 或者您想用的任何解析器) 来分析网页内容，并根据分析的数据生成item
4. 最后，由`spider`返回的`item`将被存到数据库(由某些 `Item Pipeline` 处理)或使用 `Feed exports` 存入到文件中

## scrapy.Spider

* `class scrapy.spiders.Spider`

  提供了 `start_requests()` 的默认实现，读取并请求spider属性中的 `start_urls`, 并根据返回的结果`resulting responses`调用spider的 `parse` 方法

  * `name`

    定义spider名字的字符串(string)。spider的名字定义了Scrapy如何定位(并初始化)spider，所以其必须是唯一的。

  * `allowed_domains`

    可选。包含了spider允许爬取的域名(domain)列表(list)。 当 `OffsiteMiddleware` 启用时， 域名不在列表中的URL不会被跟进

  * `start_urls`

    URL列表。当没有制定特定的URL时，spider将从该列表中开始进行爬取。 因此，第一个被获取到的页面的URL将是该列表之一。 后续的URL将会从获取到的数据中提取

  * `custom_settings`

    该设置是一个dict.当启动spider时,该设置将会`覆盖项目级的设置`. 由于设置必须在初始化`instantiation`前被更新,所以该属性必须定义为`class属性`

  * `crawler`

    该属性在初始化class后,由类方法 `from_crawler()` 设置, 并且链接了本spider实例对应的 `Crawler` 对象

  * `start_requests()`

    该方法必须返回一个可迭代对象(iterable)。该对象包含了spider用于爬取的第一个`Request`，当spider启动爬取并且未制定URL时，该方法被调用
    该方法的默认实现是使用 start_urls 的url生成Request

    ```python
    import scrapy
    from myproject.items import MyItem

    class MySpider(scrapy.Spider):
        name = 'example.com'
        allowed_domains = ['example.com']

        def start_requests(self):
            yield scrapy.Request('http://www.example.com/1.html', self.parse)
            yield scrapy.Request('http://www.example.com/2.html', self.parse)
            yield scrapy.Request('http://www.example.com/3.html', self.parse)

        def parse(self, response):
            for h3 in response.xpath('//h3').extract():
                yield MyItem(title=h3)

            for url in response.xpath('//a/@href').extract():
                yield scrapy.Request(url, callback=self.parse)
    ```

  * `make_requests_from_url(url)`

    该方法接受一个URL并返回用于爬取的 `Request` 对象。 该方法在初始化request时被 `start_requests()` 调用，也被用于转化url为request

  * `parse(response)`

    * 当response没有指定回调函数时，该方法是Scrapy处理下载的response的默认方法
    * parse 负责处理response并返回处理的数据以及(/或)跟进的URL。 Spider 对其他的Request的回调函数也有相同的要求
    * `parse(response)`及其他的Request回调函数必须返回一个包含 Request、dict 或 Item 的可迭代的对象

  * `log(message[, level, component])`

    使用 `scrapy.log.msg()` 方法记录(log)message。 log中自动带上该spider的 `name` 属性

  * `closed(reason)`

    当spider关闭时，该函数被调用。 该方法提供了一个替代调用`signals.connect()`来监听 `spider_closed` 信号的快捷方式

* `CrawlSpider`

  `class scrapy.spiders.CrawlSpider`,爬取一般网站常用的spider。其定义了一些规则(rule)来提供跟进link的方便的机制

  * `rules`

    一个包含一个(或多个) `Rule` 对象的集合(list)。 每个 `Rule` 对爬取网站的动作定义了特定表现。 Rule对象在下边会介绍。 如果多个rule匹配了相同的链接，则根据他们在本属性中被定义的顺序，第一个会被使用

  * `parse_start_url(response)`

    当`start_url`的请求返回时，该方法被调用。 该方法分析最初的返回值并必须返回一个 `Item` 对象或者 一个 `Request` 对象或者一个可迭代的包含二者对象

* `爬取规则(Crawling rules)`

  `class scrapy.spiders.Rule(link_extractor, callback=None, cb_kwargs=None, follow=None, process_links=None, process_request=None)`

  * `link_extractor` 是一个 `Link Extractor` 对象。 其定义了如何从爬取到的页面提取链接
  * `callback` 是一个`callable`或`string`(该spider中同名的函数将会被调用)。 从`link_extractor`中每获取到链接时将会调用该函数。该回调函数接受一个response作为其第一个参数， 并返回一个包含 `Item` 以及(或) `Request` 对象(或者这两者的子类)的列表`list`

    当编写爬虫规则时，请避免使用 parse 作为回调函数。 由于 CrawlSpider 使用 parse 方法来实现其逻辑，如果 您覆盖了 parse 方法，crawl spider 将会运行失败

  * `cb_kwargs` 包含传递给回调函数的参数(keyword argument)的字典。
  * `follow` 是一个布尔(boolean)值，指定了根据该规则从response提取的链接是否需要跟进。 如果 `callback` 为None， `follow` 默认设置为 True ，否则默认为 `False` 。
  * `process_links` 是一个callable或string(该spider中同名的函数将会被调用)。 从link_extractor中获取到链接列表时将会调用该函数。该方法主要用来过滤。
  * `process_request` 是一个callable或string(该spider中同名的函数将会被调用)。 该规则提取到每个request时都会调用该函数。该函数必须返回一个request或者None。 (用来过滤request)

  ```python
  import scrapy
  from scrapy.spiders import CrawlSpider, Rule
  from scrapy.linkextractors import LinkExtractor

  class MySpider(CrawlSpider):
      name = 'example.com'
      allowed_domains = ['example.com']
      start_urls = ['http://www.example.com']

      rules = (
          # 提取匹配 'category.php' (但不匹配 'subsection.php') 的链接并跟进链接(没有callback意味着follow默认为True)
          Rule(LinkExtractor(allow=('category\.php', ), deny=('subsection\.php', ))),

          # 提取匹配 'item.php' 的链接并使用spider的parse_item方法进行分析
          Rule(LinkExtractor(allow=('item\.php', )), callback='parse_item'),
      )

      def parse_item(self, response):
          self.logger.info('Hi, this is an item page! %s', response.url)

          item = scrapy.Item()
          item['id'] = response.xpath('//td[@id="item_id"]/text()').re(r'ID: (\d+)')
          item['name'] = response.xpath('//td[@id="item_name"]/text()').extract()
          item['description'] = response.xpath('//td[@id="item_description"]/text()').extract()
          return item
  ```

  该spider将从example.com的首页开始爬取，获取category以及item的链接并对后者使用 parse_item 方法。 当item获得返回(response)时，将使用XPath处理HTML并生成一些数据填入 Item 中

* `XMLFeedSpider`

  `class scrapy.spiders.XMLFeedSpider`，`XMLFeedSpider`被设计用于通过迭代各个节点来分析XML源(XML feed)。 迭代器可以从 `iternodes` ， `xml` ， `html` 选择。 鉴于 `xml` 以及 `html` 迭代器需要先读取所有DOM再分析而引起的性能问题， 一般还是推荐使用 `iternodes` 。 不过使用 `html` 作为迭代器能有效应对错误的XML，从给定的 `start_urls` 中下载feed， 并迭代feed中每个 `item` 标签，输出，并在 `Item` 中存储有些随机数据

  * `iterator`用于确定使用哪个迭代器的string，默认值为 `iternodes`。可选项有:

    * `iternodes` 高性能的基于正则表达式的迭代器
    * `html`和`xml` 使用 Selector 的迭代器。 需要注意的是该迭代器使用DOM进行分析，其需要将所有的DOM载入内存， 当数据量大的时候会产生问题。

  * `itertag`一个包含开始迭代的节点名的string。例如: `itertag = 'product'`

  * `namespaces` 一个由 `(prefix, url)` 元组(tuple)所组成的list。 其定义了在该文档中会被spider处理的可用的namespace。 `prefix` 及 `uri` 会被自动调用 `register_namespace()` 生成namespace，在 itertag 属性中制定节点的namespace

  * 除了这些新的属性之外，该spider也有以下可以覆盖(overrideable)的方法:

    * `adapt_response(response)` 该方法在spider分析response前被调用。您可以在response被分析之前使用该函数来修改内容(body)。 该方法接受一个response并返回一个response(可以相同也可以不同)

    * `parse_node(response, selector)`当节点符合提供的标签名时(itertag)该方法被调用。 接收到的response以及相应的 Selector 作为参数传递给该方法。 该方法返回一个 Item 对象或者 Request 对象 或者一个包含二者的可迭代对象(iterable)

    * `process_results(response, results)`当spider返回结果(item或request)时该方法被调用。 设定该方法的目的是在结果返回给框架核心(framework core)之前做最后的处理， 例如设定item的ID。其接受一个结果的列表(list of results)及对应的response。 其结果必须返回一个结果的列表(list of results)(包含Item或者Request对象)

  ```python
  from scrapy.spiders import XMLFeedSpider
  from myproject.items import TestItem

  class MySpider(XMLFeedSpider):
      name = 'example.com'
      allowed_domains = ['example.com']
      start_urls = ['http://www.example.com/feed.xml']
      iterator = 'iternodes' # This is actually unnecessary, since it's the default value
      itertag = 'item'

      def parse_node(self, response, node):
          self.logger.info('Hi, this is a <%s> node!: %s', self.itertag, ''.join(node.extract()))

          item = TestItem()
          item['id'] = node.xpath('@id').extract()
          item['name'] = node.xpath('name').extract()
          item['description'] = node.xpath('description').extract()
          return item
  ```

* `CSVFeedSpider`

* `SitemapSpider`
