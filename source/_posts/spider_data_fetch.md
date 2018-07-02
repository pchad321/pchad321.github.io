---
title: 非结构化和结构化数据提取
date: 2018-06-26 17:44:15
modified:
categories:
- Spider
tags:
- Spider 
- data fetch
---

### 页面解析和数据提取

一般来讲对我们而言，需要抓取的是某个网站或者某个应用的内容，提取有用的价值。内容一般分为两部分，非结构化的数据和结构化的数据。  
- 非结构化数据：先有数据，再有结构  
- 结构化数据：先有结构、再有数据  
不同类型的数据，我们需要采用不同的方式来处理。

#### 非结构化的数据处理

文本、电话号码、邮箱地址
- 正则表达式

HTML 文件
- 正则表达式
- XPath
- CSS选择器

#### 结构化的数据处理

JSON 文件
- JSON Path
- 转化成Python类型进行操作（json类）

XML 文件
- 转化成Python类型（xmltodict）
- XPath
- CSS选择器
- 正则表达式

### 正则表达式

!["HTTP请求报文格式"](/images/spider/re_principle.png) 

#### 正则表达式匹配规则

!["HTTP请求报文格式"](/images/spider/re_match_rule.png) 

#### Python的re模块

在Python中，可以使用内置的re模块来使用正则表达式。  
有一点需要特别注意的是，正则表达式使用对特殊字符进行转义，所以如果我们要使用原始字符串，只需加一个 r 前缀，例如：

> r'chuanzhiboke\t\.\tpython'

##### re模块的一般使用步骤

1. 使用`compile()`函数将正则表达式的字符串形式编译为一个`Pattern`对象；  
2. 通过`Pattern`对象提供的一系列方法对文本进行匹配查找，获得匹配结果，一个`Match`对象；  
3. 最后使用`Match`对象提供的属性和方法获得信息，根据需要进行其他的操作。  

##### compile 函数

compile函数用于编译正则表达式，生成一个Pattern对象，它的一般使用形式如下：

```python
import re

# 将正则表达式编译成 Pattern 对象
pattern = re.compile(r'\d+')
```

在上面代码中，将一个正则表达式编译成Pattern对象，接下来，就可以利用pattern的一系列方法对文本进行匹配查找了。  

Pattern 对象的一些常用方法主要有：

> match 方法：从起始位置开始查找，一次匹配  
  search 方法：从任何位置开始查找，一次匹配  
  findall 方法：全部匹配，返回列表  
  finditer 方法：全部匹配，返回迭代器  
  split 方法：分割字符串，返回列表  
  sub 方法：替换  
  
##### match 方法

match方法用于查找字符串的头部（也可以指定起始位置），它是一次匹配，只要找到了一个匹配的结果就返回，而不是查找所有匹配的结果。它的一般使用形式如下：  
> match(string[, pos[, endpos]])

其中，string是待匹配的字符串，pos和endpos 是可选参数，指定字符串的起始和终点位置，默认值分别是0和len(字符串长度)。因此，当不指定pos和endpos时，match 方法默认匹配字符串的头部。  
当匹配成功时，返回一个Match对象，如果没有匹配上，则返回None。  

```commandline
>>> import re
>>> pattern = re.compile(r'\d+')  # 用于匹配至少一个数字

>>> m = pattern.match('one12twothree34four')  # 查找头部，没有匹配
>>> print(m)
None

>>> m = pattern.match('one12twothree34four', 2, 10) # 从'e'的位置开始匹配，没有匹配
>>> print(m)
None

>>> m = pattern.match('one12twothree34four', 3, 10) # 从'1'的位置开始匹配，正好匹配
>>> print(m)                                         # 返回一个 Match 对象
<_sre.SRE_Match object at 0x10a42aac0>

>>> m.group(0)   # 可省略 0
'12'
>>> m.start(0)   # 可省略 0
3
>>> m.end(0)     # 可省略 0
5
>>> m.span(0)    # 可省略 0
(3, 5)
```

