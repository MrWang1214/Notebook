# Items

  爬取的主要目标就是从非结构性的数据源提取结构性数据。 `Scrapy spider`可以以python的dict来返回提取的数据.虽然dict很方便，并且用起来也熟悉，但是其缺少结构性，容易打错字段的名字或者返回不一致的数据，尤其在具有多个spider的大项目中

  为了定义常用的输出数据，Scrapy提供了 `Item` 类。 `Item` 对象是种简单的容器，保存了爬取到得数据。 其提供了类似于词典`(dictionary-like)`的API以及用于声明可用字段的简单语法

  许多Scrapy组件使用了Item提供的额外信息: `exporter`根据Item声明的字段来导出数据、 序列化可以通过Item字段的元数据(metadata)来定义、 `trackref` 追踪Item实例来帮助寻找内存泄露 (`see` 使用 trackref `调试内存泄露`) 等等

## 声明Item

```python
import scrapy

class Product(scrapy.Item):
    name = scrapy.Field()
    price = scrapy.Field()
    stock = scrapy.Field()
    last_updated = scrapy.Field(serializer=str)
```

## Item字段`Item Fields`

* `Field` 对象指明了每个字段的元数据(metadata)

* `Field` 对象对接受的值没有任何限制，因为这个原因，文档也无法提供所有可用的元数据的键(key)参考列表

* `Field` 对象中保存的每个键可以由多个组件使用，并且只有这些组件知道这个键的存在

* 设置 `Field` 对象的主要目的就是在一个地方定义好所有的元数据

* 用来声明item的 `Field` 对象并没有被赋值为class的属性。 不过您可以通过 `Item.fields` 属性进行访问

## 扩展Item

`class scrapy.item.Item([arg])`，返回一个根据给定的参数可选初始化的item

* Item复制了标准的 `dict API` 。包括初始化函数也相同。Item唯一额外添加的属性是:

  * `fields` 一个包含了item所有声明的字段的字典，而不仅仅是获取到的字段。该字典的key是字段(field)的名字，值是 `Item声明` 中使用到的 `Field`对象

## 字段(Field)对象

`class scrapy.item.Field([arg])`

* `Field` 仅仅是内置的 `dict 类`的一个别名，并没有提供额外的方法或者属性。换句话说， `Field` 对象完完全全就是Python字典(dict)。被用来基于类属性(class attribute)的方法来支持 `item声明语法`

## Item Loaders

`item Loaders` 提供了一种简便的构件（mechanism）来抓取:`ref:Items`. 虽然Items可以从它自己的类似字典（dictionary-like）的API得到所需信息 ,不过 `Item Loaders`提供了许多更加方便的API，这些API通过自动完成那些具有共通性的任务，可从抓取进程中得到这些信息, 比如预先解析提取到的原生数据

`Items` 提供了盛装抓取到的数据的容器 , 而`Item Loaders`提供了构件装载`populating`该容器

`Item Loaders` 被设计用来提供一个既弹性又高效简便的构件， 以扩展或重写爬虫或源格式(HTML, XML之类的)等区域的解析规则，方便了后期维护

## 用Item Loaders装载Items

* 要使用`Item Loader`, 必须先实例化. 使用类似字典的对象(例如: Item or dict)来进行实例化, 或者不使用对象也可以, 当不用对象进行实例化的时候,Item会自动使用 `ItemLoader.default_item_class` 属性中指定的Item 类在`Item Loader constructor`中实例化

* 开始收集数值到`Item Loader`时,通常使用`Selectors`. 可以在同一个`item field`里面添加多个数值;`Item Loader`将知道如何用合适的处理函数来“添加”这些数值

  * 当所有数据被收集起来之后, 调用 ItemLoader.load_item() 方法, 实际上填充并且返回了之前通过调用 add_xpath(), add_css(), and add_value() 所提取和收集到的数据的Item

  ```python
  from scrapy.loader import ItemLoader
  from myproject.items import Product

  def parse(self, response):
      l = ItemLoader(item=Product(), response=response)
      l.add_xpath('name', '//div[@class="product_name"]')
      l.add_xpath('name', '//div[@class="product_title"]')
      l.add_xpath('price', '//p[@id="price"]')
      l.add_css('stock', 'p#stock]')
      l.add_value('last_updated', 'today') # you can also use literal values
      return l.load_item()
  ```

### Input and Output processors

