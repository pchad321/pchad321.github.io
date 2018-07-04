---
title: SCRAPY框架
date: 2018-07-04 14:44:15
modified:
categories:
- Spider
tags:
- Spider
- Scrapy
- data fetch
---

### Scrapy 框架

- Scrapy是用纯Python实现一个为了爬取网站数据、提取结构性数据而编写的应用框架，用途非常广泛。

- 框架的力量，用户只需要定制开发几个模块就可以轻松的实现一个爬虫，用来抓取网页内容以及各种图片，非常之方便。

- Scrapy使用了Twisted(其主要对手是Tornado)异步网络框架来处理网络通讯，可以加快我们的下载速度，不用自己去实现异步框架，并且包含了各种中间件接口，可以灵活的完成各种需求。  

#### Scrapy架构图

!["SCRAPY架构图"](/images/spider/scrapy-framework.png)  

- Scrapy Engine(引擎): 负责Spider、ItemPipeline、Downloader、Scheduler中间的通讯，信号、数据传递等。

- Scheduler(调度器): 它负责接受引擎发送过来的Request请求，并按照一定的方式进行整理排列，入队，当引擎需要时，交还给引擎。

- Downloader（下载器）：负责下载Scrapy Engine(引擎)发送的所有Requests请求，并将其获取到的Responses交还给Scrapy Engine(引擎)，由引擎交给Spider来处理，

- Spider（爬虫）：它负责处理所有Responses,从中分析提取数据，获取Item字段需要的数据，并将需要跟进的URL提交给引擎，再次进入Scheduler(调度器)，

- Item Pipeline(管道)：它负责处理Spider中获取到的Item，并进行进行后期处理（详细分析、过滤、存储等）的地方.

- Downloader Middlewares（下载中间件）：你可以当作是一个可以自定义扩展下载功能的组件。

- Spider Middlewares（Spider中间件）：你可以理解为是一个可以自定扩展和操作引擎和Spider中间通信的功能组件（比如进入Spider的Responses和从Spider出去的Requests）。  

#### 编写Scrapy爬虫的步骤

1. 新建项目(scrapy startproject xxx)：新建一个新的爬虫项目；  
2. 明确目标(编写items.py)：明确你想要抓取的目标；  
3. 制作爬虫(spiders/xxspider.py)：制作爬虫开始爬取网页；  
4. 存储内容(pipelines.py)：设计管道存储爬取内容。
  
### 入门实例

#### 新建项目(scrapy startproject)

1. 在开始爬取之前，必须创建一个新的Scrapy项目。进入自定义的项目目录中，运行下列命令：
   > scrapy startproject mySpider  


下面来简单介绍一下各个主要文件的作用：  

> scrapy.cfg：项目的配置文件；  
  mySpider/：项目的Python模块，将会从这里引用代码；  
  mySpider/items.py：项目的目标文件；  
  mySpider/pipelines.py：项目的管道文件；
  mySpider/settings.py：项目的设置文件；
  mySpider/spiders/：存储爬虫代码目录。
  
#### 明确目标(mySpider/items.py)

抓取的目标：https://hr.tencent.com/position.php?&start=0 腾讯社招网站里的所有职位的名称、工作地点、招聘人数、职位类别、工作职责以及工作要求。

1. 打开mySpider目录下的items.py；

2. Item定义结构化数据字段，用来保存爬取到的数据，有点像Python中的dict，但是提供了一些额外的保护减少错误。  

3. 可以通过创建一个scrapy.Item类，并且定义类型为scrapy.Field的类属性来定义一个Item(可以理解成类似于ORM的映射关系)。  

4. 接下来，创建一个TencentItem类，和构建item模型(model)。  

#### 制作爬虫(spiders/itcastSpider.py)

1. 爬数据  

2. 取数据  

### Scrapy Shell

Scrapy终端是一个交互终端，我们可以在未启动spider的情况下尝试及调试代码，也可以用来测试XPath或CSS表达式，查看他们的工作方式，方便我们爬取的网页中提取的数据。  

#### 启动Scrapy Shell

进入项目的根目录，执行下列命令来启动shell:

> scrapy shell "https://hr.tencent.com/position.php?&start=0"

Scrapy Shell根据下载的页面会自动创建一些方便使用的对象，例如Response对象，以及Selector对象(对HTML及XML内容)。  