在上面，当匹配成功时返回一个Match对象，其中：  
- group([group1, …])方法用于获得一个或多个分组匹配的字符串，当要获得整个匹配的子串时，可直接使用group()或group(0)；
- start([group])方法用于获取分组匹配的子串在整个字符串中的起始位置（子串第一个字符的索引），参数默认值为0；
- end([group])方法用于获取分组匹配的子串在整个字符串中的结束位置（子串最后一个字符的索引+1），参数默认值为0；
- span([group])方法返回(start(group), end(group))。  

再看看一个例子：

```commandline
>>> import re
>>> pattern = re.compile(r'([a-z]+) ([a-z]+)', re.I)  # re.I 表示忽略大小写
>>> m = pattern.match('Hello World Wide Web')

>>> print(m)     # 匹配成功，返回一个 Match 对象
<_sre.SRE_Match object at 0x10bea83e8>

>>> m.group(0)  # 返回匹配成功的整个子串
'Hello World'

>>> m.span(0)   # 返回匹配成功的整个子串的索引
(0, 11)

>>> m.group(1)  # 返回第一个分组匹配成功的子串
'Hello'

>>> m.span(1)   # 返回第一个分组匹配成功的子串的索引
(0, 5)

>>> m.group(2)  # 返回第二个分组匹配成功的子串
'World'

>>> m.span(2)   # 返回第二个分组匹配成功的子串
(6, 11)

>>> m.groups()  # 等价于 (m.group(1), m.group(2), ...)
('Hello', 'World')

>>> m.group(3)   # 不存在第三个分组
Traceback (most recent call last):
  File "<stdin>", line 1, in <module>
IndexError: no such group
```

##### search方法

`search`方法用于查找字符串的任何位置，它也是一次匹配，只要找到了一个匹配的结果就返回，而不是查找所有匹配的结果，它的一般使用形式如下：  

> search(string[, pos[, endpos]])

其中，string 是待匹配的字符串，pos和endpos是可选参数，指定字符串的起始和终点位置，默认值分别是0和len(字符串长度)。  
当匹配成功时，返回一个`Match`对象，如果没有匹配上，则返回None。

```commandline
>>> import re
>>> pattern = re.compile('\d+')
>>> m = pattern.search('one12twothree34four')  # 这里如果使用 match 方法则不匹配
>>> m
<_sre.SRE_Match object at 0x10cc03ac0>
>>> m.group()
'12'
>>> m = pattern.search('one12twothree34four', 10, 30)  # 指定字符串区间
>>> m
<_sre.SRE_Match object at 0x10cc03b28>
>>> m.group()
'34'
>>> m.span()
(13, 15)
```

##### findall方法

上面的match和search方法都是一次匹配，只要找到了一个匹配的结果就返回。然而，在大多数时候，我们需要搜索整个字符串，获得所有匹配的结果。  
findall方法的使用形式如下：

> findall(string[, pos[, endpos]])

其中，string是待匹配的字符串，pos和endpos是可选参数，指定字符串的起始和终点位置，默认值分别是0和len(字符串长度)。  
findall以列表形式返回全部能匹配的子串，如果没有匹配，则返回一个空列表。  

```python
import re
pattern = re.compile(r'\d+')   # 查找数字

result1 = pattern.findall('hello 123456 789')
result2 = pattern.findall('one1two2three3four4', 0, 10)

print(result1)
print(result2)
```

##### finditer方法

finditer方法的行为跟findall的行为类似，也是搜索整个字符串，获得所有匹配的结果。但它返回一个顺序访问每一个匹配结果（Match 对象）的迭代器。

##### split方法

split方法按照能够匹配的子串将字符串分割后返回列表，它的使用形式如下：  

> split(string[, maxsplit])

其中，maxsplit用于指定最大分割次数，不指定将全部分割。

```python
import re
p = re.compile(r'[\s\,\;]+')
print(p.split('a,b;; c   d'))
```

##### sub方法

sub方法用于替换。它的使用形式如下：  

> sub(repl, string[, count])

其中，repl可以是字符串也可以是一个函数：  
- 如果repl是字符串，则会使用repl去替换字符串每一个匹配的子串，并返回替换后的字符串，另外，repl还可以使用id的形式来引用分组，但不能使用编号0；
- 如果repl是函数，这个方法应当只接受一个参数（Match 对象），并返回一个字符串用于替换（返回的字符串中不能再引用分组）。
- count用于指定最多替换次数，不指定时全部替换。

