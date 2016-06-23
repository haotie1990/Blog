# Android网络操作之OkHttp

## OkHttp介绍

OkHttp是Square公司开发的用于HTTP请求库，它具有以下特性：

* 支持HTTP2版本协议，使得请求同一主机地址的请求共享一个连接
* 使用连接池，可以降低网络延迟
* 使用gzip压缩节省流量
* 具有响应缓存，避免重复请求

除了以上特点外，当网络出现异常时候，OkHttp还可以尝试去修复问题。如果你的服务器有多个IP地址，当其中一个IP地址连接出现异常，OkHttp会尝试连接其他IP地址。

使用OkHttp有三种方式：

1. 下载jar包，但使用OkHttp同时也需要使用okio，这是一个拥有更高速的I/O和可调整的缓存的I/O库。[地址](http://square.github.io/okhttp/#)
2. 配置Maven

```xml
<dependency>
  <groupId>com.squareup.okhttp3</groupId>
  <artifactId>okhttp</artifactId>
  <version>3.3.1</version>
</dependency>
```

3. 配置Gradle

```groxy
compile 'com.squareup.okhttp3:okhttp:3.3.1'
```

## OkHttp中的概念

## OkHttp的使用

## OkHttp的拦截机制

## OkHttp的HTTPS

## OkHttp源码解析

# 参考

* [OkHttp Wiki](https://github.com/square/okhttp/wiki)
* [OkHttp 官网](http://square.github.io/okhttp/#)
