# async 函数

async 函数是什么？一句话，它就是 Generator 函数的语法糖

## async对比Generator的改进

### 1. 内置执行器

Generator 函数的执行必须靠执行器，所以才有了co模块，而async函数自带执行器。

### 2. 更好的语义

async和await，比起星号和yield，语义更清楚了。async表示函数里有异步操作，await表示紧跟在后面的表达式需要等待结果。

### 3. 更广泛的适用性

co模块约定，yield命令后面只能是 Thunk 函数或 Promise 对象，而async函数的await命令后面，可以是 Promise 对象和原始类型的值（数值、字符串和布尔值，但这时等同于同步操作）

### 4. 返回值是Promise

async函数的返回值是 Promise 对象，这比 Generator 函数的返回值是 Iterator 对象方便多了。你可以用then方法指定下一步的操作

## 语法

1. async函数返回一个 Promise 对象。
2. async函数内部return语句返回的值，会成为then方法回调函数的参数。
3. async函数内部抛出错误，会导致返回的 Promise 对象变为reject状态。抛出的错误对象会被catch方法回调函数接收到
4. async函数返回的 Promise 对象，必须等到内部所有await命令后面的 Promise 对象执行完，才会发生状态改变，除非遇到return语句或者抛出错误。也就是说，只有async函数内部的异步操作执行完，才会执行then方法指定的回调函数。
5. await命令后面是一个 Promise 对象。如果不是，会被转成一个立即resolve的 Promise 对象
6. 如果await后面的异步操作出错，那么等同于async函数返回的 Promise 对象被reject

``` js
async function f() {
  return 'hello world';
}

f().then(v => console.log(v))
// "hello world"
```

``` js
async function f() {
  throw new Error('出错了');
}

f().then(
  v => console.log(v),
  e => console.log(e)
)
// Error: 出错了
```