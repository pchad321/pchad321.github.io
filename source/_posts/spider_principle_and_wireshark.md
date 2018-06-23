---
title: 爬虫原理与数据抓取
date: 2018-06-123 09:44:15
modified:
categories:
- Spider
tags:
- Spider 
---

### 通用爬虫和聚焦爬虫
根据使用场景，网络爬虫可分为通用爬虫和聚焦爬虫两种。

#### 通用爬虫
通用网络爬虫是搜索引擎抓取系统的重要组成部分。主要目的是将互联网上的网页下载到本地，形成一个互联网内容的镜像备份。

##### 通用搜索引擎的工作原理
通用网络爬虫 从互联网中搜集网页，采集信息，这些网页信息用于为搜索引擎建立索引从而提供支持，它决定着整个引擎系统的内容是否丰富，信息是否即时，因此其性能的优劣直接影响着搜索引擎的效果。

第一步：抓取网页
搜索引擎网络爬虫的基本工作流程如下：
1. 首先选取一部分的种子URL 

1. 将这些URL放入待抓取URL队列；
2. 取出待抓取URL，解析DNS得到主机的IP，并将URL对应的网页下载下来，存储进已下载网页库中，并且将这些URL放进已抓取URL队列；
3. 分析已抓取URL队列中的URL，分析其中的其他URL，并且将URL放入待抓取URL队列，从而进入下一个循环。

但是搜索引擎蜘蛛的爬行是被输入了一定的规则的，它需要遵从一些命令或文件的内容，如标注为nofollow的链接，或者是Robots协议。
> Robots协议（也叫爬虫协议、机器人协议等），全称是“网络爬虫排除标准”（Robots Exclusion Protocol），网站通过Robots协议告诉搜索引擎哪些页面可以抓取，哪些页面不能抓取，例如：  
  淘宝网：https://www.taobao.com/robots.txt  
  腾讯网： http://www.qq.com/robots.txt
  
第二步：数据存储
搜索引擎通过爬虫爬取到的网页，将数据存入原始页面数据库。其中的页面数据与用户浏览器得到的HTML是完全一样的。  
搜索引擎蜘蛛在抓取页面时，也做一定的重复内容检测，一旦遇到访问权重很低的网站上有大量抄袭、采集或者复制的内容，很可能就不再爬行。

第三步：预处理
搜索引擎将爬虫抓取回来的页面，进行各种步骤的预处理。

- 提取文字
- 中文分词
- 消除噪音（比如版权声明文字、导航条、广告等……）
- 索引处理
- 链接关系计算
- 特殊文件处理
- ...

除了HTML文件外，搜索引擎通常还能抓取和索引以文字为基础的多种文件类型，如 PDF、Word、WPS、XLS、PPT、TXT 文件等。我们在搜索结果中也经常会看到这些文件类型。  
但搜索引擎还不能处理图片、视频、Flash 这类非文字内容，也不能执行脚本和程序。

第四步：提供检索服务，网站排名
搜索引擎在对信息进行组织和处理后，为用户提供关键字检索服务，将用户检索相关的信息展示给用户。  
同时会根据页面的PageRank值（链接的访问量排名）来进行网站排名，这样Rank值高的网站在搜索结果中会排名较前，当然也可以直接使用 Money 购买搜索引擎网站排名，简单粗暴。

**但是，这些通用性搜索引擎也存在着一定的局限性：**
1. 通用搜索引擎所返回的结果都是网页，而大多情况下，网页里90%的内容对用户来说都是无用的。
2. 不同领域、不同背景的用户往往具有不同的检索目的和需求，搜索引擎无法提供针对具体某个用户的搜索结果。
3. 万维网数据形式的丰富和网络技术的不断发展，图片、数据库、音频、视频多媒体等不同数据大量出现，通用搜索引擎对这些文件无能为力，不能很好地发现和获取。
4. 通用搜索引擎大多提供基于关键字的检索，难以支持根据语义信息提出的查询，无法准确理解用户的具体需求。

#### 聚焦爬虫
聚焦爬虫，是"面向特定主题需求"的一种网络爬虫程序，它与通用搜索引擎爬虫的区别在于：***聚焦爬虫在实施网页抓取时会对内容进行处理筛选，尽量保证只抓取与需求相关的网页信息***。

### HTTP/HTTPS的请求与相应

