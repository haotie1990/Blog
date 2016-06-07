
# 观察者模式与EventBus3

## 1. 观察者模式

### 1.1 观察者模式定义

定义对象间一种一对多的依赖关系，使得每当一个对象改变状态，则所有的依赖于它的对像都会得到通知并被自动更新

### 1.2 观察者模式的使用场景

* 关联行为场景
* 事件多级触发场景
* 跨系统的消息交换场景

### 1.3 观察者模式的UML

### 1.4 Android源码中的观察者模式

## 2.EventBus

### 2.1 EventBus简介

EventBus是一个开源的基于发布/订阅模式的用来解决对象间耦合问题的Android库。

![EventBus框架](../Image/EventBus-Publish-Subscribe.png)

### 2.2 EventBus特性

* 基于注解的，使用`@Subscribe`注解事件订阅函数。
* 订阅函数可异步，通过`ThreadMode`，可以设置订阅函数运行在Main线程还是异步后台线程
* 事件和订阅支持继承，发布子类的事件，则订阅了父类事件的函数也会接收到消息。
* 支持定制`EventBus`，通过`EventBusBuilder`可以定制`EventBus`

### 2.3 使用EventBus3

#### 2.3.1 定义事件

EventBus中的事件即是一个简单的Java对象
```java
public class MessageEvent {
    public String message;

    public MessageEvent(String message) {
        this.message = message;
    }
}
```

#### 2.3.2 准备订阅函数

如果事件被发布，那么事件订阅函数就会被调用。在EventBus3使用注解`@Subscribe`标注事件订阅函数，函数的命名可以自由进行，不再像EventBus2那也有严格的限制。

```java
// This method will be called when a MessageEvent is posted
@Subscribe
public void onMessageEvent(MessageEvent event){
    Toast.makeText(getActivity(), event.message, Toast.LENGTH_SHORT).show();
}
```

当定义好事件订阅函数后，需要将其注册到EventBus中，也只有事件订阅函数被注册，其才能接收到相应的事件。在Activity和Fragment中通常会根据他们的生命周期完成注册和取消注册。

```java
@Override
public void onStart() {
    super.onStart();
    EventBus.getDefault().register(this);
}

@Override
public void onStop() {
   EventBus.getDefault().unregister(this);
    super.onStop();
}
```

#### 2.3.3 发布事件

可以在代码中的任何位置发布事件，但同时也需要注意事件订阅函数和事件发布的运行线程。

```java
EventBus.getDefault().post(new MessageEvent("Hello world!"));
```

### 2.4 EventBus3的异步

EventBus3可以设置订阅函数的执行线程，通过在注解`@Subscribe`中设置`threadMode`。`ThreadMode`有四种选择，分别是`POSTING`、`MAIN`、`BACKGROUND`、`ASYNC`。

#### 2.4.1 ThreadMode.POSTING

#### 2.4.2 ThreadMode.MAIN

#### 2.4.3 ThreadMode.BACKGROUND

#### 2.4.4 ThreadMode.ASYNC

### 2.5 定制EventBus3

### 2.6 Sticky Events

### 2.7 订阅函数优先级和取消事件传递

### 2.8 订阅索引

### 2.9 代码混淆

### 2.10 EventBus中的AsyncExecutor


# 参考

* [Android源码设计模式解析与实战](http://product.dangdang.com/23802445.html)

*  [EventBus](http://greenrobot.org/eventbus/)