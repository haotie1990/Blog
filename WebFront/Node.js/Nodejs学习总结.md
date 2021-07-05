### Node.js学习总结

#### 1. 什么是Node.js

Node.js是基于Chrome V8 JavaScript引擎的JavaScript运行时环境。

![nodejs-architechture](/asset/a9e67142615f49863438cc0086b594e48984d1c9.jpeg)

> 1. Node bindings将Chrome v8引擎暴露的c/c++接口转换为JavaScript API，并结合这些API编写Node.js标准库，我们将所有这些API称为Node.js SDK
> 2. Chrome v8引擎负责解释并执行JavaScript代码
> 3. libuv则由事件循环和线程池组成，负责所有的I/O任务的分发与执行

#### 2. Node.js的特点

#### 3. Node.js环境与浏览器环境的区别

#### 4. Node.js的CommonJS模块系统

#### 5. Node.js的进程处理

#### 6. Node.js的异步编程及EventLoop

#### 7. Node.js的HTTP

#### 8. Node.js的Buffer及Stream

#### 9. Node.js的Debug

#### 10. Node.js的性能调优