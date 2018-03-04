# Link Extractors

`Link Extractors` 是那些目的仅仅是从网页(`scrapy.http.Response` 对象)中抽取最终将会被`follow`链接的对象｡

Scrapy提供了 `scrapy.linkextractors import LinkExtractor` , 通过实现一个接口创建定制的`Link Extractor`来满足需求｡

每个link extractor有唯一的公共方法是 `extract_links` ,它接收一个 `Response` 对象,并返回一个 `scrapy.link.Link`对象｡`Link Extractors`,要实例化一次并且 `extract_links` 方法会根据不同的response调用多次提取链接｡

Link Extractors在 `CrawlSpider` 类(在Scrapy可用)中使用, 通过一套规则,但你也可以用它在你的Spider中, 即使你不是从 CrawlSpider 继承的子类, 因为它的目的很简单: 提取链接｡

## 内置Link Extractor 参考

Scrapy提供的`Link Extractor类`在 `scrapy.linkextractors` 模块提供｡ 默认的link extractor是 `LinkExtractor`,其实就是 `LxmlLinkExtractor`:

`from scrapy.linkextractors import LinkExtractor`

```python
from scrapy.linkextractors import LinkExtractor
from scrapy.spiders import CrawlSpider, Rule

class GoogleDirectorySpider(CrawlSpider):
    name = 'directory.google.com'
    allowed_domains = ['directory.google.com']
    start_urls = ['http://directory.google.com/']

    rules = (
        Rule(LinkExtractor(allow='directory\.google\.com/[A-Z][a-zA-Z_/]+$'),
            'parse_category', follow=True,
        ),
    )

    def parse_category(self, response):
        # write the category page data extraction code here
        pass
```