`Item Loader`在每个(Item)字段中都包含了一个`输入处理器`和一个`输出处理器`｡ 输入处理器收到数据时立刻提取数据 (通过 `add_xpath()`, `add_css()` 或者 `add_value()` 方法) 之后输入处理器的结果被收集起来并且保存在ItemLoader内. 收集到所有的数据后, 调用 ItemLoader.load_item() 方法来填充,并得到填充后的 Item 对象. 这是当输出处理器被和之前收集到的数据(和用输入处理器处理的)被调用.输出处理器的结果是被分配到Item的最终值

```python
l = ItemLoader(Product(), some_selector)
l.add_xpath('name', xpath1) # (1)
l.add_xpath('name', xpath2) # (2)
l.add_css('name', css) # (3)
l.add_value('name', 'test') # (4)
return l.load_item() # (5)
```

1. 从xpath1提取出的数据，给传递输入侧处理器的name字段。输入处理器的结果被收集和保存在档案装载机中（但尚未分配给该项目）
2. 从xpath2提取出来的数据，传递给（1）中使用的相同的输入处理器。输入处理器的结果被附加到在（1）中收集的数据（如果有的话）
3. 除了数据是从cssCSS选择器中提取并通过（1）和（2）中使用的相同输入处理器之外，这种情况与之前的类似。输入处理器的结果附加到（1）和（2）中收集的数据（如果有的话）
4. 这种情况也类似于以前的情况，不同之处在于要收集的值是直接分配的，而不是从XPath表达式或CSS选择器中提取。但是，该值仍然通过输入处理器。在这种情况下，由于该值不可迭代，因此在将其传递给输入处理器之前将其转换为单个元素的迭代，因为输入处理器总是接收迭代
5. 步骤（1），（2），（3）和（4）中收集的数据通过该字段的输出处理器name。输出处理器的结果是分配给name 项目中字段的值

## Item Pipeline

当Item在Spider中被收集之后，它将会被传递到`Item Pipeline`，一些组件会按照一定的顺序执行对Item的处理。每个`item pipeline`组件(有时称之为“Item Pipeline”)是实现了简单方法的Python类。他们接收到Item并通过它执行一些行为，同时也决定此Item是否继续通过`pipeline`，或是被丢弃而不再进行处理

`item pipeline`的一些典型应用：

1. 清理HTML数据
2. 验证爬取的数据(检查item包含某些字段)
3. 查重(并丢弃)
4. 将爬取结果保存到数据库中

* 构建`item pipeline`

  * `process_item(self, item, spider)`

    每个`item pipeline组件`都需要调用该方法，这个方法必须返回一个具有数据的dict，或是 `Item` (或任何继承类)对象， 或是抛出 `DropItem 异常`，被丢弃的item将不会被之后的pipeline组件所处理。

    `item` (Item 对象或者一个dict) – 被爬取的item
    `spider` (Spider 对象) – 爬取该item的spider

  * open_spider(self, spider)
    当spider被开启时，这个方法被调用。

    `spider` (Spider 对象) – 被开启的spider

  * close_spider(self, spider)
    当spider被关闭时，这个方法被调用

    `spider` (Spider 对象) – 被关闭的spider

  * from_crawler(cls, crawler)
    如果存在，这个classmethod被调用创建一个`pipeline`实例。它必须返回一个新的`pipeline`实例。`Crawler`提供对所有Scrapy核心组件的访问，如`settings`和`signals`; 它是`pipeline`访问它们并将其功能`hook`到Scrapy的一种方式

    `crawler` (Crawler object) – 使用此`pipeline`的crawler

* 例子——去重

  ```python
  from scrapy.exceptions import DropItem

  class DuplicatesPipeline(object):

      def __init__(self):
          self.ids_seen = set()

      def process_item(self, item, spider):
          if item['id'] in self.ids_seen:
              raise DropItem("Duplicate item found: %s" % item)
          else:
              self.ids_seen.add(item['id'])
              return item
  ```

* 启用一个`Item Pipeline`组件

为了启用一个`Item Pipeline`组件，必须将它的类添加到 `ITEM_PIPELINES` 配置

``` python
ITEM_PIPELINES = {
    'myproject.pipelines.PricePipeline': 300,
    'myproject.pipelines.DuplicatesPipeline': 800,
}
```

分配给每个类的整型值，确定了他们运行的顺序，item按数字从低到高的顺序，通过pipeline，通常将这些数字定义在0-1000范围内