```python
import re
p = re.compile(r'(\w+) (\w+)') # \w = [A-Za-z0-9]
s = 'hello 123, hello 456'

print(p.sub(r'hello world', s))  # 使用 'hello world' 替换 'hello 123' 和 'hello 456'
print(p.sub(r'\2 \1', s))        # 引用分组

def func(m):
    return 'hi' + ' ' + m.group(2)

print(p.sub(func, s))
print(p.sub(func, s, 1))         # 最多替换一次
```

##### 匹配中文

在某些情况下，需要匹配文本中的汉字，有一点需要注意的是，中文的unicode编码范围 主要在[u4e00-u9fa5]，这里说主要是因为这个范围并不完整，比如没有包括全角（中文）标点，不过，在大部分情况下，应该是够用的。  

假设现在想把字符串title = u'你好，hello，世界'中的中文提取出来，可以：

```python
import re

title = u'你好，hello，世界'
pattern = re.compile(ur'[\u4e00-\u9fa5]+')
result = pattern.findall(title)

print(result)
```

##### 贪婪模式与非贪婪模式

- 贪婪模式：在整个表达式匹配成功的前提下，尽可能多的匹配( * )；
- 非贪婪模式：在整个表达式匹配成功的前提下，尽可能少的匹配( ? )；
- Python里数量词默认是贪婪的。

#### 使用正则表达式的爬虫

