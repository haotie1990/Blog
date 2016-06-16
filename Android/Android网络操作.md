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

TCP(Transmission Control Protocol,传输控制协议)是面向连接的协议，也就是说。在收发数据前，必须和对方建立可靠的连接。TCP协议建立连接需要**三次握手**，在三次握手的过程中主要通过判断位码来建立连接，总共有六种位码：SYN(Synchronous 建立连接)，ACK(Acknowledgement 确认)，FSH(Push 传送)，FIN(Finish 结束)，RST(Reset 重置)，URG(Urgent 紧急)。Sequence Number(顺序码)，Acknowledge Number(确认号)。

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

UDP(User Datagram Protocol 用户数据报协议)，是一种面向非连接的协议，面向非连接指的是在正式通讯前不必与对方先建立连接，不管对方状态就直接发送。至于对方是否可以接到通信数据内容，UDP协议无法控制，因此说UDP协议是一种不可靠的协议。UDP协议只适合一次性传输少量数据，对可靠性要求不高的应用环境。

### 3. HTTP/HTTPS

#### 3.1 HTTP协议

HTTP(HyperText Transfer Protocol, 超文本传输协议)，HTTP是一个客户端（用户）和服务器（网站）请求和应答的标准。通过使用Web浏览器、网络爬虫或者其他的工具，客户端发起一个HTTP请求到服务器上指定的端口（默认端口为80）。我们称这个客户端为用户代理程序（user agent）。应答服务器上存储着一些资源，比如HTML文件和图像，我们称这个应答服务器为源服务器（origin server）。

#### 3.2 HTTPS协议

HTTPS(HyperText Transfer Protocol Over Secure Socket Layer)，以安全为目的的HTTP通道。HTTPS的安全基础是SST/TSL，SSL(Secure Sockets Layer, 安全套接层)是网景公司开发为了解决HTTP协议明文传输会造成内容被嗅探和篡改问题的加密协议，后来由于SSL的应用广泛，IETF就把SSL协议标准化，标准化之后改名为TLS(Transport Layer Security, 传输层安全协议)，因此两者可以视为同一个东西的不同阶段。

#### 3.3 URL

URL(Uniform Resouce Locator)统一资源定位器，它是指向互联网资源的指针。资源可以是简单的文件后者目录，也可以是更为复杂的对象的引用，例如对数据库或者搜索引擎的查询。其格式如下：

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

#### 3.5 Http响应报文格式

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
