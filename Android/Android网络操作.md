# Android 网络操作

在移动互联网时代，基本所有的的Android应用都已联网为充分必要条件，所有Android应用的网络开发也成为必须技能。

## 基本概念

在网络开发过程中最经常接触到的概念就是TCP/IP协议，SOCKET套接字，HTTP协议。

### 1. TCP/IP

TCP/IP是一个协议族,它包含了很多协议，和它相对应的是OSI（Open System Interconnect）网络七层模型。

|OSI分层|功能|TCP/IP协议|
|:--------:|:--------:|:--------:|
|应用层(Application Layer)|文件传输，电子邮件，文件服务，虚拟终端|HTTP,SNMP,FTP,SMTP,Telnet,TFTP|
|表示层(Presentation Layer)|数据格式化，代码转换，数据加密|没有协议|
|会话层(Session Layer)|解除或建立与其他接点的联系|没有协议|
|传输层(Transport Layer)|提供端对端的接口|TCP,UDP|
|网络层(Network Layer)|为数据包选择路由(地址解析和路由)|IP,ICMP,RIP,OSPF,BGP,IGMP|
|数据链路层(Data Link Layer)|数据成帧，错误检测，流量控制|SLIP,CSLIP,PPP,ARP,RARP,MTU|
|物理层(Physical Layer)|二进制数据形式在物理媒体上传输数据|IOS2110,IEEE802，IEEE802.2|

![](../Image/七层网络模型.jpg)

除了OSI划分的七层模型，在更早的时候是TCP/IP的五层模型，其中TCP/IP五层模型中的应用层对应OSI七层模型的应用层+表示层+回话层，其余相同。

### 2. Socket

