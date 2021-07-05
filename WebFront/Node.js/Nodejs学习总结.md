### Node.js学习总结

#### 1. 什么是Node.js

Node.js是基于Chrome V8 JavaScript引擎的JavaScript运行时环境。

![nodejs-architechture](/asset/a9e67142615f49863438cc0086b594e48984d1c9.jpeg)

> 1. Node bindings将Chrome v8引擎暴露的c/c++接口转换为JavaScript API，并结合这些API编写Node.js标准库，我们将所有这些API称为Node.js SDK
> 2. Chrome v8引擎负责解释并执行JavaScript代码
> 3. libuv则由事件循环和线程池组成，负责所有的I/O任务的分发与执行

#### 2. Node.js的特点

Node.js是可扩展的构建高性能Web应用的最简单解决方案。

Node.js可以说有四个特点：构建Web应用、高性能、可扩展、简单。

在构建Web应用方面，Node.js可以说有非常多的应用。包括：创建网站（基于Nuxt.js或Next.js框架构建网站应用服务）、后端服务（Express、Koa、Egg.js）、PC客户端（Electron）、APP客户端（React Native、Weex）、前端工程化及构建工具（Webpack、Rollup、Gulp）

高性能：Chrome V8加持在解释型动态语言中执行速度最快。天生异步（事件驱动及非阻塞I/O）使其适用I/O密集型的网络任务处理。

可扩展：一是npm拥有世界上最大的开源工具包；二是可以基于C/C++编写Node.js插件扩展Node.js功能，比如使其支持CPU密集型任务处理。

简单：一是JavaScript编程语言简单易学；二是Node.js的单线程执行、异步编程模型让并发编程更加简单。

#### 3. Node.js环境与浏览器环境的区别

#### 4. Node.js的CommonJS模块系统

#### 5. Node.js的进程处理

#### 6. Node.js的异步编程及EventLoop

#### 7. Node.js的HTTP

#### 8. Node.js的Buffer及Stream

#### 9. Node.js的Debug

#### 10. Node.js的性能调优

### 参考

1. 更了不起的Node.js（卷1）
2. [深入理解Node.js：核心思想与源码分析](https://yjhjstz.gitbooks.io/deep-into-node/content/chapter1/chapter1-0.html)
3. [Node.js架构剖析](http://www.ayqy.net/blog/node-js-architecture-overview/)