使用正则表达式爬去腾讯社招的信息：[腾讯社招](https://hr.tencent.com/position.php)



### XPath与lxml库

XPath(XML Path Language)是一门在XML文档中查找信息的语言，可用来在XML文档中对元素和属性进行遍历。

#### XPath开发工具

- 开源的XPath表达式编辑工具:XMLQuire(XML格式文件可用)
- Chrome插件 XPath Helper
- Firefox插件 XPath Checker

#### 选取节点

XPath使用路径表达式来选取XML文档中的节点或者节点集。这些路径表达式和我们在常规的电脑文件系统中看到的表达式非常相似。

最常用的路径表达式如下：  

|表达式|描述|
|----|:---|
|nodename|选取此节点的所有子节点|
|/|从根节点选取|
|//|从匹配选择的当前节点选择文档中的节点，而不考虑它们的位置|
|.|选取当前节点|
|..|选取当前节点的父节点|
|@|选取属性|

#### 谓语（Predicates）

谓语用来查找某个特定的节点或者包含某个指定的值的节点，被嵌在方括号中。  
在下面的表格中，列出了带有谓语的一些路径表达式，以及表达式的结果：

|路径表达式|结果|
|----|:----|
|/bookstore/book[1]|选取属于 bookstore 子元素的第一个 book 元素。
|/bookstore/book[last()]|选取属于 bookstore 子元素的最后一个 book 元素。
|/bookstore/book[last()-1]|选取属于 bookstore 子元素的倒数第二个 book 元素。
|/bookstore/book[position()<3]|选取最前面的两个属于 bookstore 元素的子元素的 book 元素。
|//title[@lang]|选取所有拥有名为 lang 的属性的 title 元素。
|//title[@lang=’eng’]|选取所有 title 元素，且这些元素拥有值为 eng 的 lang 属性。
|/bookstore/book[price>35.00]|选取 bookstore 元素的所有 book 元素，且其中的 price 元素的值须大于 35.00。
|/bookstore/book[price>35.00]/title|选取 bookstore 元素中的 book 元素的所有 title 元素，且其中的 price 元素的值须大于 35.00。

#### 选取未知节点

XPath通配符可用来选取未知的XML元素。

|通配符|描述|
|----|:----|
|*|匹配任何元素节点。
|@*|匹配任何属性节点。
|node()|匹配任何类型的节点。

#### 选取若干路径

通过在路径表达式中使用`|`运算符，可以选取若干个路径。  
在下面的表格中，列出了一些路径表达式，以及这些表达式的结果：

|路径表达式|结果|
|----|:----|
|//book/title &#124; //book/price|选取 book 元素的所有 title 和 price 元素。
|//title &#124; //price|选取文档中的所有 title 和 price 元素。
|/bookstore/book/title &#124; //price|选取属于 bookstore 元素的 book 元素的所有 title 元素，以及文档中所有的 price 元素。

#### lxml库

> lxml是一个HTML/XML的解析器，主要的功能是如何解析和提取 HTML/XML 数据。  
  lxml和正则一样，也是用 C 实现的，是一款高性能的 Python HTML/XML 解析器，可以利用XPath语法，来快速的定位特定元素以及节点信息。  
  
#### 使用xpath获取腾讯社招每个职位的详细信息

```python
from urllib import request
from lxml import etree
import json
import codecs


class TencentSpider(object):
    def __init__(self, file, beginPage, endPage):
        self.url = "https://hr.tencent.com/position.php?&start="
        self.headers = {
            "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
        }
        self.beginPage = beginPage
        self.endPage = endPage
        self.file = file
        self.positionJson = []

    def start_spider(self):
        for page in range(self.beginPage, self.endPage + 1):
            offsetNumber = (page - 1) * 10
            fullUrl = self.url + str(offsetNumber)
            self.load_page(fullUrl)

        self.file.write(json.dumps(self.positionJson, ensure_ascii=False))

    def load_page(self, url):
        req = request.Request(url, headers=self.headers)
        res = request.urlopen(req)
        html = etree.HTML(res.read())

        links = html.xpath('//div[@class="left wcont_b box"]//tr[@class="even" or "odd"]/td[1]/a/@href')
        for link in links:
            fullLink = "https://hr.tencent.com/" + link
            self.get_position_detail(fullLink)

    def get_position_detail(self, url):
        req = request.Request(url, headers=self.headers)
        res = request.urlopen(req)
        html = etree.HTML(res.read())

        positionDetail = {}
        # 职位名称
        positionDetail['positionName'] = html.xpath('//div[@class="box wcont_a"]/table//tr[1]/td/text()')[0]
        # 工作地点
        positionDetail['positionAddress'] = html.xpath('//div[@class="box wcont_a"]/table//tr[2]/td[1]/text()')[0]
        # 职位类别
        if(html.xpath('//div[@class="box wcont_a"]/table//tr[2]/td[2]/text()')):
            positionDetail['positionCategory'] = html.xpath('//div[@class="box wcont_a"]/table//tr[2]/td[2]/text()')[0]
        else:
            positionDetail['positionCategory'] = ""
        # 招聘人数
        positionDetail['postionNumber'] = html.xpath('//div[@class="box wcont_a"]/table//tr[2]/td[3]/text()')[0]
        # 工作职责
        positionDutyList = html.xpath('//div[@class="box wcont_a"]/table//tr[3]/td/ul//li')
        positionDutyStr = ""

        for duty in positionDutyList:
            positionDutyStr += duty.text
            # print(positionDutyStr)

        positionDetail['positionDutyStr'] = positionDutyStr

        # 工作要求
        positionRequirementList = html.xpath('//div[@class="box wcont_a"]/table//tr[4]/td/ul//li')
        positionRequirementStr = ""

        for requirement in positionRequirementList:
            positionRequirementStr += requirement.text
            # print(positionRequirementStr)

        positionDetail['positionRequirementStr'] = positionRequirementStr

        self.positionJson.append(positionDetail)


if __name__ == "__main__":
    beginPage = int(input("请输入起始页:"))
    endPage = int(input("请输入终止页"))
    file = codecs.open("tencentposition.json", "w", "utf-8")
    tencentspider = TencentSpider(file, beginPage, endPage)
    tencentspider.start_spider()
    file.close()
```

### 数据提取之JSON与JsonPATH

JSON(JavaScript Object Notation)是一种轻量级的数据交换格式，它使得人们很容易的进行阅读和编写。同时也方便了机器进行解析和生成。适用于进行数据交互的场景，比如网站前台与后台之间的数据交互。

#### JSON

json简单说就是javascript中的对象和数组，所以这两种结构就是对象和数组两种结构，通过这两种结构可以表示各种复杂的结构。  

>  对象：对象在js中表示为`{ }`括起来的内容，数据结构为`{ key：value, key：value, ... }`的键值对的结构，在面向对象的语言中，key为对象的属性，value为对应的属性值，所以很容易理解，取值方法为 对象.key 获取属性值，这个属性值的类型可以是数字、字符串、数组、对象这几种。  
   数组：数组在js中是中括号`[ ]`括起来的内容，数据结构为`["Python", "javascript", "C++", ...]`，取值方式和所有语言中一样，使用索引获取，字段值的类型可以是 数字、字符串、数组、对象几种。
   
#### json模块

json模块提供了四个功能：dumps、dump、loads、load，用于字符串和python数据类型间进行转换。

##### json.loads()

把Json格式字符串解码转换成Python对象 从json到python的类型转化对照如下：

|Python|JSON|
|:----|----|
|dict|object|
|list, tuple|array|
|str, unicode|string|
|int, long, float|number|
|True|true|
|False|false|
|None|null|

##### json.dumps()

实现python类型转化为json字符串，返回一个str对象 把一个Python对象编码转换成Json字符串。  

##### json.dump()

将Python内置类型序列化为json对象后写入文件。  

```python
import json

listStr = [{"city": "北京"}, {"name": "大刘"}]
json.dump(listStr, open("listStr.json","w"), ensure_ascii=False)

dictStr = {"city": "北京", "name": "大刘"}
json.dump(dictStr, open("dictStr.json","w"), ensure_ascii=False)
```

##### json.load()

读取文件中json形式的字符串元素转化成python类型。

```python
import json

strList = json.load(open("listStr.json"))
print(strList)
```

#### JsonPath

JsonPath是一种信息抽取类库，是从JSON文档中抽取指定信息的工具，提供多种语言实现版本，包括：Javascript、Python、PHP和Java。  
JsonPath对于JSON来说，相当于XPATH对于XML。

##### JsonPath与XPath语法对比

XPath|	JSONPath|	描述
:---:|:---|----
/|	$|	根节点
.|	@|	现行节点
/|	.or[]|	取子节点
..|	n/a|	取父节点，Jsonpath未支持
//|	..|	就是不管位置，选择所有符合条件的条件
*|	*|	匹配所有元素节点
@|	n/a|	根据属性访问，Json不支持，因为Json是个Key-value递归结构，不需要。
[]|	[]|	迭代器标示（可以在里边做简单的迭代操作，如数组下标，根据内容选值等）
&#124;|	[,]|	支持迭代器中做多选。
[]|	?()|	支持过滤操作.
n/a|	()|	支持表达式计算
()|	n/a|	分组，JsonPath不支持

##### 注意事项

json.loads()是把Json格式字符串解码转换成Python对象，如果在json.loads的时候出错，要注意被解码的Json字符的编码。  
如果传入的字符串的编码不是UTF-8的话，需要指定字符编码的参数encoding：

```python
import json
dataDict = json.loads(jsonStrGBK, encoding="GBK");
```

- decode的作用是将其他编码的字符串转换成 Unicode 编码；  
- encode的作用是将 Unicode 编码转换成其他编码的字符串。  

*一句话：UTF-8是对Unicode字符集进行编码的一种编码方式。*  

### 多线程爬虫实例

#### Queue（队列对象）

Queue是python中的标准库，可以直接import Queue引用，队列是线程间最常用的交换数据的形式。  

*python下多线程的思考*  
对于资源，加锁是个重要的环节。因为python原生的list,dict等，都是not thread safe的。而Queue是线程安全的，因此在满足使用条件下，建议使用队列。

1. 初始化：class Queue.Queue(maxsize)  FIFO先进先出  
2. 包中的常用方法:  
    - Queue.qsize() 返回队列的大小；  
    - Queue.empty() 如果队列为空，返回True，反之False；  
    - Queue.full() 如果队列满了，返回True，反之False；  
    - Queue.full与maxsize大小对应；
    - Queue.get([block[, timeout]])获取队列，其中timeout为等待时间。  
3. 创建一个“队列”对象  
    - import Queue  
    - myqueue = Queue.Queue(maxsize=10)  
4. 将一个值放入队列中  
    myqueue.put(10)  
5. 将一个值从队列中取出  
    myqueue.get()  

#### 多线程示意图

!["多线程示意图"](/images/spider/multi_thread_spider.png)

```python

```