Socket，一个应用程序接口，使主机或一台计算机上的进程间可以通讯。（[摘自维基百科](https://zh.wikipedia.org/wiki/Berkeley%E5%A5%97%E6%8E%A5%E5%AD%97)）因此Socket是把TCP/IP层复杂的操作抽象为简单的接口供应用层调用来实现网络通讯。

![](../Image/socket结构图.jpg)

在Socket中封装的主要是使用TCP协议的流套接字和使用UDP协议的数据报套接字。

`TCP`(Transmission Control Protocol,传输控制协议)是面向连接的协议，也就是说。在收发数据前，必须和对方建立可靠的连接。TCP协议建立连接需要**三次握手**，在三次握手的过程中主要通过判断位码来建立连接，总共有六种位码：SYN(Synchronous 建立连接)，ACK(Acknowledgement 确认)，FSH(Push 传送)，FIN(Finish 结束)，RST(Reset 重置)，URG(Urgent 紧急)。Sequence Number(顺序码)，Acknowledge Number(确认号)。

1. 客户端发送SYN包，其中SYN=1，Sequence Number=x(x为随机生成)，进入SYN_SENT状态
2. 服务器收到SYN包，发送SYN+ACK包到客户端，其中SYN=1，ACK=1，Sequence Numer=y(y为服务器随机生成)，Acknowledge Numer=x+1(x为主动发SYN包中携带的顺序码)，进入SYN_RECV状态
3. 客户端收到SYN+ACK包，发送ACK包到服务器，其中ACK=1，Acknowledge Number=y+1(y为服务器发送SYN+ACK包中携带的顺序码)，至此发送完毕，到服务器接收到数据包，双方进入ESTABLISHED状态，建立连接完成。

当通讯双发完成数据传输后，就可以断开连接。由于TCP连接时全双工的，因此每个方向都必须单独进行关闭。TCP断开连接需要经历**四次握手**过程。

1. 客户端发送FIN+ACK包，其中FIN=1，ACK=1，Sequence Number=x，Acknowledge Number=y，客户端进入FIN_WAIT_1状态并关闭客户端到服务器的连接
2. 服务器收到FIN+ACK包，发送ACK包到客户端，其中ACK=1，Acknowledge Number=x+1(x为客户端中发送的顺序码)，客户端收到ACK包后进入FIN_WAIT_2状态
3. 服务器关闭与客户端的连接，发送FIN包到客户端，其中FIN=1，Sequence Number=y+1。服务器进入LAST_ACK状态。
4. 客户端收到FIN包，进入TIME_WAIT状态，并发ACK包到服务器，其中ACK=1，Acknowledge Number=y+2，至此双方完成四次握手断开连接。

![](../Image/tcp1.jpg)

![](../Image/20160615201932685.jpg)

`UDP`(User Datagram Protocol 用户数据报协议)，是一种面向非连接的协议，面向非连接指的是在正式通讯前不必与对方先建立连接，不管对方状态就直接发送。至于对方是否可以接到通信数据内容，UDP协议无法控制，因此说UDP协议是一种不可靠的协议。UDP协议只适合一次性传输少量数据，对可靠性要求不高的应用环境。

### 3. HTTP/HTTPS

#### 3.1 HTTP协议

`HTTP`(HyperText Transfer Protocol, 超文本传输协议)，HTTP是一个客户端（用户）和服务器（网站）请求和应答的标准。通过使用Web浏览器、网络爬虫或者其他的工具，客户端发起一个HTTP请求到服务器上指定的端口（默认端口为80）。我们称这个客户端为用户代理程序（user agent）。应答服务器上存储着一些资源，比如HTML文件和图像，我们称这个应答服务器为源服务器（origin server）。

#### 3.2 HTTPS协议

`HTTPS`(HyperText Transfer Protocol Over Secure Socket Layer)，以安全为目的的HTTP通道。HTTPS的安全基础是`SST/TSL`，`SSL`(Secure Sockets Layer, 安全套接层)是网景公司开发为了解决`HTTP`协议明文传输会造成内容被嗅探和篡改问题的加密协议，后来由于`SSL`的应用广泛，IETF就把`SSL`协议标准化，标准化之后改名为`TLS`(Transport Layer Security, 传输层安全协议)，因此两者可以视为同一个东西的不同阶段。

#### 3.3 URL

`URL`(Uniform Resouce Locator)统一资源定位器，它是指向互联网资源的指针。资源可以是简单的文件后者目录，也可以是更为复杂的对象的引用，例如对数据库或者搜索引擎的查询。其格式如下：

```url
schema://host[:port#]/path/../[?query-string][#anchor]
```

* `schema`：指定底层使用的协议，例如http，https，ftp
* `host`：http服务器的IP地址或者域名
* `port#`：http服务器的默认端口是80，这种情况下可以省略。如果使用了逼的端口必须指明，例如`http://127.0.0.1:8085`
* `path`：访问资源的路径
* `query-string`：发送给http服务器的数据
* `anchor`： 锚

#### 3.4 Http请求报文格式

HTTP的请求报文分为三部分：请求行、请求头、请求体

![](../Image/1724103-c43900117e983241.png)

##### 请求行

请求行（Request Line）分为三部分：请求方法（Method）、请求地址（Request-URI）、协议及版本（HTTP-Version）。`HTTP/1.1`定义了8种请求方法：`GET`、`POST`、`PUT`、`DELETE`、`PATCH`、`HEAD`、`OPTIONS`、`TRACE`。最常用的是`GET`和`POST`。

请求资源即URI（Uniform Resouce Identifiers, 统一资源标识符）地址。

Http协议有如下版本：HTTP/0.9、HTTP/1.0、HTTP/1.1、HTTP/2.0，其中现在被广泛使用的是HTTP/1.1协议版本。

#### 3.5 Http响应报文格式

HTTP响应的格式出了状态行与请求的请求行不一样，其他就格式而言是一样的。但同过请求头和响应头的不用还是可以判断出请求报文还是响应报文。

![](../Image/1724103-e8ebcab6c80b9044.png)

响应状态行（Response Line）分为三部分：协议版本（HTTP-Version）、响应码（Status Code）、响应信息（Reason Phrase）。

响应码也叫状态码，有三位数字组成，第一个数字定义了响应的级别，有五种可能：

* 1XX：指示消息，表示请求已接收，继续处理
* 2XX：成功，表示请求已被成功接收、理解、处理
* 3XX：重定向，表示要完成请求必须进行进一步的操作
* 4XX：客户端错误，请求有语法错误或请求无法实现
* 5XX：服务端错误，服务器未能实现合法请求

[状态码详解](http://tool.oschina.net/commons?type=5)

响应信息即是对状态码的描述

#### 3.6 Header

Header可以分为请求头和响应头，它们的格式都是键值对存在。

##### 3.6.1 请求和响应通过的Header

|名称|作用|
|:--------:|:----------:|
|Content-Type|请求体或响应体的类型，如：text/plain、application/json|
|Accept|说明接收的类型，可以有多个值，用`,`隔开|
|Content-Length|请求体或响应体的长度，单位字节|
|Content-Encoding|请求体或响应体的编码格式，如gzip、deflate|
|Accept-Encoding|告知对方我放接受的Content-Encoding|
|ETag|给当前资源的标示，和`Last-Modified`、`If-None-Match`、`If-Modified-Since`配合，用于缓存控制|
|Cache-Control|取值一般为`no-cache`或`max-age=XX`，XX为整数，标示改资源存在有效期（秒）|

##### 3.6.2 请求常见Header

|名称|作用|
|:--------:|:----------:|
|Authorization|用于设置身份认证信息|
|User-Agent|用户标示，如：OS和浏览器的类型和版本|
|If-Modified-Since|值为上一次服务器返回的`Last-Modified`值，用于确认某个资源是否被改过，没有更改过（304）就从缓存中读取|
|If-None-Match|值为服务器上一次返回的`ETag`值，一般会和`If-Modified-Since`一起出现|
|Cookie|已有的Cookie|
|Referer|表示请求引用自哪个地址|
|Host|请求的主机和端口号|

##### 3.6.3

|名称|作用|
|:--------:|:----------:|
|Date|服务器的日期|
|Last-Modified|该资源最后被修改的时间|
|Transter-Encoding|取值一般为`chunked`，出现在`Content-Length`不能确定的情况下，表示服务器不知道响应版体的数据大小，一般还会出现`Content-Encoding`响应头|
|Set-Cookie|设置Cookie|
|Location|重定向到另一个URL|
|Server|后台服务器|

#### 3.7 请求体

##### 3.7.1 Get方法

使用`GET`方法没有请求体，请求参数和对应的值附加在URL后面，利用一个`?`代表URL结尾和请求参数的开始，多个参数之间用`&`隔开，参数与对应值之间用`=`隔开。

```url
http://www.google.cn/search?hl=zh-CN&source=hp
```

###### 3.7.2 Post方法

`POST`方法将提交的数据以键值对的形式放在请求体（Body）中，此时`Content-Type`为`application/x-www-form-urlencoded`。

![](../Image/1724103-18847d9a34c50bdd.png)

## 系统API

### 1. Java Socket

### 2. HttpURLConnection

### 3. HttpClient

## 第三方网络库

### 1. OkHttp

### 2. Retrofit

# 参考

* [简单理解Socket](http://www.cnblogs.com/dolphinX/p/3460545.html)
* [OSI七层与TCP/IP五层网络架构详解](http://network.51cto.com/art/201310/413853.htm)
* [Android网络操作和优化相关](http://blog.csdn.net/sdkfjksf/article/details/51645315)
* [你应该知道的HTTP基础知识](http://www.jianshu.com/p/e544b7a76dac)
* [Android网络请求心路历程](http://www.jianshu.com/p/3141d4e46240)
* [TCP网络关闭的状态变换时序图](http://coolshell.cn/articles/1484.html)
