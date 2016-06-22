# Android网络操作之Volley

## 1. Volley简介

Volley是谷歌开发的一个HTTP请求库，它基于Android平台，使得开发Android应用的网络功能更加简单、快速和高效。它具有以下特点：

* 网络请求的自动调度
* 并发的网络连接
* 基于内存和本地的响应缓存
* 支持网络请求排序
* 支持取消网络请求
* 支持定制
* 支持异步网络请求

### 配置导入Volley库

如果仍然使用Eclipse开发，那么首先需要下载Volley的源码，然后生成.jar包

```
git clone https://android.googlesource.com/platform/frameworks/volley

```

如果是Android Studio那么需要在app下的`build.gradle`配置：

```
compile 'com.android.volley:volley:1.0.0'
```

## 2. StringRequest、JsonRequest和ImageRequest

Volley支持字符串、Json对象和图片三种格式的请求。

**StringRequest**请求支持返回字符串，其构造函数有四个参数，第一个参数`method`指定请求的方式GET、POST或者其他，第二参数`url`指定请求的地址，第三个参数`listener`指定一个监听实现来接收请求结果，第四个参数`errorListener`指定如果请求失败的监听接口。

```java
/**
 * Creates a new request with the given method.
 *
 * @param method the request {@link Method} to use
 * @param url URL to fetch the string at
 * @param listener Listener to receive the String response
 * @param errorListener Error listener, or null to ignore errors
 */
public StringRequest(int method, String url, Listener<String> listener,
        ErrorListener errorListener) {
    //---
}
```

**JsonRequest**请求支持返回json格式的数据，它有两个子类JsonObjectRequest和JsonArrayRequest。两个并未有什么特殊的区别，这里主要说明一下JsonObjectRequest。JsonObjectRequest的构造函数有五个参数，第一个参数`method`指定了请求方法，第二个参数`url`指定了请求地址，第三个参数`jsonRequest`指定了请求携带的数据（针对POST请求，如果为null，则默认请求方法为GET），第四个参数`listener`成功监听器，第五个参数失败监听器。

```java
/**
 * Creates a new request.
 * @param method the HTTP method to use
 * @param url URL to fetch the JSON from
 * @param jsonRequest A {@link JSONObject} to post with the request. Null is allowed and
 *   indicates no parameters will be posted along with request.
 * @param listener Listener to receive the JSON response
 * @param errorListener Error listener, or null to ignore errors.
 */
public JsonObjectRequest(int method, String url, JSONObject jsonRequest,
        Listener<JSONObject> listener, ErrorListener errorListener) {
    //---
}
```

**ImageRequest**请求用于获取图片，其构造函数有六个参数：第一个参数`url`指定图片的URL地址。第二个参数`listener`指定请求成功后的监听回调，在这里我们可以把返回的Bitmap设置到ImageView等控件中。第三个参数`maxWidth`指定允许图片的最大宽度，第四个参数`maxHeight`指定允许图片的最大高度，如果获取的网络图片的宽度或高度大于指定值，则会对图片进行压缩。第五个参数`scaleType`指定图片的展开方式用于计算图片大小。第六个参数`decodeConfig`指定图片的解码属性，使用Bitmap.Config下的常量，其中ARGB_8888代表最好的图片显示，其每个像素占据4个字节，RGB_565代表每个像素2个字节，一般使用RGB_565即可。第七个参数`errorListener`指定请求失败的监听回调。

```java
/**
 * Creates a new image request, decoding to a maximum specified width and
 * height. If both width and height are zero, the image will be decoded to
 * its natural size. If one of the two is nonzero, that dimension will be
 * clamped and the other one will be set to preserve the image's aspect
 * ratio. If both width and height are nonzero, the image will be decoded to
 * be fit in the rectangle of dimensions width x height while keeping its
 * aspect ratio.
 *
 * @param url URL of the image
 * @param listener Listener to receive the decoded bitmap
 * @param maxWidth Maximum width to decode this bitmap to, or zero for none
 * @param maxHeight Maximum height to decode this bitmap to, or zero for
 *            none
 * @param scaleType The ImageViews ScaleType used to calculate the needed image size.
 * @param decodeConfig Format to decode the bitmap to
 * @param errorListener Error listener, or null to ignore errors
 */
public ImageRequest(String url, Response.Listener<Bitmap> listener, int maxWidth, int maxHeight,
        ScaleType scaleType, Config decodeConfig, Response.ErrorListener errorListener) {
    //---
}

```

## 3. RequestQueue

在Volley中，所有构造出来的`Request`请求只要添加到`RequestQueue`中，Volley就会自动的去执行它们，然后将结果通过Request的回调函数返回来。Volley通过静态方法`newRequestQueue`来获取一个`RequestQueue`，通过add方法将一个`Request`请求放入请求队列中。

