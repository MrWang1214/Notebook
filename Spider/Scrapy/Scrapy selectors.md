# 选择器 `selectors`

  当抓取网页时，你做的最常见的任务是从HTML源码中提取数据。现有的一些库可以达到这个目的:

* `BeautifulSoup` 网页分析库，它基于HTML代码的结构来构造一个Python对象， 对不良标记的处理也非常合理，缺点：慢
* `lxml` 基于 `ElementTree` (不是Python标准库的一部分)的`python化的XML解析库`(也可以解析HTML)

  Scrapy提取数据有自己的一套机制。它们被称作`选择器(seletors)`，因为他们通过特定的 `XPath` 或者 `CSS` 表达式来“选择” HTML文件中的某个部分

* `XPath` 是一门用来在XML文件中选择节点的语言，也可以用在HTML上
* `CSS` 是一门将HTML文档样式化的语言。选择器由它定义，并与特定的HTML元素的样式相关连

## 构造选择器

  `response.selector.xpath('//span/text()').extract()`

  由于在response中使用XPath、CSS查询十分普遍，因此，Scrapy提供了两个实用的快捷方式: `response.xpath()` 及 `response.css()`

  `response.xpath('//span/text()').extract()`

## 使用选择器

* 提取真实的原文数据，调用 `.extract()` 方法如下:

```python
    >>> response.xpath('//title/text()').extract()

    [u'Example website']
```

* 如果要提取到第一个匹配到的元素, 调用 `.extract_first()` selector

```python
    >>> response.xpath('//div[@id="images"]/a/text()').extract_first()

    u'Name: My image 1 '
```

* 如果没有匹配的元素，则返回 None:

```python
    >>> response.xpath('//div/[id="not-exists"]/text()').extract_first() is None

    True
```

* 也可以设置默认的返回值，替代 None :

```python
    >>> sel.xpath('//div/[id="not-exists"]/text()').extract_first(default='not-found')
    'not-found'
```

* CSS选择器可以使用CSS3伪元素(pseudo-elements)来选择文字或者属性节点:

```python
    >>> response.css('title::text').extract()
    [u'Example website']
```

## 嵌套选择器

  选择器方法( `.xpath()` or `.css()` )返回相同类型的选择器列表，因此也可以对这些选择器调用选择器方法

```python
    >>> links = response.xpath('//a[contains(@href, "image")]')
    >>> links.extract()
    >>> for index, link in enumerate(links):
            args = (index, link.xpath('@href').extract(), link.xpath('img/@src').extract())
            print 'Link number %d points to url %s and image %s' % args

    Link number 0 points to url [u'image1.html'] and image [u'image1_thumb.jpg']
    Link number 1 points to url [u'image2.html'] and image [u'image2_thumb.jpg']
    Link number 2 points to url [u'image3.html'] and image [u'image3_thumb.jpg']
```

## 结合正则表达式使用选择器

  `Selector` 也有一个 `.re()` 方法，用来通过正则表达式来提取数据。不同于使用 `.xpath()` 或者 `.css()` 方法, `.re()` 方法返回unicode字符串的列表。所以无法构造嵌套式的 `.re()` 调用

* `.re_first()`糅合了 `.extract_first()` 与 `.re()`，使用该函数可以提取第一个匹配到的字符串

## 使用相对`XPaths`

  如果使用嵌套的选择器，并使用起始为 `/` 的XPath，那么该XPath将对文档使用绝对路径，而且对于调用的 `Selector` 不是相对路径

```python
    >>> divs = response.xpath('//div')
    >>> for p in divs.xpath('.//p'):
            print p.extract()
```

  `注意 .//p XPath的点前缀`

## 使用EXSLT扩展

  因建于 `lxml` 之上, `Scrapy选择器`也支持一些 `EXSLT` 扩展，可以在XPath表达式中使用预先制定的命名空间
