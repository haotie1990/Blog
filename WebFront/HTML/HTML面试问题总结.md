# HTML面试问题总结

##### 1.``<!DOCTYPE>``是什么？有什么作用？

* ``<!DOCTYPE>``声明位于文档中的最前面，处于``<html>``标签之前，告知浏览器以何种模式（严格模式/混杂模式）来渲染文档
* 严格模式的（标准模式）排版和``JS``运作模式是以该浏览器支持的最高标准运行
* 混杂（怪异模式）模式中，页面以宽松的向后兼容的方式显示，模拟老式浏览器的行为以防止站点无法工作
* ``<!DOCTYPE>``不存在或格式不正确导致文档以混杂模式呈现

##### 2.``cookie``、``localStorage``、``sessionStorage``的区别？

||cookie|localStorage|sessionStorage|
|--|--|--|--|
|由谁初始化|客户端或服务器，服务器可通过响应头``set-cookie``设置|客户端|客户端|
|过期时间|取决于设置时字段``Expires``和``Max-Age``|永不过期|当前窗口有效，打开一个新的同源窗口或重启浏览器都失效|
|在当前浏览器会话中是否保持不变|取决是否设置了过期时间，未设置过期时间则是会话期Cookie|是|否|
|是否与域名相关联|是|||
|是否跟随http请求发送给服务器|Cookie会自动通过请求头``Cookie``发送给服务器|||
|容量|4KB|5MB|5MB|
|访问权限|除了被``HttpOnly``标识任意窗口都可以访问|任意窗口|当前窗口|

##### 3.``<script>``、``<script async>``、``<script defer>``的区别？

* ``<script>``：HTML解析中断，脚步被提取并立即执行。执行结束后，HTML解析继续
* ``<script async>``：脚步的提取与解析过程，和HTML的解析过程并行。脚本执行完毕后可能在HTML解析完成之前，当脚步与页面其他脚本独立时，可以使用``async``，比如统计分析类的脚本
* ``<script defer>``：脚本仅提取过程与HTML解析过程并行，脚步的执行将在HTML解析完成后进行。如果有多个``defer``的脚本，脚本的执行顺序将按照其在文档中的出现顺序，从上到下进行

![whats-the-difference-between-async-vs-defer-attributes.jpeg](/Image/whats-the-difference-between-async-vs-defer-attributes.jpeg)

[彻底搞懂 async & defer](https://github.com/xiaoyu2er/blog/issues/8)

##### 4.`clientHeight`,`scrollHeight`,`offsetHeight`,以及`scrollTop`, `offsetTop`,`clientTop`的区别？

* clientHeight：表示的是可视区域的高度，不包含border和滚动条
* offsetHeight：表示可视区域的高度，包含了border和滚动条
* scrollHeight：表示了所有区域的高度，包含了因为滚动被隐藏的部分。
* clientTop：表示边框border的厚度，在未指定的情况下一般为0
* scrollTop：滚动后被隐藏的高度，获取对象相对于由offsetParent属性指定的父坐标(css定位的元素或body元素)距离顶端的高度。