- 当shell载入后，将得到一个包含response数据的本地response变量，输入response.body将输出response的响应体，输出response.headers可以看到response的响应头。  

- 输入response.selector时，将获取到一个response初始化的类Selector的对象，此时可以通过使用response.selector.xpath()或response.selector.css()来对response进行查询。

- Scrapy也提供了一些快捷方式，例如response.xpath()或response.css()同样可以生效。  

#### Selectors选择器

*Scrapy Selectors内置`XPath`和`CSS Selector`表达式机制*

Selector有四个基本的方法，最常用的还是xpath:

1. xpath(): 传入xpath表达式，返回该表达式所对应的所有节点的selector list列表；  
2. extract(): 序列化该节点为Unicode字符串并返回list；  
3. css(): 传入CSS表达式，返回该表达式所对应的所有节点的selector list列表，语法同BeautifulSoup4；  
4. re(): 根据传入的正则表达式对数据进行提取，返回Unicode字符串list列表。

### Item Pipeline

当Item在Spider中被收集之后，它将会被传递到Item Pipeline，这些Item Pipeline组件按定义的顺序处理Item。

每个Item Pipeline都是实现了简单方法的Python类，比如决定此Item是丢弃还是存储。以下是item pipeline的一些典型应用：

- 验证爬取的数据(检查item包含某些字段，比如说name字段)  
- 查重(并丢弃)  
- 将爬取结果保存到文件或者数据库中  

#### 编写item pipeline

item pipiline组件是一个独立的Python类，其中process_item()方法必须实现。

```python
import something

class SomethingPipeline(object):
    def __init__(self):    
        # 可选实现，做参数初始化等
        # doing something

    def process_item(self, item, spider):
        # item (Item 对象) – 被爬取的item
        # spider (Spider 对象) – 爬取该item的spider
        # 这个方法必须实现，每个item pipeline组件都需要调用该方法，
        # 这个方法必须返回一个 Item 对象，被丢弃的item将不会被之后的pipeline组件所处理。
        return item

    def open_spider(self, spider):
        # spider (Spider 对象) – 被开启的spider
        # 可选实现，当spider被开启时，这个方法被调用。

    def close_spider(self, spider):
        # spider (Spider 对象) – 被关闭的spider
        # 可选实现，当spider被关闭时，这个方法被调用
```

#### 数据写入json文件

##### item写入JSON文件

以下pipeline将所有(从所有'spider'中)爬取到的item，存储到一个独立地items.json 文件，每行包含一个序列化为JSON格式的item：  

```python
import json

class TencentJsonPipeline(object):

    def __init__(self):
        self.file = open('tencent.json', 'wb')

    def process_item(self, item, spider):
        content = json.dumps(dict(item), ensure_ascii=False) + "\n"
        self.file.write(content)
        return item

    def close_spider(self, spider):
        self.file.close()
```

##### 启用一个Item Pipeline组件

为了启用Item Pipeline组件，必须将它的类添加到settings.py文件ITEM_PIPELINES配置，例如：

```python
# Configure item pipelines
# See http://scrapy.readthedocs.org/en/latest/topics/item-pipeline.html
ITEM_PIPELINES = {
    #'mySpider.pipelines.SomePipeline': 300,
    "mySpider.pipelines.TencentJsonPipeline":300
}
```

*分配给每个类的整型值，确定了他们运行的顺序，item按数字从低到高的顺序，通过pipeline，通常将这些数字定义在0-1000范围内(0-1000随意设置，数值越低，组件的优先级越高)*  

### Spider

Spider类定义了如何爬取某个(或某些)网站。包括了爬取的动作(例如:是否跟进链接)以及如何从网页的内容中提取结构化数据(爬取item)。 换句话说，Spider就是定义爬取的动作及分析某个网页(或者是有些网页)的地方。  

> class scrapy.Spider是最基本的类，所有编写的爬虫必须继承这个类。

主要用到的函数及调用顺序为：

- `__init__()`：初始化爬虫名字和start_urls列表  
- `start_requests()`调用`make_requests_from url()`生成Requests对象交给Scrapy下载并返回response  
- `parse()`：解析response，并返回Item或Requests(需指定回调函数)。Item传给Item pipline持久化，而Requests交由Scrapy下载，并由指定的回调函数处理(默认`parse()`)，一直进行循环，直到处理完所有的数据为止。  

### CrawlSpiders