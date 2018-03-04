# python爬虫

网络爬虫，是一种按照一定的规则，自动的抓取万维网信息的程序或者脚本。

Python 爬虫开发流程我总结如下：

1. 分析网络交互信息
2. 设计爬虫结构、功能
3. 根据分析和设计进行开发
4. 测试和debug
5. 执行爬虫程序
6. 维护、改进、优化程序

`Software is grown,not built.`

根据我目前的进度，学习Python爬虫，共有以下几点：

## Python基础知识

我入门python的书是由Swaroop C H编写的《A Byte of Python》，在这里总结的东西是基于其他编程语言（C/java）基础，python中一些特性。

### 注释

尽可能的使用注释是python开发一个非常好的习惯，一方面开源给他人是一种便利，一方面对自己后面维护代码也很作用，注释要把握以下几个方面：

* 解释假设
* 说明重要的决定
* 解释重要的细节
* 说明你想要解决的问题
* 说明你想在程序中克服的问题

### 字符串是不可变的

所有字符串都是`str`类下的对象，python中没有单独的`char`数据类型，这区别于`C/C++`。

```Python
age = 24
name = Captain

print('{0} was {1} years old when he wrote this.'.format(name,age))

print('{} was {} years old when he wrote this.'.format(name,age))
```

数字是可选项，同时也可以看出python中字符串可以选择某些特定的格式。

### 变量（Variables）

* 定义规则
  * 第一个字符必须是字母表中的字符或下滑线
  * 标识符其他部分仅可以由字符、下划线、数字组成
  * 标识符名称区分大小写

* 数据类型

    通过`类（Classes）`来创建类型。

### 对象（Object）

Python中将任何内容统称为`对象（Object）`。

### 缩进

根据Python官方的建议，Python中使用四个空格作为缩进。

### 可变参数

定义的函数中可能需要任意数量的参数，可以通过`*`来表示。`*`代表的是元组，`*`代表的是字典。

```Python
    def total(a=5,*numbers,**phonebook)
        print('a',a)
        for single_item in numbers:
            print('single_item',single_item)
        for first_part,seconf_part in phonebook.items():
            print(first_part,seconf_part)
    print(total(10,1,2,3,Jack=121,Tom=212))
```

### 模块

用于代码重用，提高编码效率，开源利器。编写模块的方法：

* 创建一个包含函数变量、参数，以`.py`为后缀的文件。
* 使用撰写`Pythom解释器`本身的本地语言撰写模块，再通过`Pythom解释器`在你的代码中使用它们。

一个模块通过程序被其他程序导入并使用其函数功能，Python标准库同样需要导入。

* import `Modules`
* from `Modules` import `Func`

### 包（packages）

用以组织程序的层次结构，`__init__.py`向python表明这一文件夹包含了Python模块。

### 数据结构

用来存储一系列相关数据的集合

* 列表 List

  * `List.append` `List.sort`
  * 列表是可变的，字符串是不可变的

* 元组 Tuple
* 字典 Dictionary

  * `d = {key1 : value1 ,key2 : value2}`

* 集合 Set

### 面向对象编程

* Python中类`Class`、对象`Object`、实例`Instance`的概念。
* `__init__`方法，对目标对象进行初始化操作。
* 类变量和对象变量。
* 继承，代码重用

### 输入与输出

* `input`、`print`
* 文件`file类`，`read`、`readline`、`write`等等方法。
* `Pickle`标准模块，将对象存储到一个文件中去，并稍后取回，叫做持久地存储对象。

### 异常Exception

Python中的异常、处理异常与其他语言相似度很高。

```Python    
try ... except ...

try ... finally ...   
```

### 标准库

Python标准库包含了大量有用的模块，熟悉每个模块的用途，就已经能够解决很多事情。
学习Python标准库，可以通过阅读安装包中自带的`Library Reference`来查看模块的细节。

常用的模块：

* `sys`模块
* `logging`模块
* ......
