# Android中的Intent需要知道的那些事

## Intent介绍

Intent是一个消息传递对象，可以使用它在应用组件之间传递信息。其基本使用有如下三个方面：
* 启动Activity，通过将Intent传递给startActivity()或者startActivityForReult()可以启动新的Activity实例。其中Intent描述了要启动的Activity，并携带任何必要的数据。
* 启动Service，通过将Intent传递给startService()或bindService()，可以启动一个Service实例，其中Intent描述了要启动的Service，并携带任何必要的数据。
* 发送Broadcast，通过将Intent传递给sendBroadcast()、sendOrderBroadcast()或sendStickyBroadcast()可以发送广播。

## Intent类型

Intent分两种类型：
* 显示Intent：按名称（完全限定类名）指定要启动的组件。通常会在自己的应用中使用显示Intent来启动组件，这是因为我们知道要启动的Activity或Service类名。
* 隐式Intent：不会指定特定的组件，而是声明要执行的操作（action），从而允许其他应用中的组件来处理它。

创建显示Intent启动Activity或Service时，系统会立即启动Intent中指定的应用组件。

创建隐式Intent时，系统会将Intent的内容与设备上其他应用的AndroidManifest中声明的IntentFilter比较，从而找到要启动的相应的组件。

> Warning：为了确保应用的安全性，启动Service时，请始终使用显示Intent，并且不要为Service声明IntentFilter。并且从Android5.0开始，如果使用隐式的intent调用bindService()将会抛出异常。

## 构建Intent

Intent对象携带了Android系统用来确定要启动哪个组件的信息（例如，准确的组件名称），以及收件人组件为了正确执行操作而使用的信息（例如，要采取的操作以及要处理的数据）

一个Intent对象通常包含如下主要信息：

**组件名称（Component Name）**

要启动组件的名称，这是一个可选项，但是也是显示Intent重要信息，因为如果没有组件名称，那么此Intent即是隐式Intent。Intent中的组件名称由ComponentName对象指定，可以使用目标组件的完全限定类名初始化此对象，其中包含应用的软件包名称。例如，`com.example.MainActivity`。您可以使用setComponent()，setClass()，setClassName()或Intent构造函数设置组件名称。

**操作（Action）**

指要执行的通用操作的字符串。对于广播Intent，是指已经发生正在报告的操作。操作很大程度上决定了其余Intent的构成，特别是数据和Extra部分。可以使用setAction()和Intent的构造函数指定操作。

**数据（Data）**

数据包括一个指向待操作数据的URI对象和该数据的MIME类型。要是仅设置数据URI，请调用setData()方法，要是仅设置MIME类型，请调用setType()，要是同时设置两者请调用setDataAndType()。

> Warning：若要同时设置URI和MIME类型，不可以分别调用setData()和setType()，因为它们会相互抵消彼此的值。

**类别（Category）**

一个包含应处理Intent组件类型的附加信息的字符串。可以将任意数量的Category描述放入到一个Intent中。一些常见的Category例如：
* CATEGORY_BROWSABLE：目标Activity允许本身通过Web浏览器启动
* CATEGORY_LAUNCHER：该Activity是task的初始Activity

可以使用addCategory()指定Category。

**Extra**

携带完整的请求操作所需要的附加信息的键值对。可以使用各种putExtra()方法添加附加数据，每个方法都接收两个参数：键名和值。

**Flags**

在Intent中定义的、充当Intent元数据的标志。标志可以指示系统如果启动Activity。和Activity的启动模式有关。可以通过setFlags()方法设置标志。

## IntentFilter的匹配

前面已经知道Intent分为显示和隐式两种。在AndroidManifest文件中定义的intent-filter即是用来与隐式Intent做匹配，如果可以匹配成功则启动相应的组件（Activity或Service）。

一个intent-filter中的action、category、data有多个，所有的action、category和data分别构成不同的类别，同一类别的信息共同约束当前类编的匹配过程。只有一个Intent同时匹配action类别、category类别和data类编才算完全匹配，只有完全匹配才能启动目标组件。

### 1. action匹配规则

Action的匹配规则是Intent中的action必须能够和过滤规则中的action匹配，这里的匹配是指字符串完全一样。一个过滤规则中可以有多个action，那么只要Intent总的action能够和过滤规则中的任何一个action相同既可以匹配成功。

> Warning：如果Intent中没有action，那么匹配失败。

### 2. Category匹配规则

category是一个字符串，系统定义了一些category，同时我们也可以在应用定义自己的category。category的匹配规则和action不同，它要求如果Intent中含有category，那么所有的category必须和过滤规则中的其中一个category相同。 也就是说，Intent如果包含category，不管有几个category，对于每个category来说，它必须是过滤规则中已经定义的category。当然Intent中可以没有category，如果没有category安装上面的规则，则是匹配成功。

> NOTE: 这个主意与action的匹配规则的不同，action是要求intent中必须有**一个**action且必须能够和过滤规则中的某个action相同。而category是要求intent中可以没有category，如果存在category，不管存在几个，每一个都要能够和过滤规则中的任何一个category相同。

### 3. Data匹配规则

data的匹配规则和action类似，如果过滤规则中定义了data，那么intent中必须要包含可匹配的data。

data由两部分组成，mimeType和URI。mimeType值媒体类型，比如image/jpeg、audio/mpeg4-generic等，可以表示图片、文本、视频等不同的媒体格式。而URI结构如下：

```java
<scheme>://<host>:<port>/[<path>|<pathPrefix>|<pathPattern>]
```

# 参考
* [Intent 和 Intent 过滤器](https://developer.android.com/guide/components/intents-filters.html?hl=zh)
* [Android开发艺术探索](#inner)
