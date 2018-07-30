# JavaScript面试问题总结

##### 1.``==``与``===``的区别？

##### 2.``this``的理解？

##### 3.什么是闭包（closure）？

一句话可以概括：闭包就是能够读取其他函数内部变量的函数，或者子函数在外调用，子函数所在的父函数的作用域不会被释放。

##### 4.JavaScript中的作用域和变量声明提升？

##### 5.``call``和``apply``的区别？

[Javascript中的apply和call继承](https://github.com/Wscats/Good-text-Share/issues/56)

[JavaScript深入之call和apply的模拟实现](https://juejin.im/post/5907eb99570c3500582ca23c)

##### 6.javascript的浅拷贝和深拷贝？

[面试官:请你实现一个深克隆](https://juejin.im/post/5abb55ee6fb9a028e33b7e0a)
[Javascript深浅拷贝](https://github.com/Wscats/Good-text-Share/issues/57)

##### 7.javascript中的类型检测？

##### 8.javascript实现快速排序？

[归并排序与快速排序的简明实现及对比](https://juejin.im/post/5a2969656fb9a0452405bbb4?utm_medium=fe&utm_source=weixinqun)

[JS中可能用得到的全部的排序算法](https://juejin.im/entry/5a0a86d15188257bfe455cba?utm_medium=fe&utm_source=weixinqun)

##### 9.javascript实现二分查找？

##### 10.event loop？

[Event Loop 必知必会（六道题）](https://zhuanlan.zhihu.com/p/34182184)

##### 11.AJAX的``readystate``的状态？

|readystate code | 说明 | 
|--|--|
|0：uninitialized（未初始化）|the object has been created but not initialized.(the open method has not be called);(``XMLHttpRequest``)对象已经创建，但尚未初始化（open方法也未被调用）|
|1：loading（载入）|the object has been created,but the send method has not been called.(``XMLHttpRequest``)对象已经创建，但尚未调用``send``方法|
|2：loaded（载入完成）|the send method has been called,but the status and headers are not yet available.``send``方法已经被调用|
|3：interactive（交互）|some data has been received,calling the responseBody and responseText properties at this state to obtai partial results will return an error,because status and response headers are not fully available.|
|4：completed（完成）|all the data has been received,and the complete data is available in the responseBody and responseText properties.|

##### 12.什么是`MVVM`?

MVVM 由以下三个内容组成

* View：界面
* Model：数据模型
* ViewModel：作为桥梁负责沟通 View 和 Model

在 JQuery 时期，如果需要刷新 UI 时，需要先取到对应的 DOM 再更新 UI，这样数据和业务的逻辑就和页面有强耦合。

在 MVVM 中，UI 是通过数据驱动的，数据一旦改变就会相应的刷新对应的 UI，UI 如果改变，也会改变对应的数据。这种方式就可以在业务处理中只关心数据的流转，而无需直接和页面打交道。ViewModel 只关心数据和业务的处理，不关心 View 如何处理数据，在这种情况下，View 和 Model 都可以独立出来，任何一方改变了也不一定需要改变另一方，并且可以将一些可复用的逻辑放在一个 ViewModel 中，让多个 View 复用这个 ViewModel。

在 MVVM 中，最核心的也就是数据双向绑定，例如 Angluar 的脏数据检测，Vue 中的数据劫持

**数据劫持**

Vue 内部使用了 `Obeject.defineProperty()` 来实现双向绑定，通过这个函数可以监听到 `set` 和 `get` 的事件。

```js
var data = { name: 'yck' }
observe(data)
let name = data.name // -> get value
data.name = 'yyy' // -> change value

function observe(obj) {
  // 判断类型
  if (!obj || typeof obj !== 'object') {
    return
  }
  Object.keys(data).forEach(key => {
    defineReactive(data, key, data[key])
  })
}

function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log('get value')
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
    }
  })
}

```

以上代码简单的实现了如何监听数据的 `set` 和 `get` 的事件，但是仅仅如此是不够的，还需要在适当的时候给属性添加发布订阅

``` jsx
<div>
    {{name}}
</div>
```

在解析如上模板代码时，遇到 `{{name}}` 就会给属性 `name` 添加发布订阅。

```js
// 通过 Dep 解耦
class Dep {
  constructor() {
    this.subs = []
  }
  addSub(sub) {
    // sub 是 Watcher 实例
    this.subs.push(sub)
  }
  notify() {
    this.subs.forEach(sub => {
      sub.update()
    })
  }
}
// 全局属性，通过该属性配置 Watcher
Dep.target = null

function update(value) {
  document.querySelector('div').innerText = value
}

class Watcher {
  constructor(obj, key, cb) {
    // 将 Dep.target 指向自己
    // 然后触发属性的 getter 添加监听
    // 最后将 Dep.target 置空
    Dep.target = this
    this.cb = cb
    this.obj = obj
    this.key = key
    this.value = obj[key]
    Dep.target = null
  }
  update() {
    // 获得新值
    this.value = this.obj[this.key]
    // 调用 update 方法更新 Dom
    this.cb(this.value)
  }
}
var data = { name: 'yck' }
observe(data)
// 模拟解析到 `{{name}}` 触发的操作
new Watcher(data, 'name', update)
// update Dom innerText
data.name = 'yyy' 
```

接下来,对 `defineReactive` 函数进行改造

```js
function defineReactive(obj, key, val) {
  // 递归子属性
  observe(val)
  let dp = new Dep()
  Object.defineProperty(obj, key, {
    enumerable: true,
    configurable: true,
    get: function reactiveGetter() {
      console.log('get value')
      // 将 Watcher 添加到订阅
      if (Dep.target) {
        dp.addSub(Dep.target)
      }
      return val
    },
    set: function reactiveSetter(newVal) {
      console.log('change value')
      val = newVal
      // 执行 watcher 的 update 方法
      dp.notify()
    }
  })
}
```

以上实现了一个简易的双向绑定，核心思路就是手动触发一次属性的 getter 来实现发布订阅的添加。

##### 13.路由原理

前端路由实现起来其实很简单，本质就是监听 URL 的变化，然后匹配路由规则，显示相应的页面，并且无须刷新。目前单页面使用的路由就只有两种实现方式

* hash 模式
* history 模式

`www.test.com/#/` 就是 `Hash URL`，当 `#` 后面的哈希值发生变化时，不会向服务器请求数据，可以通过 `hashchange` 事件来监听到 `URL` 的变化，从而进行跳转页面。

![](/Image/164888109d57995f.jpg)

`History` 模式是 `HTML5` 新推出的功能，比之 `Hash URL` 更加美观

![](/Image/164888478584a217.jpg)


### 参考

* [前端回忆录-前端面试](https://github.com/Wscats/Good-Text-Share#%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95)
* [readyState的五种状态](https://blog.csdn.net/allen19901008/article/details/38680857)
* [一道常被人轻视的前端JS面试题](http://www.cnblogs.com/xxcanghai/p/5189353.html)
* [带你理解 JS 容易出错的坑和细节](https://juejin.im/post/59f54321f265da43085d4a7f?utm_medium=fe&utm_source=weixinqun)
* [关于Object的getter和setter](https://zhuanlan.zhihu.com/p/25672454?utm_source=wechat_session&amp;utm_medium=social)
* [44 道 JavaScript 难题（JavaScript Puzzlers!）](https://juejin.im/post/5b1f899fe51d4506c60e46ee?utm_source=gold_browser_extension)