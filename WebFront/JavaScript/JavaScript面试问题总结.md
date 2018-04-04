# JavaScript面试问题总结

##### 1.``==``与``===``的区别？

##### 2.``this``的理解？

##### 3.什么是闭包（closure）？

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

### 参考

* [前端回忆录-前端面试](https://github.com/Wscats/Good-Text-Share#%E5%89%8D%E7%AB%AF%E9%9D%A2%E8%AF%95)
* [readyState的五种状态](https://blog.csdn.net/allen19901008/article/details/38680857)
* [一道常被人轻视的前端JS面试题](http://www.cnblogs.com/xxcanghai/p/5189353.html)
* [带你理解 JS 容易出错的坑和细节](https://juejin.im/post/59f54321f265da43085d4a7f?utm_medium=fe&utm_source=weixinqun)