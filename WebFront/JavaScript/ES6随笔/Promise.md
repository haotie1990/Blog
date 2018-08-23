# Promise

## Promise含义

1. Promise对象代表一个异步操作，有三种状态：pending（进行中）、fulfilled（已成功）和rejected（已失败）
2. Promise对象的状态改变，只有两种可能：从pending变为fulfilled和从pending变为rejected

> Promise也有一些缺点。首先，无法取消Promise，一旦新建它就会立即执行，无法中途取消。其次，如果不设置回调函数，Promise内部抛出的错误，不会反应到外部。第三，当处于pending状态时，无法得知目前进展到哪一个阶段（刚刚开始还是即将完成）

## 基本用法

1. Promise 新建后立即执行。然后，then方法指定的回调函数，将在当前脚本所有同步任务执行完才会执行

2. resolve函数的参数除了正常的值以外，还可能是另一个 Promise 实例，作为参数的Promise将接管发出resolve的Promise的状态

## Promise.prototype.then()

then方法返回的是一个新的Promise实例（注意，不是原来那个Promise实例）。因此可以采用链式写法，即then方法后面再调用另一个then方法

## Promise.prototype.catch()

1. Promise.prototype.catch方法是.then(null, rejection)的别名，用于指定发生错误时的回调函数

2. 如果 Promise 状态已经变成resolved，再抛出错误是无效的

3. 一般来说，不要在then方法里面定义 Reject 状态的回调函数（即then的第二个参数），总是使用catch方法。理由是第二种写法可以捕获前面then方法执行中的错误，也更接近同步的写法（try/catch）

> 跟传统的try/catch代码块不同的是，如果没有使用catch方法指定错误处理的回调函数，Promise 对象抛出的错误不会传递到外层代码，即不会有任何反应

> Promise 内部的错误不会影响到 Promise 外部的代码，通俗的说法就是“Promise 会吃掉错误”

## Promise.prototype.finally()

finally方法用于指定不管 Promise 对象最后状态如何，都会执行的操作

finally方法的回调函数不接受任何参数

## Promise.all()

Promise.all方法用于将多个 Promise 实例，包装成一个新的 Promise 实例。

Promise.all方法的参数可以不是数组，但必须具有 Iterator 接口，且返回的每个成员都是 Promise 实例

> 如果作为参数的 Promise 实例，自己定义了catch方法，那么它一旦被rejected，并不会触发Promise.all()的catch方法

## Promise.race()

Promise.race方法同样是将多个 Promise 实例，包装成一个新的 Promise 实例。若其中有一个实例率先改变状态，Promise.race的状态跟着改变，同时那个率先改变的 Promise 实例的返回值，就传递给Promise.race的回调函数

## Promise.try()

...