```java
// 创建字符串请求
StringRequest dataRequest = new StringRequest(Request.Method.GET, url,
    new Response.Listener<String>() {
        @Override
        public void onResponse(String response) {

        }
    },
    new Response.ErrorListener() {
        @Override
        public void onErrorResponse(VolleyError error) {

        }
    });
dataRequest.setTag(url);
// 将创建的Request放入请求队列，Volley会自动执行此请求
RequestQueue requestQueue = Volley.newRequestQueue(MainApplication.getContext());
requestQueue.add(dataRequest);
```
`RequestQueue`同样支持取消`Request`请求，通过`cancelAll`方法结束相应的`Request`请求。`cancelAll`有两个重载方法参数分别是`Request`请求的标签对象`Tag`或者是一个`Request`请求的过滤器对象`RequestFilter`。

```java
/**
* Cancels all requests in this queue with the given tag. Tag must be non-null
* and equality is by identity.
*/
public void cancelAll(final Object tag) {
//---
}
```
```java
requestQueue.cancelAll(url);
```

***

```java
/**
 * Cancels all requests in this queue for which the given filter applies.
 * @param filter The filtering function to use
 */
public void cancelAll(RequestFilter filter) {
    //---
}
```
```java
requestQueue.cancelAll(new RequestQueue.RequestFilter() {
    @Override
    public boolean apply(Request<?> request) {
        return request.getUrl().equals(url);
    }
});
```

不仅可以使用静态方法`newRequestQueue`获取一个`RequestQueue`，我们同样可以自己构建一个`RequestQueue`，`RequestQueue`的构造函数有四个参数，第一个参数`cache`代表一个实现`Cache`缓存接口的实现；第二个参数`network`代表一个实现`Network`网络接口的实现，此接口用于执行请求并获取响应结果；第三个参数`threadPoolSize`代表网络请求处理线程的数量，默认为4；第四个参数`delivery`代表一个响应分发接口`ResponseDelivery`的实现，用来分发请求获得的响应或失败信息。

```java
/**
 * Creates the worker pool. Processing will not begin until {@link #start()} is called.
 *
 * @param cache A Cache to use for persisting responses to disk
 * @param network A Network interface for performing HTTP requests
 * @param threadPoolSize Number of network dispatcher threads to create
 * @param delivery A ResponseDelivery interface for posting responses and errors
 */
public RequestQueue(Cache cache, Network network, int threadPoolSize,
        ResponseDelivery delivery) {
    //---
}
```
现在只修改缓存实现和网络实现，缓存使用Volley中的`DiskBasedCache`，网络请求处理使用Volley中的`HurlStack`。

```java
RequestQueue mRequestQueue;

// 初始化缓存
Cache cache = new DiskBasedCache(getCacheDir(), 1024 * 1024); // 1MB cap

// 使用HttpURLConnection作为网络处理请求的Http实现
Network network = new BasicNetwork(new HurlStack());

// 使用自定义的Cache和Network构造一个RequestQueue
mRequestQueue = new RequestQueue(cache, network);

// 使设置的缓存和网络请求处理生效，此函数必须调用
mRequestQueue.start();
```

## 4. ImageLoader和自定义Cache

`ImageLoader`是一个更加方便的图片加载工具类，它可以设置自定义的`ImageCache`，其可以是内存缓存也可以是Disk缓存；还可以定义图片请求过程中显示的图片和请求失败后的图片；并且相同的请求（相同地址，相同大小）只会发送一个，可以避免重复请求。

```java
// 新建一个ImageLoader,传入requestQueue和图片缓存类
ImageLoader imageLoader = new ImageLoader(requestQueue,new BitmapCache());
// 参数分别为要显示的图片控件，默认显示的图片（用于图片未下载完时显示），
// 下载图片失败时显示的图片
ImageListener listener = ImageLoader.getImageListener(imageView,default_image,  failed_image);
imageLoader.get(url, listener, maxWidth, maxHeight);//开始请求网络图片
```

前面已经讲过，`ImageLoader`不仅支持Disk缓存还支持内存缓存。

```java
// 基于LruCache内存缓存的实现
public class LruBitmapCache extends LruCache<String, Bitmap>
    implements ImageLoader.ImageCache{

    /**
     * @param maxSize for caches that do not override {@link #sizeOf}, this is
     *                the maximum number of entries in the cache. For all other caches,
     *                this is the maximum sum of the sizes of the entries in this cache.
     */
    public LruBitmapCache(int maxSize) {
        super(maxSize);
    }

    public LruBitmapCache(Context context){
        this(getCacheSize(context));
    }

    // Returns a cache size equal to approximately three screens worth of images
    private static int getCacheSize(Context context) {
        final DisplayMetrics dm = context.getResources().getDisplayMetrics();
        final int screenWidth = dm.widthPixels;
        final int screenHeight = dm.heightPixels;
        final int screenBytes = screenWidth * screenHeight * 4;
        return screenBytes * 3;
    }

    @Override
    protected int sizeOf(String key, Bitmap value) {
        return value.getRowBytes() * value.getHeight();
    }

    @Override
    public Bitmap getBitmap(String url) {
        return get(url);
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        put(url, bitmap);
    }
}
```

