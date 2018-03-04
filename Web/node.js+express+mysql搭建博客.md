### 需求
一直没有习惯写博客，做笔记的习惯也比较差，知识积累杂乱，萌生搭建一个私人博客的想法。

我选择了node.js+express+mysql这种技术栈来实现博客搭建，在阿里云上租了一个轻量应用服务器（45一个月 T_T），零零散散花了一个星期贴了出来，push到github上(https://github.com/MrWang1214/CaptainBlog-1.0)，欢迎fork，不吝star。

### mac node.js环境的配置
默认的安装参数不会安装npm包管理器。

```
brew install node --with-npm  
```

检查安装是否成功

```
➜  node -v  
v6.0.0  
➜  npm -v  
3.8.6  
```

使用淘宝的npm源，淘宝提供了多种使用方式，这里我使用别名的方式，

```
echo '\n#alias for cnpm\nalias cnpm="npm  
--registry=https://registry.npm.taobao.org \   
--cache=$HOME/.npm/.cache/cnpm \   
--disturl=https://npm.taobao.org/dist \   
--userconfig=$HOME/.cnpmrc"' >> ~/.zshrc && source ~/.zshrc  
```

设置好别名后就可以

```  
cnpm install [name]  
express-generator安装  
cnpm install express-generator -g  
```  

express项目文件结构
-- -----------------------------
  
├── app.js  
├── bin  
│   └── www  
├── package.json  
├── node_modules  
├── public  
│   ├── images  
│   ├── javascripts  
│   └── stylesheets  
│       └── style.css  
├── routes  
│   ├── index.js  
│   └── users.js  
└── views  
    ├── error.jade  
    ├── index.jade  
    └── layout.jade  
  
7 directories   
-- -------------------------------------------

* bin，存放启动项目的脚本文件
* node_modules，项目所有依赖的库，以及存放 package.json 中安装的模块，当你在 package.json 添加依赖的模块并安装后，存放在这个文件夹下
* public，静态文件(css,js,img)
* routes，路由文件(MVC中的C,controller)
* views，页面文件(支持ejs、jade)
* package.json，存储着工程的信息及模块依赖
* app.js，应用核心配置文件（入口文件）

### 各种撸、各种造轮子、各种copy
完成下面几步，基本上程序这一块就结束了

* 博客数据库的建模，最简单也得要有两张表吧

-- ------------------------------------------  
--  Table structure for users and article  
-- ------------------------------------------  
SET NAMES utf8;  
SET FOREIGN_KEY_CHECKS= 0;  
CREATE TABLE users  
(  
  id smallint NOT NULL AUTO_INCREMENT,  
  user_name varchar(20) NOT NULL DEFAULT '' COMMENT '用户名',  
  user_pass varchar(255) NOT NULL DEFAULT '',  
  PRIMARY KEY(id)  
) DEFAULT CHARSET=utf8;  
  
CREATE TABLE article  
(  
  article_id smallint(5) NOT NULL AUTO_INCREMENT COMMENT '日志自增ID号',  
  article_name varchar(128) NOT NULL COMMENT '文章名称',  
  article_time datetime NOT NULL COMMENT '发布时间',  
  sort_article_id varchar(50) NOT NULL COMMENT '所属分类',  
  PRIMARY KEY(article_id)  
) DEFAULT CHARSET=utf8 ;   
-- -------------

* 配置app.js，涉及`mysql`的一些配置要提前读完文档，可以避免很多`坑`。
    * mysql基本用法，模糊搜索、DISTINCT、时间类型的控制......）不都是等着你跳的坑吗，不想造轮子，也有很多工具可以用。
    * 连接池的配置使用。
    * 回掉函数的有效使用。
    * `session`
* 配置好路由，多使用代码重用，不然后面出现一个bug，涉及到具体哦功能的每个路由都得改。
    * 理解http／https，在post／get时所做的工作，后续`web security`也是大坑，node.js在SQL注入方面并没有太有效的方案。）不过我也懒得去给我这个Blog来个渗透测试。
    * 要学会用`Express`API文档
* 撸好模版文件，同样做好代码重用，要熟悉ejs的基本语法。
    * `<%= code %>`
    * `<%- code %>`
    * `<% code %>`
* 优化页面友好度，我果断选择Bootstrap，不过还是很渣渣。
* 最后，在本地测试、调试。

### 在阿里云清亮应用服务器上部署
因为我选择的这款服务器，`nginx`已经配置好了，如果在本地没有问题了，用pm2启动一下，就可以像你这样看到你写的Blog了。

### 不断维护

* V 1.0
* V 1.1
* ......
* V 2.0
* V 2.1
* ......
* V 1024.0

#### 还是再把项目贴一下吧，哎，操碎了心
* [CaptainBlog](https://github.com/MrWang1214/CaptainBlog-1.0)。
