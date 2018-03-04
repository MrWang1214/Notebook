# 基于scrapy爬取扇贝每日翻译板块

## chrome调试分析扇贝网每日翻译板块

* 打开chorme开发者工具，`network`工具，过滤只剩`XHR`

    [![2018-03-03_2.36.56.png](https://s18.postimg.org/ac1r7zd7d/2018-03-03_2.36.56.png)](https://postimg.org/image/yfsiw9vo5/)

* 打开要爬取的每日翻译板块，URL:`https://www.shanbay.com/forum/daily-translation/#p1`
* 观察`request`情况，找到ajax对象
* 分析`Headers`，得到`Request URL`:`https://www.shanbay.com/api/v1/forum/11077/thread/?page=1&_=1520058969537`
* `preview`中可以看到交互的`json`数据

    [![2018-03-03_2.49.07.png](https://s18.postimg.org/7uq00rqqx/2018-03-03_2.49.07.png)](https://postimg.org/image/srm85forp/)

## 构建Scrapy项目

### 创建Scrapy项目 `Scrapy startproject shanbaySpider`

[![2018-03-03_2.53.55.png](https://s26.postimg.org/r82vzbnd5/2018-03-03_2.53.55.png)](https://postimg.org/image/f67i56e4l/)

### 创建shanbayitems

根据需要爬取的内容，创建`shanbayitems`，这个爬虫的功能是爬取每日翻译模块用户发布动态的`题目`、`作者`、`时间`。

```python
import scrapy

class ShanbayItem(scrapy.Item):
    title = scrapy.Field()
    author = scrapy.Field()
    topic_post_time = scrapy.Field()
```

### 创建爬虫shanbay

* 根据chrome调试，每一分页的`ajax Request URL`规则是`https://www.shanbay.com/api/v1/forum/11077/thread/?page=`,`page=`后面的数字是页数

  ```python
  class shanbaySpider(scrapy.Spider):
    name = 'shanbay'
    allow_domains = ["www.shanbay.com"]

    def start_requests(self):
        for i in range(1, 5):  #爬取最新五页的动态
            url = "https://www.shanbay.com/api/v1/forum/11077/thread/?page={}".format(i)
            yield scrapy.Request(url, callback=self.parse) #结果回调给parse
  ```

* 调用`parse`对爬取内容进行控制

  ```python
  def parse(self, response):
        res = json.loads(response.body)
        threads = res['data']['threads']
        threadItems = []
        for dict in threads:
            threadItem = ShanbayItem()
            threadItem['title'] = dict['title']
            threadItem['author'] = dict['author']['nickname']
            threadItem['topic_post_time'] = dict['topic_post_time']
            threadItems.append(threadItem)
        return threadItems
  ```

* 用到的python库

  ```python
  import scrapy
  import json  # 对response对象json化
  from scrapy.http import Request
  from shanbaySpider.shanbayitems import ShanbayItem #调用shanbayitems
  ```

### 设置pipilines

* 爬取结果输出到`data_shanbay.json`文件

```python
class JsonWithEncodingPipeline(object):
    def __init__(self):
        self.file = codecs.open('data_shanbay.json', 'wb', encoding='utf-8')

    def process_item(self, item, spider):
        line = json.dumps(dict(item)) + '\n'
        self.file.write(line.decode("unicode_escape"))
        return item
```

* 爬取结果存储到`mysql数据库`中

```python
class MysqlWithJsonPipeline(object):
    def __init__(self):
        dbparams = dict(
            host=settings['MYSQL_HOST'],  # 读取settings中的配置
            db=settings['MYSQL_DBNAME'],
            user=settings['MYSQL_USER'],
            passwd=settings['MYSQL_PASSWD'],
            charset='utf8',  # 编码要加上，否则可能出现中文乱码问题
            cursorclass=pymysql.cursors.DictCursor,
            use_unicode=False,
        )
        #**表示将字典扩展为关键字参数,相当于host=xxx,db=yyy....
        dbpool = adbapi.ConnectionPool('pymysql', **dbparams)

        self.dbpool = dbpool

    def connect(self):
        return self.dbpool

    def insert(self, item):
        sql = "INSERT INTO daily_translation (daily_translation_title, daily_translation_author, daily_translation_date) VALUES(%s,%s,%s)"
        #调用插入的方法
        self.dbpool.runInteraction(self._conditional_insert, sql, item)

    def _conditional_insert(self, tx, sql, item):
        params = (item["title"], item['author'], item['topic_post_time'])
        tx.execute(sql, params)
    
    def process_item(self, item, spider):
        # 插入数据库
        self.insert(item)
        return item
```

### 设置项目settings

```python
#设置数据库初始化数据
MYSQL_HOST = '127.0.0.1'
MYSQL_DBNAME = 'shanbay'
MYSQL_USER = 'root'
MYSQL_PASSWD = '********'
CHARSET='utf8'
MYSQL_PORT = 3306

#设置ITEM_PIPELINES
ITEM_PIPELINES = {
    'shanbaySpider.pipelines.JsonWithEncodingPipeline': 300,
    'shanbaySpider.pipelines.MysqlWithJsonPipeline': 300,
}
```

## 执行shanbay爬虫

在项目目录下执行`scrapy crawl shanbay`

* 结果存储到`data_shanbay.json`

  [![2018-03-03_3.17.33.png](https://s26.postimg.org/4ks86z2cp/2018-03-03_3.17.33.png)](https://postimg.org/image/i1p6puco5/)

* 结果存储到`mysql数据库`

  [![2018-03-03_3.18.53.png](https://s26.postimg.org/q778o5ljd/2018-03-03_3.18.53.png)](https://postimg.org/image/ei3906ukl/)