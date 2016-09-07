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

## OkHttp的使用

### 1. 创建[Request](http://square.github.io/okhttp/3.x/okhttp/okhttp3/Request.html)

HTTP客户端所要做的就是执行请求获取响应，每一个HTTP的`Request`都必然包含一个请求方法（如GET或POST）、一个URL请求地址和一系列的请求头。有些时候还会包含特定类型的数据流的请求体。在OkHttp中一个`Request`对象需要使用工具类`Request.Builder`来构建。

一个简单的GET请求

```java
Request request = new Request.Builder()
    .url("http://gank.io/api/day/2016/06/22")//设置请求URL地址，默认Method为GET
    .build();
```

HTTP头是请求和响应的重要部分，在创建请求时可以设置一些HTTP头，在获取响应后，也可以对其中包含的HTTP头进行解析。在设置HTTP头是，可以使用`header(name, value)`方法设置HTTP头的唯一值，对于同一个HTTP头，多次调用会覆盖之前的设置，使用`addHeader(name, value)`方法为HTTP头添加新值。

```java
Request request = new Request.Builder()
    .url(url)
    .header("User-Agent","OkHttpDemo")
    .addHeader("Accept", "application/json")
    .build();
```

OkHttp如果要发送POST请求，只需要在创建`Request`对象时，使用`post(requestBody)`方法来指定提交的内容即可。

String的提交，可以直接使用`RequestBody`对象的静态方法`create(mediaType, body)`创建一个`RequestBody`对象实例。

```java
MediaType mediaType = MediaType.parse("text/plain; charset=utf-8");
Request request = new Request.Builder()
    .url(url)
    .post(RequestBody.create(mediaType, "OkHttpDeme"))
    .build();
```

数据流的提交，需要覆写`RequestBody`对象的`contentType()`方法和`writeTo(sink)`方法，在`contentType()`方法返回指定的媒体类型，在`writeTo(sink)`方法中向`BufferedSink`对象中写入数据即可。

```java
final MediaType mediaType = MediaType.parse("text/plain; charset=utf-8");
RequestBody requestBody = new RequestBody() {
    @Override
    public MediaType contentType() {
        return mediaType;
    }

    @Override
    public void writeTo(BufferedSink sink) throws IOException {
        OutputStream out = sink.outputStream();
        String body = "This is a test message.";
        out.write(body.getBytes());
        out.close();
    }
};
Request request = new Request.Builder()
    .url(url)
    .post(requestBody)
    .build();
```

表单数据的提交，可以使用`FormBody.Builder`工具类直接创建表单数据。

```java
RequestBody requestBody = new FormBody.Builder()
    .add("user","admin")
    .add("Passwd", "123456")
    .build();

Request request = new Request.Builder()
    .url(url)
    .post(requestBody)
    .build();
```