#### HTTP和HTTPS
HTTP协议（HyperText Transfer Protocol，超文本传输协议）：是一种发布和接收 HTML页面的方法。  
HTTPS（Hypertext Transfer Protocol over Secure Socket Layer）简单讲是HTTP的安全版，在HTTP下加入SSL层。  
SSL（Secure Sockets Layer 安全套接层）主要用于Web的安全传输协议，在传输层对网络连接进行加密，保障在Internet上数据传输的安全。  
> HTTP的端口号为80  
  HTTPS的端口号为443
  
#### HTTP的请求与响应
HTTP通信由两部分组成：*客户端请求消息*与*服务器响应消息*。

##### 浏览器发送HTTP请求的过程：
当用户在浏览器的地址栏中输入一个URL并按回车键之后，浏览器会向HTTP服务器发送HTTP请求。HTTP请求主要分为“Get”和“Post”两种方法。  
当我们在浏览器输入URL http://www.baidu.com 的时候，浏览器发送一个Request请求去获取 http://www.baidu.com 的html文件，服务器把Response文件对象发送回给浏览器。  
浏览器分析Response中的 HTML，发现其中引用了很多其他文件，比如Images文件，CSS文件，JS文件。 浏览器会自动再次发送Request去获取图片，CSS文件，或者JS文件。  
当所有的文件都下载成功后，网页会根据HTML语法结构，完整的显示出来了。  
URL（Uniform / Universal Resource Locator的缩写）：统一资源定位符，是用于完整地描述Internet上网页和其他资源的地址的一种标识方法。  

基本格式：
```
scheme://host[:port#]/path/…/[?query-string][#anchor]
```
- scheme：协议(例如：http, https, ftp)
- host：服务器的IP地址或者域名
- port#：服务器的端口（如果是走协议默认端口，缺省端口80）
- path：访问资源的路径
- query-string：参数，发送给http服务器的数据
- anchor：锚（跳转到网页的指定锚点位置）

#### 客户端HTTP请求
URL只是标识资源的位置，而HTTP是用来提交和获取资源。客户端发送一个HTTP请求到服务器的请求消息，包括以下格式：  
> 请求行、请求头部、空行、请求数据

!["HTTP请求报文格式"](/images/spider/client_request_format.png)  

一个典型的HTTP请求示例：
```
GET https://www.baidu.com/ HTTP/1.1
Host: www.baidu.com
Connection: keep-alive
Upgrade-Insecure-Requests: 1
User-Agent: Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/54.0.2840.99 Safari/537.36
Accept: text/html,application/xhtml+xml,application/xml;q=0.9,image/webp,*/*;q=0.8
Referer: http://www.baidu.com/
Accept-Encoding: gzip, deflate, sdch, br
Accept-Language: zh-CN,zh;q=0.8,en;q=0.6
Cookie: BAIDUID=04E4001F34EA74AD4601512DD3C41A7B:FG=1; BIDUPSID=04E4001F34EA74AD4601512DD3C41A7B; PSTM=1470329258; MCITY=-343%3A340%3A; BDUSS=nF0MVFiMTVLcUh-Q2MxQ0M3STZGQUZ4N2hBa1FFRkIzUDI3QlBCZjg5cFdOd1pZQVFBQUFBJCQAAAAAAAAAAAEAAADpLvgG0KGyvLrcyfrG-AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAFaq3ldWqt5XN; H_PS_PSSID=1447_18240_21105_21386_21454_21409_21554; BD_UPN=12314753; sug=3; sugstore=0; ORIGIN=0; bdime=0; H_PS_645EC=7e2ad3QHl181NSPbFbd7PRUCE1LlufzxrcFmwYin0E6b%2BW8bbTMKHZbDP0g; BDSVRTM=0
```

##### HTTP请求主要分为Get和Post两种方法
GET是从服务器上获取数据，POST是向服务器传送数据。  
GET请求参数显示，都显示在浏览器网址上，HTTP服务器根据该请求所包含URL中的参数来产生响应内容，即“Get”请求的参数是URL的一部分。 例如： http://www.baidu.com/s?wd=Chinese  
POST请求参数在请求体当中，消息长度没有限制而且以隐式的方式进行发送，通常用来向HTTP服务器提交量比较大的数据（比如请求中包含许多参数或者文件上传操作等），请求的参数包含在“Content-Type”消息头里，指明该消息体的媒体类型和编码。  
*注意：避免使用Get方式提交表单，因为有可能会导致安全问题。 比如说在登陆表单中用Get方式，用户输入的用户名和密码将在地址栏中暴露无遗。*  

