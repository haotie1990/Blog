# JavaScript的作用域与闭包

## var、let、const的区别

### 1、var声明的变量具有如下性质

* 全局作用域及函数作用域
* 在其作用域内变量提升
* 可重复声明

### 2、let声明的变量具有如下性质

* 块级作用域
* 可修改但不可重复声明
* 在代码块内，let命令声明的变量之前，该变量都是不可用的，语法上成为[“暂时性死区”](https://segmentfault.com/a/1190000015603779)

看下的几段代码
```javascript
for (let i = 0; i < 10; i++) {
  // ...
}

console.log(i);
// ReferenceError: i is not defined 块级作用域，在作用域外无法访问
```

```javascript
var a = [];
/**
 * 由于var造成的变量提升，实际相当于
 * var i;
 * for(i = 0; i < 10; i++) {}
 */
for (var i = 0; i < 10; i++) {
  a[i] = function () {
    console.log(i);
  };
}
a[6](); // 10 变量i是一个全局作用域内的变量，当方法执行的时候会从其创建时刻的作用域开始向上寻找变量
```

```javascript
for (let i = 0; i < 3; i++) {
  let i = 'abc';
  console.log(i);
}
// abc for循环设置循环变量部分和循环体分属两个作用域
// abc
// abc
```

### 3、const声明的变量具有如下性质

* 块级作用域
* 不可被修改且不可被重新声明
* 声明的变量必须在被声明时初始化

### 参考

* [闭包](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Closures)
* [图解Javascript上下文与作用域](http://blog.rainy.im/2015/07/04/scope-chain-and-prototype-chain-in-js/)
* [JavaScript深入之作用域链](https://github.com/mqyqingfeng/Blog/issues/6)
* [JavaScript中的作用域](https://www.w3cplus.com/javascript/the-scope-in-javascript-explained.html)
* [ES6箭头函数和它的作用域](https://www.w3cplus.com/javascript/arrow-functions-and-their-scope.html)
* [JavaScript 变量作用域、this、闭包](http://dongss.cn/posts/3.html)