还有更多的提交文件、MultiPart的提交等。可以参考[OkHttp的介绍](https://github.com/square/okhttp/wiki/Recipes)。

### 2. 创建[OkHttpClient](http://square.github.io/okhttp/3.x/okhttp/okhttp3/OkHttpClient.html)和[Call](http://square.github.io/okhttp/3.x/okhttp/okhttp3/Call.html)执行请求

首先需要创建一个`OkHttpClient`对象，该对象是使用OkHttp的入口，同时也是`Call`对象的工厂类。OkHttp使用`Call`对象来执行一个请求并获取响应，`Call`是一个接口。使用`OkHttpClient`对象的`newCall(request)`方法并传入一个`Request`来创建一个`Call`的实现。当获得一个`Call`实例后，既可以调用`execute()`（同步）方法或`enqueue(Callback)`（异步）方法来执行这个`Request`请求。

使用`execute`方法同步获取响应结果

```java
OkHttpClient client = new OkHttpClient();
Response response = client.newCall(request).execute();
```

使用`enqueue`方法异步获取响应结果，其将请求加入到执行队列中，当获得请求结果后会回调设置的`Callback`接口的`onFailure`或`onResponse`方法。

```java
client.newCall(request).enqueue(new Callback() {
    @Override
    public void onFailure(Call call, IOException e) {
        Log.e("TAG", e.getMessage());
    }

    @Override
    public void onResponse(Call call, Response response) throws IOException {
        Log.i("TAG", response.code()+";"+response.message());
    }
});
```

### 3. 解析[Response](http://square.github.io/okhttp/3.x/okhttp/okhttp3/Response.html)

获取响应对象`Response`后，既可以使用其方法`isSuccessful()`方法判断响应是否成功，也可以使用`code()`和`message()`方法获取响应状态码及其描述。如果响应成功，则可以使用`headers()`方法获得包含响应头信息的[`Headers`](http://square.github.io/okhttp/3.x/okhttp/okhttp3/Headers.html)对象，`body()`方法获得包含响应体信息的[`ResponseBody`](http://square.github.io/okhttp/3.x/okhttp/okhttp3/RequestBody.html)对象。

```java
if(!response.isSuccessful())
    throw new IOException(response.code()+":"+response.message());
Headers headers = response.headers();
for (int i = 0, len = headers.size(); i < len; i++){
    Log.i("TAG", headers.name(i) + ":" + headers.value(i));
}
ResponseBody responseBody = response.body();
MediaType mediaType = responseBody.contentType();
Log.i("TAG", mediaType.type()+";"+mediaType.charset().displayName());
Log.i("TAG", responseBody.string());
```

### 4. 使用响应缓存

OkHttp可以对HTTP的响应在磁盘上进行缓存。在进行HTTP请求的时候，如果请求的响应已经被缓存而且没有过期，OkHttp会直接使用缓存中的响应内容。如果要设置响应缓存，首先需要创建缓存`    Cache`对象，其构造函数传入两个参数，分别是缓存文件所在路径和缓存空间大小。创建好缓存对象后，使用工具类`OkHttpClient.Builder`构建`OkHttpClient`对象。这里需要注意的是，如果一个缓存目录有多个缓存访问时错误的，否则两个缓存互相干扰，破坏响应缓存，这样会导致应用崩溃。

```java
Cache cache = new Cache(MainApp.getContext().getCacheDir(),
    5 * 1024 * 1024/* 5MiB */);
mClient = new OkHttpClient.Builder()
    .cache(cache)
    .build();
```

### 5. 配置OkHttpClient

使用工具类`OkHttpClient.Builder`可以很方便的设置超时时间、代理、缓存、认证等信息。并且OkHttp建议，全局应保证只有一个`OkHttpClient`实例，从而达到共享响应缓存、线程池和连接池等资源。

配置超时时间

```java
mClient = new OkHttpClient.Builder()
    .connectTimeout(10 , TimeUnit.SECONDS)
    .readTimeout(30 , TimeUnit.SECONDS)
    .writeTimeout(10, TimeUnit.SECONDS)
    .build();
```

## OkHttp的拦截机制

拦截器是OkHttp提供的对HTTP请求和响应进行统一处理的强大机制拦截器在执行的时候，可以先对请求的`Request`进行修改，再得到响应的`Response`后，再对其进行修改后返回。拦截器对象`Interceptor`接口包含一个`intercept`方法，其参数是`Chain`对象。`Chain`对象表示的是当前的拦截器链条，通过`Chain`的`request()`方法可以获取当前的`Request`对象。在使用完`Request`对象后，通过`Chain`对象的`proceed()`方法来继续拦截器链条的执行。

OkHttp中的拦截器分成应用拦截器和网络拦截器。应用拦截器对于每一个HTTP请求都只会调用一次，可以通过不调用`Chain.proceed`方法来终止请求，也可以通过多次调用`Chain.proceed`方法来进行重试。网络拦截器对于调用执行过程中的重定向和重试所产生的响应也会调用，而如果响应来自缓存，则不会调用。

```java
class LogInterceptor implements Interceptor{

    @Override
    public Response intercept(Chain chain) throws IOException {
        Response response = null;

        long t1 = System.nanoTime();
        Request request = chain.request();
        System.out.println(String.format("Send Request: [%s] %s%n%s",
            request.url(), chain.connection(), request.headers()));

        response = chain.proceed(request);

        long t2 = System.nanoTime();
        System.out.println(String.format("Receive Response: [%s] %.1fms%n%s",
            response.request().url(), (t2-t1)/1e6d, response.headers()));

        return response;
    }
}

mClient.newBuilder()
    .addInterceptor(new LogInterceptor())
    .addNetworkInterceptor(new LogInterceptor())
    .build();
```

# 参考

* [OkHttp Wiki](https://github.com/square/okhttp/wiki)
* [OkHttp 官网](http://square.github.io/okhttp/#)
* [OkHttp：Java平台上的新一代HTTP客户端](https://www.ibm.com/developerworks/cn/java/j-lo-okhttp/)