##### 常用的请求报头
1. Host (主机和端口号)  
Host：对应网址URL中的Web名称和端口号，用于指定被请求资源的Internet主机和端口号，通常属于URL的一部分。

2. Connection (链接类型)  
Connection：表示客户端与服务连接类型  
    1. Client 发起一个包含 Connection:keep-alive 的请求，HTTP/1.1使用 keep-alive 为默认值。  
    2. Server收到请求后：  
       如果 Server 支持 keep-alive，回复一个包含 Connection:keep-alive 的响应，不关闭连接；  
       如果 Server 不支持 keep-alive，回复一个包含 Connection:close 的响应，关闭连接；  
    3. 如果client收到包含 Connection:keep-alive 的响应，向同一个连接发送下一个请求，直到一方主动关闭连接。  

    *keep-alive在很多情况下能够重用连接，减少资源消耗，缩短响应时间，比如当浏览器需要多个文件时(比如一个HTML文件和相关的图形文件)，不需要每次都去请求建立连接。*  

3. Upgrade-Insecure-Requests (升级为HTTPS请求)  
Upgrade-Insecure-Requests：升级不安全的请求，意思是会在加载 http 资源时自动替换成 https 请求，让浏览器不再显示https页面中的http请求警报。  
HTTPS 是以安全为目标的 HTTP 通道，所以在 HTTPS 承载的页面上不允许出现 HTTP 请求，一旦出现就是提示或报错。  

4. User-Agent (浏览器名称)  
User-Agent：是客户浏览器的名称。

