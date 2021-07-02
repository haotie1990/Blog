EventLoop是学习JavaScript无法避开的话题，无论是在浏览器环境下还是在Node.js环境下

先看下面两个题,写出代码执行结果

```javascript
console.log(1);
setTimeout(() => {
  console.log(2);
});
process.nextTick(() => {
  console.log(3);
});
setImmediate(() => {
  console.log(4);
});
new Promise((resolve) => {
  console.log(5);
  resolve();
  console.log(6);
}).then(() => {
  console.log(7);
});
Promise.resolve().then(() => {
  console.log(8);
  process.nextTick(() => {
    console.log(9);
  });
});

// Tasks:[2,4] Jobs:[3, 7, 8] 8->9
// 1 5 6 3 7 8 9 2 4
```

上一个题的变体
```javascript
console.log(1);
setImmediate(() => {
  console.log(4);
});
setTimeout(() => {
  console.log(2);
});
new Promise((resolve) => {
  console.log(5);
  resolve();
  console.log(6);
}).then(() => {
  console.log(7);
});
process.nextTick(() => {
  console.log(3);
});
Promise.resolve().then(() => {
  console.log(8);
  process.nextTick(() => {
    console.log(9);
  });
});
// task:[2, 4] jobs:[3, 7, 8]
// 1 5 6 3 7 8 9 2 4
```

```javascript
/*
new Promise((resolve) => {
  console.log("async1 start");
  return new Promise((resolve) => {
    console.log("async2");  
  }).then(() => {
    console.log("async1 end");
  })
})
*/
async function async1() {
  console.log("async1 start");
  await async2();
  console.log("async1 end");
}
async function async2() {
  console.log("async2");
}
console.log("script start");
setTimeout(function () {
  console.log("setTimeout");
}, 0);
async1();
new Promise(function (resolve) {
  console.log("promise1");
  resolve();
}).then(function () {
  console.log("promise2");
});
console.log("script end");

// Tasks:[setTimeout] Jobs:[async1 end, promise2]
// script start -> async1 start -> async2 -> promise1 -> script end -> async1 end -> promise2 -> setTimeout
```

```html
<script>
  // task:[script tag1 , script tag 2, 2, 5] jobs:[4, 7]
  // script tag 1 -> 3 -> 4 -> script tag 2 -> 6  -> 7 -> 2 -> 5
  console.log('script tag 1');
  setTimeout(() => {
    console.log('2');
  });
  new Promise((resolve) => {
    console.log('3')
    resolve();
  }).then(() => {
    console.log('4')
  })
</script>

<script>
  console.log('script tag 2');
  setTimeout(() => {
    console.log('5');
  });
  new Promise((resolve) => {
    console.log('6')
    resolve();
  }).then(() => {
    console.log('7')
  })
</script>
```

### 总结

1. 关键词：执行栈（同步任务）、任务队列
2. 宏任务类型（Macrotask/Task）: setTimeout、setInterval、setImmediate、I/O、Script Tag（第四题）
3. 微任务类型（Microtask/Jobs）: Promise/Async、MutationObserver、process.nextTick
4. 每一次事件循环中，从宏任务队列中提取一个任务执行，然后检查微任务队列如果不为空则清空微任务队列，并且在执行微任务的过程中随时可向微任务队列添加任务（注意上面第一题，在打印输出``8``后会立即打印输出``9``），直到微任务队列为空为止。然后判断是否需要进行UI渲染（只在浏览器环境存在）。
5. Node.js环境下微任务process.nextTick类型会先于Promise类型的任务执行
6. Node.js环境下宏任务setTimeout/setInterval会先于setImmediate类型的任务执行
7. Node.js环境下宏任务执行顺序（粗略）: setTimeout/setInterval -> I/O -> setImmediate

### 参考

* [理解JavaScript中的宏任务（Macrotask）和微任务（Microtask）](https://juejin.cn/post/6844903471280291854)
* [HTML系列：Macrotask和Microtask](https://zhuanlan.zhihu.com/p/24460769)
* [Event loop 及 macrotask & microtask](https://zhuanlan.zhihu.com/p/76131519)
* [JavaScript 事件循环：从起源到浏览器再到 Node.js](https://mp.weixin.qq.com/s?__biz=MzI5NjM5NDQxMg==&mid=2247491540&idx=1&sn=820020db13f6687de94c2f6b4d7a3fdf&scene=21#wechat_redirect)
* [setTimeout和setImmediate到底谁先执行，本文让你彻底理解Event Loop](https://juejin.cn/post/6844904100195205133)