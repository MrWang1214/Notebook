# Ajax Learning

`AJAX` = `Asynchronous JavaScript and XML`（异步的 `JavaScript` 和`XML`）。

`AJAX` 优点是在不重新加载整个页面的情况下，可以与服务器交换数据并更新部分网页内容。

`AJAX` 不需要任何浏览器插件，但需要用户允许`JavaScript`在浏览器上执行。

## Ajax的主要运用

* 运用XHTML+CSS来表达资讯
* 运用JavaScript操作DOM（Document Object Model）来执行动态效果
* 运用XML和XSLT操作资料
* 运用XMLHttpRequest或新的Fetch API与网页服务器进行异步资料交换
* 注意：AJAX与Flash、Silverlight和Java Applet等RIA技术是有区分的

## AJAX是基于现有的Internet标准

* AJAX是基于现有的Internet标准，并且联合使用它们：

  * XMLHttpRequest 对象 (异步的与服务器交换数据)
  * JavaScript/DOM (信息显示/交互)
  * CSS (给数据定义样式)
  * XML (作为转换数据的格式)

## XMLHttpRequest 对象

所有现代浏览器均支持 `XMLHttpRequest` 对象（IE5 和 IE6 使用 `ActiveXObject`）。

`XMLHttpRequest` 用于在后台与服务器交换数据。

* 创建`XMLHttpRequest`对象

  ```JavaScript
      variable=new XMLHttpRequest();
  ```
  IE5和IE6使用ActiveX对象

  ```JavaScript
      variable=new ActiveXObject("Microsoft.XMLHTTP");
  ```
* `XMLHttpRequest`对象`open()`和`send()`方法，用于将请求发往服务器

  * open(method,url,async)，规定请求的类型、URL 以及是否异步处理请求。

    * method：请求的类型；GET 或 POST
    * url：文件在服务器上的位置
    * async：true（异步）或 false（同步），Ajax必须是异步的（true）

  * send(string)，将请求发送到服务器。

    * string：仅用于 POST 请求

  * xmlhttp.setRequestHeade，添加 HTTP header。

  * xmlhttp.onreadystatechange 存储函数（或函数名），readyState 属性改变时，就会调用该函数。

  * xmlhttp.readyState 存有 XMLHttpRequest 的状态。从 0 到 4 发生变化。

    * 0: 请求未初始化
    * 1: 服务器连接已建立
    * 2: 请求已接收
    * 3: 请求处理中
    * 4: 请求已完成，且响应已就绪

  * status属性中，200: "OK"，404: 未找到页面

## Ajax服务器响应

* responseText属性
  
  服务器响应非XML时使用

* responseXML属性

  来自服务器的响应是 XML，而且需要作为 XML 对象进行解析，使用`responseXML`属性

  ```JavaScript
      xmlDoc=xmlhttp.responseXML;
      txt="";
      x=xmlDoc.getElementsByTagName("ARTIST");
      for (i=0;i<x.length;i++)
      {
          txt=txt + x[i].childNodes[0].nodeValue + "<br>";
      }
      document.getElementById("myDiv").innerHTML=txt;
  ```

## onreadystatechange 事件

当请求被发送到服务器时，我们需要执行一些基于响应的任务。

每当 readyState 改变时，就会触发 onreadystatechange 事件。

readyState 属性存有 XMLHttpRequest 的状态信息。

`XMLHttpRequest`属性

## 使用回调函数

通过使用回调函数来应对页面上多个Ajax任务。

```JavaScript
    var xmlhttp;
    function loadXMLDoc(url,cfunc)
    {
    if (window.XMLHttpRequest)
    {// IE7+, Firefox, Chrome, Opera, Safari 代码
    xmlhttp=new XMLHttpRequest();
    }
    else
    {// IE6, IE5 代码
    xmlhttp=new ActiveXObject("Microsoft.XMLHTTP");
    }
    xmlhttp.onreadystatechange=cfunc;
    xmlhttp.open("GET",url,true);
    xmlhttp.send();
    }
    function myFunction()
    {//回调loadXMLDoc(url,cfunc)
    loadXMLDoc("/try/ajax/ajax_info.txt",function()
        {
        if (xmlhttp.readyState==4 && xmlhttp.status==200)
        {
        document.getElementById("myDiv").innerHTML=xmlhttp.responseText;
        }
        });
    }
```