5. Accept (传输文件类型)  
Accept：指浏览器或其他客户端可以接受的MIME（Multipurpose Internet Mail Extensions（多用途互联网邮件扩展））文件类型，服务器可以根据它判断并返回适当的文件格式。  
    > Accept: */*：表示什么都可以接收；  
      Accept：image/gif：表明客户端希望接受GIF图像格式的资源；  
      Accept：text/html：表明客户端希望接受html文本;  
      Accept: text/html, application/xhtml+xml;q=0.9, image/*;q=0.8：表示浏览器支持的 MIME 类型分别是 html文本、xhtml和xml文档、所有的图像格式资源。  
  
    *q是权重系数，范围 0 =< q <= 1，q 值越大，请求越倾向于获得其“;”之前的类型表示的内容。若没有指定q值，则默认为1，按从左到右排序顺序；若被赋值为0，则用于表示浏览器不接受此内容类型。*  
    *Text：用于标准化地表示的文本信息，文本消息可以是多种字符集和或者多种格式的；Application：用于传输应用程序数据或者二进制数据。*  

6. Referer(页面跳转处)  
Referer：表明产生请求的网页来自于哪个URL，用户是从该 Referer页面访问到当前请求的页面。这个属性可以用来跟踪Web请求来自哪个页面，是从什么网站来的等。

   有时候遇到下载某网站图片，需要对应的referer，否则无法下载图片，那是因为人家做了防盗链，原理就是根据referer去判断是否是本网站的地址，如果不是，则拒绝，如果是，就可以下载；

7. Accept-Encoding（文件编解码格式）  
Accept-Encoding：指出浏览器可以接受的编码方式。编码方式不同于文件格式，它是为了压缩文件并加速文件传递速度。浏览器在接收到Web响应之后先解码，然后再检查文件格式，许多情形下这可以减少大量的下载时间。  

   >Accept-Encoding:gzip;q=1.0, identity; q=0.5, *;q=0  
    如果有多个Encoding同时匹配, 按照q值顺序排列，本例中按顺序支持 gzip, identity压缩编码，支持gzip的浏览器会返回经过gzip编码的HTML页面。 如果请求消息中没有设置这个域服务器假定客户端对各种内容编码都可以接受。

8. Accept-Language（语言种类）  
Accept-Langeuage：指出浏览器可以接受的语言种类，如en或en-us指英语，zh或者zh-cn指中文，当服务器能够提供一种以上的语言版本时要用到。

9. Accept-Charset（字符编码）  
Accept-Charset：指出浏览器可以接受的字符编码。

   >举例：Accept-Charset:iso-8859-1,gb2312,utf-8  
    - ISO8859-1：通常叫做Latin-1。Latin-1包括了书写所有西方欧洲语言不可缺少的附加字符，英文浏览器的默认值是ISO-8859-1.
    - gb2312：标准简体中文字符集;  
    - utf-8：UNICODE 的一种变长字符编码，可以解决多种语言文本显示问题，从而实现应用国际化和本地化。
    如果在请求消息中没有设置这个域，缺省是任何字符集都可以接受。  

10. Cookie （Cookie）  
Cookie：浏览器用这个属性向服务器发送Cookie。Cookie是在浏览器中寄存的小型数据体，它可以记载和服务器相关的用户信息，也可以用来实现会话功能，以后会详细讲。

11. Content-Type (POST数据类型)  
Content-Type：POST请求里用来表示的内容类型。

    >Content-Type = Text/XML; charset=gb2312：  
     指明该请求的消息体中包含的是纯文本的XML类型的数据，字符编码采用“gb2312”。
     
#### 服务端HTTP响应
HTTP响应也由四个部分组成，分别是： 状态行、消息报头、空行、响应正文

```
HTTP/1.1 200 OK
Server: Tengine
Connection: keep-alive
Date: Wed, 30 Nov 2016 07:58:21 GMT
Cache-Control: no-cache
Content-Type: text/html;charset=UTF-8
Keep-Alive: timeout=20
Vary: Accept-Encoding
Pragma: no-cache
X-NWS-LOG-UUID: bd27210a-24e5-4740-8f6c-25dbafa9c395
Content-Length: 180945

<!DOCTYPE html PUBLIC "-//W3C//DTD XHTML 1.0 Transitional//EN" ....
```

##### 常用的响应报头  
理论上所有的响应头信息都应该是回应请求头的。但是服务端为了效率，安全，还有其他方面的考虑，会添加相对应的响应头信息：

1. Cache-Control：must-revalidate, no-cache, private。   
这个值告诉客户端，服务端不希望客户端缓存资源，在下次请求资源时，必须要从新请求服务器，不能从缓存副本中获取资源。  
    - Cache-Control是响应头中很重要的信息，当客户端请求头中包含Cache-Control:max-age=0请求，明确表示不会缓存服务器资源时,Cache-Control作为作为回应信息，通常会返回no-cache，意思就是说，"那不缓存了"。
    - 当客户端在请求头中没有包含Cache-Control时，服务端往往会定,不同的资源不同的缓存策略，比如说oschina在缓存图片资源的策略就是Cache-Control：max-age=86400,这个意思是，从当前时间开始，在86400秒的时间内，客户端可以直接从缓存副本中读取资源，而不需要向服务器请求。  

2. Connection：keep-alive  
这个字段作为回应客户端的Connection：keep-alive，告诉客户端服务器的tcp连接也是一个长连接，客户端可以继续使用这个tcp连接发送http请求。  

3. Content-Encoding:gzip  
告诉客户端，服务端发送的资源是采用gzip编码的，客户端看到这个信息后，应该采用gzip对资源进行解码。  

4. Content-Type：text/html;charset=UTF-8  
告诉客户端，资源文件的类型，还有字符编码，客户端通过utf-8对资源进行解码，然后对资源进行html解析。如果我们看到有些网站是乱码的，往往就是服务器端没有返回正确的编码。  

5. Date：Sun, 21 Sep 2016 06:18:21 GMT    
这个是服务端发送资源时的服务器时间，GMT是格林尼治所在地的标准时间。http协议中发送的时间都是GMT的，这主要是解决在互联网上，不同时区在相互请求资源的时候，时间混乱问题。

6. Expires:Sun, 1 Jan 2000 01:00:00 GMT  
这个响应头也跟缓存有关，告诉客户端在这个时间前，可以直接访问缓存副本，很显然这个值会存在问题，因为客户端和服务器的时间不一定会都是相同的，如果时间不同就会导致问题。所以这个响应头是没有Cache-Control：max-age=*这个响应头准确的，因为max-age=date中的date是个相对时间，不仅更好理解，也更准确。

7. Pragma:no-cache  
这个含义与Cache-Control等同。

8. Server：Tengine/1.4.6  
这个是服务器和相对应的版本，只是告诉客户端服务器的信息。

9. Transfer-Encoding：chunked  
这个响应头告诉客户端，服务器发送的资源的方式是分块发送的。一般分块发送的资源都是服务器动态生成的，在发送时还不知道发送资源的大小，所以采用分块发送，每一块都是独立的，独立的块都能标示自己的长度，最后一块是0长度的，当客户端读到这个0长度的块时，就可以确定资源已经传输完了。

10. Vary: Accept-Encoding  
告诉缓存服务器，缓存压缩文件和非压缩文件两个版本，现在这个字段用处并不大，因为现在的浏览器都是支持压缩的。  

##### 响应状态码
响应状态代码有三位数字组成，第一个数字定义了响应的类别，且有五种可能取值。  
>常见状态码：  
- 100~199：表示服务器成功接收部分请求，要求客户端继续提交其余请求才能完成整个处理过程。
- 200~299：表示服务器成功接收请求并已完成整个处理过程。常用200（OK 请求成功）。
- 300~399：为完成请求，客户需进一步细化请求。例如：请求的资源已经移动一个新地址、常用302（所请求的页面已经临时转移至新的url）、307和304（使用缓存资源）。
- 400~499：客户端的请求有错误，常用404（服务器无法找到被请求的页面）、403（服务器拒绝访问，权限不够）。
- 500~599：服务器端出现错误，常用500（请求未完成。服务器遇到不可预知的情况）。  

#### Cookie 和 Session：
服务器和客户端的交互仅限于请求/响应过程，结束之后便断开，在下一次请求时，服务器会认为新的客户端。  
为了维护他们之间的链接，让服务器知道这是前一个用户发送的请求，必须在一个地方保存客户端的信息。  
*Cookie*：通过在 客户端 记录的信息确定用户的身份。  
*Session*：通过在 服务器端 记录的信息确定用户的身份。 
 
### HTTP代理神器Fiddler


### urllib库的基本使用

所谓网页抓取，就是把URL地址中指定的网络资源从网络流中读取出来，保存到本地。 在Python3中一般使用urllib库。

#### urlopen和Request

可以直接使用urlopen()打开一个网页；但是，如果需要执行更复杂的操作，比如增加HTTP报头，必须创建一个 Request 实例来作为urlopen()的参数；而需要访问的url地址则作为 Request 实例的参数。  

```python
from urllib import request

def get_page_info():
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
    }

    req = request.Request("http://www.baidu.com/", headers=headers)
    res = request.urlopen(req)
    html = res.read()
    # 请求页面的源代码
    print(html)
    # 请求的状态码
    print(res.getcode())
    # 请求的链接
    print(res.geturl())
    # 请求头信息
    print(res.info())

if __name__ == "__main__":
    get_page_info()
```

> 新建Request实例，除了必须要有 url 参数之外，还可以设置另外两个参数：  
  - data（默认空）：是伴随 url 提交的数据（比如要post的数据），同时 HTTP 请求将从 "GET"方式 改为 "POST"方式。  
  - headers（默认空）：是一个字典，包含了需要发送的HTTP报头的键值对。  
  
#### User-Agent
浏览器就是互联网世界上公认被允许的身份，如果我们希望我们的爬虫程序更像一个真实用户，那我们第一步，就是需要伪装成一个被公认的浏览器。用不同的浏览器在发送请求的时候，会有不同的User-Agent头。 urllib默认的User-Agent头为：Python-urllib/x.y（x和y是Python主版本和次版本号）

#### 添加更多的Header信息
在 HTTP Request 中加入特定的 Header，来构造一个完整的HTTP请求消息。
- 可以通过调用Request.add_header() 添加/修改一个特定的header 
- 也可以通过调用Request.get_header()来查看已有的header。

#### parse.urlencode()
一般HTTP请求提交数据，需要编码成URL编码格式，然后做为url的一部分，或者作为参数传到Request对象中。  
此时需要使用parse.urlencode对需要提交的数据进行编码。

#### Get方式
GET请求一般用于向服务器获取数据。

```python
from urllib import request, parse

def baidu_search_by_keyword(kw):
    url = "http://www.baidu.com/"
    wd = {"wd": kw}
    wd = parse.urlencode(wd)

    full_url = url + "s?" + wd

    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
    }

    req = request.Request(full_url, headers=headers)
    res = request.urlopen(req)

    with open("02_result.html", "wb") as f:
        f.write(res.read())

if __name__ == "__main__":
    keyword = input("请输入要搜索的内容：")
    baidu_search_by_keyword(keyword)
```

##### 批量爬取贴吧页面数据
首先需要对百度贴吧的链接进行分析，找出其中的规律。

```python
from urllib import request, parse


def tieba_spider(url, beginpage, endpage):
    """
        作用：负责处理url，分配每个url去发送请求
        url：需要处理的第一个url
        beginPage: 爬虫执行的起始页面
        endPage: 爬虫执行的截止页面
    """

    for page in range(beginpage, endpage + 1):
        pn = (page - 1) * 50

        filename = "第" + str(page) + "页.html"
        # 组合为完整的 url，并且pn值每次增加50
        fullurl = url + "&pn=" + str(pn)

        # 调用loadPage()发送请求获取HTML页面
        html = load_page(fullurl, filename)
        # 将获取到的HTML页面写入本地磁盘文件
        write_file(html, filename)


def load_page(url, filename):
    """ 
        作用：根据url发送请求，获取服务器响应文件
        url：需要爬取的url地址
        filename: 文件名
    """

    print("正在下载" + filename)
    headers = {"User-Agent": "Mozilla/5.0 (compatible; MSIE 9.0; Windows NT 6.1; Trident/5.0;"}
    req = request.Request(url, headers=headers)
    res = request.urlopen(req)
    return res.read()


def write_file(html, filename):
    """
        作用：保存服务器响应文件到本地磁盘文件里
        html: 服务器响应文件
        filename: 本地磁盘文件名
    """
    print("正在存储" + filename)
    with open(filename, 'wb') as f:
        f.write(html)
    print("-" * 20)

if __name__ == "__main__":
    kw = input("请输入需要爬取的贴吧:")
    # 输入起始页和终止页，str转成int类型
    beginPage = int(input("请输入起始页："))
    endPage = int(input("请输入终止页："))
    url = "http://tieba.baidu.com/f?"
    key = parse.urlencode({"kw": kw})
    # 组合后的url示例：http://tieba.baidu.com/f?kw=python
    url = url + key
    tieba_spider(url, beginPage, endPage)
```

#### POST方式
Request请求对象的里有data参数，它就是用在POST里的，此时要传送的数据就是这个参数data，data是一个字典，里面要匹配键值对。  

```python
from urllib import request, parse

def youdao_translate(keyword):
    url = "http://fanyi.youdao.com/translate?smartresult=dict&smartresult=rule"
    headers = {
        "User-Agent": "Mozilla/5.0 (Windows NT 10.0; WOW64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/65.0.3325.181 Safari/537.36"
    }

    formdata = {
        "from": "AUTO",
        "to": "AUTO",
        "i": keyword,
        "smartresult": "dict",
        "client": "fanyideskweb",
        "doctype": "json",
        "version": "2.1",
        "keyfrom": "fanyi.web",
        "ue": "UTF-8",
        "action": "FY_BY_ENTER",
        "typoResult": "false"
    }

    data = parse.urlencode(formdata).encode('utf-8')
    req = request.Request(url, data=data, headers=headers)
    res = request.urlopen(req)

    print(res.read().decode('utf-8'))

if __name__ == "__main__":
    youdao_translate("黑暗")
```

#### 获取AJAX加载的内容
有些网页内容使用AJAX加载，AJAX一般返回的是JSON，直接对AJAX地址进行post或get，就返回JSON数据了。  
在写爬虫程序时，最需要关注的，是数据的来源。  

#### 处理HTTPS请求 SSL证书验证
在访问网页的时候则会报出SSLError：
urllib.error.URLError:<urlopen error [SSL: CERTIFICATE_VERIFY_FAILED] certificate verity failed (.ssl.c:777)

此时需要单独处理SSL证书，让程序忽略SSL证书验证错误，即可正常访问。解决办法：  

```python
import ssl

ssl._create_default_https_context = ssl._create_unverified_context
```

#### 关于CA
CA(Certificate Authority)是数字证书认证中心的简称，是指发放、管理、废除数字证书的受信任的第三方机构，如北京数字认证股份有限公司、上海市数字证书认证中心有限公司等。  
CA的作用是检查证书持有者身份的合法性，并签发证书，以防证书被伪造或篡改，以及对证书和密钥进行管理。  
现实生活中可以用身份证来证明身份， 那么在网络世界里，数字证书就是身份证。和现实生活不同的是，并不是每个上网的用户都有数字证书的，往往只有当一个人需要证明自己的身份的时候才需要用到数字证书。  
普通用户一般是不需要，因为网站并不关心是谁访问了网站，现在的网站只关心流量。但是反过来，网站就需要证明自己的身份了。  
比如说现在钓鱼网站很多的，比如你想访问的是www.baidu.com，但其实你访问的是www.daibu.com”，所以在提交自己的隐私信息之前需要验证一下网站的身份，要求网站出示数字证书。  
一般正常的网站都会主动出示自己的数字证书，来确保客户端和网站服务器之间的通信数据是加密安全的。  
