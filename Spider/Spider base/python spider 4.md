# Python爬虫框架Scrapy

Scrapy是一个为了爬取网站数据，提取结构性数据而编写的应用框架。 可以应用在包括数据挖掘，信息处理或存储历史数据等一系列的程序中。

## Mac下安装Scrapy

因为Mac下自带Python中`six`库不能满足`Scrapy`使用，又不影响系统的情况下，使用virtualenv来安装

  1. `sudo pip install virtualenv`
  2. `virtualenv scrapyenv`
  3. `cd scrapyenv`
  4. `source bin/activate`
  5. `pip install Scrapy`
  6. `Scrapy startproject xxxx` //创建scrapy项目

## Scrapy项目结构

```json
xxxx/
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

## Scrapy

爬取StackOverflow上具有投票数最多的链接的例子：

```python
import scrapy

class StackOverflowSpider(scrapy.Spider):
    name = 'stackoverflow'
    start_urls = ['http://stackoverflow.com/questions?sort=votes']

    def parse(self, response):
        for href in response.css('.question-summary h3 a::attr(href)'):
            full_url = response.urljoin(href.extract())
            yield scrapy.Request(full_url, callback=self.parse_question)

    def parse_question(self, response):
        yield {
            'title': response.css('h1 a::text').extract()[0],
            'votes': response.css('.question .vote-count-post::text').extract()[0],
            'body': response.css('.question .post-text').extract()[0],
            'tags': response.css('.question .post-tag::text').extract(),
            'link': response.url,
        }
```

* `scrapy runspider stackoverflow_spider.py -o top-stackoverflow-questions.json`

  * 当您运行 `scrapy runspider stackoverflow_spider.py` 命令时，Scrapy尝试从该文件中查找Spider的定义，并且在爬取引擎中运行它

  * `-o top-stackoverflow-questions.json` 旨在将结果输出到top-stackoverflow-questions.json文件中

* 爬虫执行的主要过程：

  1. 读取定义在`start_urls`属性中的URL，创建请求`Request`

     请求`request`是被异步调度和处理的。这意味着，`Scrapy`并不需要等待一个请求`request`完成及处理，在此同时，也发送其他请求或者做些其他事情。这也意味着，当有些请求失败或者处理过程中出现错误时，其他的请求也能继续处理。

  2. 将接收到的`response`作为参数调用默认的回调函数`parse`，启动爬取

  3. 在回调函数 `parse` 中，我们使用`CSS Selector`来提取链接。`Scrapy`提供CSS选择器`css selector`、`XPath表达式`，以及一些帮助函数`helper method`来使用正则表达式来提取数据

  4. 接着，我们产生`(yield)`更多的请求， 注册 `parse_question` 作为这些请求完成时的回调函数。从每个页面中爬取到问题`question`的数据并产生了一个`dict`，`Scrapy`收集并按照终端`command line`的要求将这些结果写入到了`JSON`文件中。

     这里使用了 `feed exports` 来创建了JSON文件，您可以很容易的改变导出的格式(比如`XML`或`CSV`)或者存储后端(例如`FTP`或者 `Amazon S3`)。 您也可以编写 `item pipeline` 来将item存储到数据库中。

* 定义Item

  * Item 是保存爬取到的数据的容器；其使用方法和python字典类似
  * 类似在ORM中做的一样，您可以通过创建一个 `scrapy.Item`类， 并且定义类型为 `scrapy.Field` 的类属性来定义一个Item