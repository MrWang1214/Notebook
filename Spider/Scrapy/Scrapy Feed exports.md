# Feed exports

实现爬虫时最经常提到的需求就是能合适的保存爬取到的数据，或者说，生成一个带有爬取数据的”输出文件”(通常叫做”输出feed”)，来供其他系统使用

Scrapy自带了`Feed输出`，并且支持多种序列化格式`serialization format`及存储方式`storage backends`

## 序列化方式(Serialization formats)

feed输出使用到了 `Item exporters` 。其自带支持的类型有:

* JSON
  * `FEED_FORMAT`: json
  * 使用的exporter: `JsonItemExporter`
  * `JSON` 是一个简单而有弹性的格式, 但对大量数据的扩展性不是很好,因为这里会将整个对象放入内存. 如果要JSON既强大又简单,可以考虑 `JsonLinesItemExporter`,或把输出对象分为多个块.

* JSON lines
  * `FEED_FORMAT`: jsonlines
  * 使用的exporter: `JsonLinesItemExporter`

* CSV
  * `FEED_FORMAT`: csv
  * 使用的exporter: `CsvItemExporter`
  * 指定要export的列及其顺序使用 `FEED_EXPORT_FIELDS`。其他`feed exporters`也可以使用此选项，但它对CSV很重要，因为与许多其他导出格式不同，CSV使用 `a fixed header`

* XML
  * `FEED_FORMAT`: xml
  * 使用的exporter: `XmlItemExporter`

也可以通过 `FEED_EXPORTERS` 设置扩展支持的属性

## 存储(Storages)

使用`feed`输出时可以通过使用`URI`(通过 `FEED_URI` 设置) 来定义存储端。 feed输出支持`URI`方式支持的多种存储后端类型

自带支持的存储后端有:

* 本地文件系统
  * URI scheme: `file`
  * URI样例: `file:///tmp/export.csv`
  * 需要的外部依赖库: none

  注意: (只有)存储在本地文件系统时，您可以指定一个绝对路径 /tmp/export.csv 并忽略协议`scheme`,不过这仅仅只能在Unix系统中工作

* FTP
  * URI scheme: `ftp`
  * URI样例: `ftp://user:pass@ftp.example.com/path/to/export.csv`
  * 需要的外部依赖库: none

* S3 (需要 boto)
  * URI scheme: `s3`
  * URI样例:
    * `s3://mybucket/path/to/export.csv`
    * `s3://aws_key:aws_secret@mybucket/path/to/export.csv`
  * 需要的外部依赖库: boto

  可以通过在URI中传递user/pass来完成AWS认证，或者也可以通过下列的设置来完成:

  * `AWS_ACCESS_KEY_ID`
  * `AWS_SECRET_ACCESS_KEY`

* 标准输出
  * URI scheme: `stdout`
  * URI样例: `stdout:`
  * 需要的外部依赖库: none

有些存储后端会因所需的外部库未安装而不可用。例如，S3只有在`boto`库安装的情况下才可使用

## 存储URI参数

存储URI也包含参数。当feed被创建时这些参数可以被覆盖:

* `%(time)s` - 当feed被创建时被timestamp覆盖
* `%(name)s` - 被spider的名字覆盖

其他命名的参数会被spider同名的属性所覆盖。当feed被创建时，`%(site_id)s`将会被`spider.site_id`属性所覆盖

* 存储在FTP: `ftp://user:password@ftp.example.com/scraping/feeds/%(name)s/%(time)s.json`

* 存储在S3: `s3://mybucket/scraping/feeds/%(name)s/%(time)s.json`

## 设定(Settings)

* `FEED_URI` (必须)
  * Default: `None`.输出feed的URI。为了启用feed输出，该设定是必须的

* `FEED_FORMAT`
  * 输出feed的序列化格式

* `FEED_STORAGES` `FEED_STORAGES_BASE`
  * `Default:: {}`. 包含项目支持的额外feed存储端的字典。 字典的键`key`是URI协议`scheme`，值是存储类`storage class`的路径

* `FEED_EXPORTERS` `FEED_EXPORTERS_BASE`
  * `Default:: {}`. 包含项目支持的额外输出器`exporter`的字典。 该字典的键`key`是URI协议`scheme`，值是Item输出器`exporter`类的路径

* `FEED_STORE_EMPTY`
  * 是否输出空feed(没有item的feed)

* `FEED_EXPORT_FIELDS`
  * 要导出的字段列表，可选。例如：`FEED_EXPORT_FIELDS = ["foo", "bar", "baz"]`
    * 使用`FEED_EXPORT_FIELDS`选项来定义要导出的字段及其顺序
    * 当`FEED_EXPORT_FIELDS`为空或无（默认值）时，Scrapy使用在`dict`或`Item`正在生成的子类中定义的字段
    * 如果导出程序需要一组固定的字段（针对CSV导出格式），而`FEED_EXPORT_FIELDS`为空或无，则Scrapy会尝试从导出的​​数据中推断字段名称,目前它使用第一项中的字段名称