```java
// 基于DiskLruCache的Disk缓存实现
public class DiskLruImageCache implements ImageLoader.ImageCache{

    private static int VERSION = 1;

    private static int VALUE_COUNT = 1;

    private Context mContext;

    private DiskLruCache mDiskLruCache;

    public DiskLruImageCache(Context context) {
        mContext = context;
        try {
            mDiskLruCache = DiskLruCache.open(getDiskCacheDir("image"), VERSION, VALUE_COUNT,
                20 * 1024 * 1024);
        } catch (IOException e) {
            e.printStackTrace();
        }
    }

    private File getDiskCacheDir(String name){
        String cachePath;
        if(Environment.MEDIA_MOUNTED.equals(Environment.getExternalStorageState())
            || !Environment.isExternalStorageRemovable()){
            cachePath = mContext.getExternalCacheDir().getPath();
        }else{
            cachePath = mContext.getCacheDir().getPath();
        }
        return new File(cachePath + File.separator + name);
    }

    private String bytes2HexString(byte[] bytes){
        StringBuilder sb = new StringBuilder();
        for (int i = 0; i < bytes.length; i++){
            String hex = Integer.toHexString(0xFF & bytes[i]);
            if(hex.length() == 1){
                sb.append("0");
            }
            sb.append(hex);
        }
        return sb.toString();
    }

    private String MD5(String url){
        String md5Key;
        try {
            final MessageDigest messageDigest = MessageDigest.getInstance("MD5");
            messageDigest.update(url.getBytes());
            md5Key = bytes2HexString(messageDigest.digest());
        } catch (NoSuchAlgorithmException e) {
            md5Key = String.valueOf(url.hashCode());
        }
        return md5Key;
    }

    @Override
    public Bitmap getBitmap(String url) {
        String md5Key = MD5(url);
        try {
            DiskLruCache.Snapshot snapshot = mDiskLruCache.get(md5Key);
            InputStream is = snapshot.getInputStream(0);
            Bitmap bitmap = BitmapFactory.decodeStream(is);
            is.close();
            return bitmap;
        } catch (IOException e) {
            e.printStackTrace();
        }
        return null;
    }

    @Override
    public void putBitmap(String url, Bitmap bitmap) {
        String md5Key = MD5(url);
        try {
            DiskLruCache.Editor editor = mDiskLruCache.edit(md5Key);
            bitmap.compress(Bitmap.CompressFormat.PNG, 100, editor.newOutputStream(0));
        } catch (IOException e) {
            e.printStackTrace();
        }
    }
}
```
## 5. 自定义数据处理

所有的网络请求都是继承自`Request<T>`类，那么我们可以通过继承此类来实现一个自定义的请求类，下面是自定义的XML请求类。

```java
public class XmlRequest extends Request<XmlPullParser>{

    private Response.Listener<XmlPullParser> mListener;

    public XmlRequest(int method, String url, Response.Listener<XmlPullParser> listener, Response.ErrorListener errorListener) {
        super(method, url, errorListener);
        mListener = listener;
    }

    @Override
    protected Response<XmlPullParser> parseNetworkResponse(NetworkResponse response) {
        try {
            String xmlString = new String(response.data, HttpHeaderParser.parseCharset(response.headers));
            XmlPullParserFactory factory = XmlPullParserFactory.newInstance();
            XmlPullParser xmlPullParser = factory.newPullParser();
            xmlPullParser.setInput(new StringReader(xmlString));
            return Response.success(xmlPullParser, HttpHeaderParser.parseCacheHeaders(response));
        } catch (UnsupportedEncodingException e) {
            return Response.error(new ParseError(e));
        } catch (XmlPullParserException e) {
            return Response.error(new ParseError(e));
        }
    }

    @Override
    protected void deliverResponse(XmlPullParser response) {
        mListener.onResponse(response);
    }
}
```
# Demo

[VolleyDemo地址](https://github.com/haotie1990/VolleyDeme)

# 参考

* [Volley框架的使用](http://www.cnblogs.com/cpacm/p/4193011.html)
* [Volley 源码解析](http://a.codekk.com/detail/Android/grumoon/Volley%20%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90)
