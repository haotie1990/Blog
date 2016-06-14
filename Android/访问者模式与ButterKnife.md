# 访问者模式与ButterKnife

## 访问者模式

### 1. 访问者模式介绍

访问者模式是一种将数据操作与数据结构分离的设计模式。在一个软件系统中拥有一个由许多对象构成，比较稳定的对象结构，这些类都有一个`accept`方法来接收访问者对象的访问。访问者是一个接口，它拥有一个`vist`方法，这个方法会对访问到的对象结构中不同类型的元素做出不同的处理。

### 2. 访问者模式定义

封装一些作用于某种数据结构中的各元素的操作，它可以在不改变这个数据结构的前提下定义作用于这些元素的新的操作。

### 3. 访问者模式使用场景

* 对象结构比较稳定，但经常需要在此对象结构上定义新的操作
* 需要对一个对象结构中的对象进行很多不同步的并且不相关的操作，而需要避免这些操作“污染”这些对象的类，也不希望在增加新操作时修改这些类。

### 4. 访问者模式UML

![](../Image/1335165175_6219.jpg)

## ButterKnife

### 1. ButterKnife介绍

ButterKnife是通过注解处理生成样板代码来完成`Android View`的属性和方法的绑定的类库。它简化了以往Android开发中初始化`view`和设置监听的操作。

### 2. Butterknife特性

* 通过使用`@BindView`代替`findViewById`初始化`view`
* 可以同时处理列表或数组中的多个`view`
* 使用类似`@OnClick`的注解替代原有匿名内部类监听方法
* 使用类似`@BindString`的资源注解来替代原有的`getResouces()`查询资源方式

### 3. ButterKnife使用

# 参考

* [Android源码设计模式解析与实战](http://product.dangdang.com/23802445.html)
* [绝对不容错过，ButterKnife使用详谈](http://www.jianshu.com/p/b6fe647e368b)
* [Butter Knife 源码解析](http://mp.weixin.qq.com/s?__biz=MzA4MjU5NTY0NA==&mid=404147665&idx=1&sn=a16153b2a658db64ab80926cd3b76447&scene=2&srcid=0316ZcLDaO7NOqcReomltlmA&from=timeline&isappinstalled=0#wechat_redirect)
* [公共技术点之 Java 注解 Annotation](http://a.codekk.com/detail/Android/Trinea/%E5%85%AC%E5%85%B1%E6%8A%80%E6%9C%AF%E7%82%B9%E4%B9%8B%20Java%20%E6%B3%A8%E8%A7%A3%20Annotation)
* [Butter Knife](http://jakewharton.github.io/butterknife/)
