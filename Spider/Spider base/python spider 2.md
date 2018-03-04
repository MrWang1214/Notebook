# Python中urllib和urllib2库的用法

## `urllib`提供了一系列用于操作URL的功能，即发送请求到指定页面，返回服务器（HTTP）的响应。包括以下模块：

* urllib.request 请求模块
  * urlopen函数

    * `urllib.request.urlopen(url, data=None, [timeout, ]*, cafile=None, capath=None, cadefault=False, context=None)`
    * 例子
      ```Python
      import urllib.request
      response = urllib.request.urlopen('http://120.55.55.87')
      print(response.read().decode('utf-8'))
      ```
    * 常用的三个参数`url`、`data`、`timeout`。`data`参数多用于post时请求数据构造，如果没有`data`就是get请求。`timeout`参数用于服务器超时、异常的情况程序等待的时间。
    * `data`数据构造
      ```Python
      headers = {
      'User-Agent': 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)',
      'Host': '120.55.55.87'
      }
      dict = {
      'user_name': 'capath',
      'password': '123456'
      }
      ```
      也可以通过`req.add_header('User-Agent', 'Mozilla/4.0 (compatible; MSIE 5.5; Windows NT)')`来构建header。
  * ProxyHandler函数
    * 用于设置代理，避免因为同一IP访问过多被网站限制，而无法继续爬取。
      ```Python
      import urllib.request
      proxy_support = urllib.request.ProxyHandler({'sock5': 'localhost:1080'})
      opener = urllib.request.build_opener(proxy_support)
      urllib.request.install_opener(opener)
      a = urllib.request.urlopen("http://www.jb51.net ").read().decode("utf8")
      print(a)
      ```

* urllib.error 异常处理模块
  * urllib.error有两个异常错误：URLError,HTTPError
    * HTTPError是URLError的子类
    * URLError里只有一个属性：reason,即抓异常的时候只能打印错误信
    * HTTPError里有三个属性：code,reason,headers，即抓异常的时候可以获得code,reson，headers三个信息。
  ```Python
    from urllib import request,error
    try:
        response = request.urlopen("http://pythonsite.com/1111.html")
    except error.HTTPError as e:
        print(e.reason)
        print(e.code)
        print(e.headers)
    except error.URLError as e:
        print(e.reason)
    else:
        print("reqeust successfully")
  ```
* urllib.parse url解析模块
  * parse模块用于切分url组成，或是组成url,常用两个函数`urlparse`，`urlunpars`。
    * urlparse函数
      ```Python
      from urllib import parse
      url = 'http://120.55.55.87/login'
      result = parse.urlparse(url)
      print(result)
      ```
    * urlunpars函数
      ```python
      from urllib.parse import urlunparse
      data = ['http','120.55.55.87','/login']
      print(urlunparse(data))
      ```
    * `urljoin`用法和`urlunpars`相似
    * `urlencode`功能是将字典转化为参数
* urllib.robotparser robots.txt解析模块

## `urllib2`也是python中用于处理URL的模块，但是有很多新的特性：

* `urllib2`接受一个Request类的实例来设置URL请求的header，而`urllib`只能接受URL
* `urllib`提供`urlencode`方法用来GET查询字符串的产生，而`urllib2`没有

因此`urllib`和`urllib2`常常配合使用，目前大部分http请求是通过`urllib2`来访问的。
