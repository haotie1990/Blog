# Generator

## 概念

1. Generator 函数是一个状态机，封装了多个内部状态。
2. 执行 Generator 函数会返回一个遍历器对象，返回的遍历器对象，可以依次遍历 Generator 函数内部的每一个状态。

## 特征

1. function关键字与函数名之间有一个星号
2. 函数体内部使用yield表达式，定义不同的内部状态（yield在英语里的意思就是“产出”）

调用 Generator 函数，返回一个遍历器对象，代表 Generator 函数的内部指针。以后，每次调用遍历器对象的next方法，就会返回一个有着value和done两个属性的对象。value属性表示当前的内部状态的值，是yield表达式后面那个表达式的值；done属性是一个布尔值，表示是否遍历结束

> yield表达式只能用在 Generator 函数里面，用在其他地方都会报错。
> yield表达式如果用在另一个表达式之中，必须放在圆括号里面

## next方法的参数

yield表达式本身没有返回值，或者说总是返回undefined。next方法可以带一个参数，该参数就会被当作上一个yield表达式的返回值

通过next方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。

> 由于next方法的参数表示上一个yield表达式的返回值，所以在第一次使用next方法时，传递参数是无效的

``` js
function* foo(x) {
  var y = 2 * (yield (x + 1));
  var z = yield (y / 3);
  return (x + y + z);
}

var a = foo(5);
a.next() // Object{value:6, done:false}
a.next() // Object{value:NaN, done:false}
a.next() // Object{value:NaN, done:true}

var b = foo(5);
b.next() // { value:6, done:false }
b.next(12) // { value:8, done:false }
b.next(13) // { value:42, done:true }
```

## for...of循环

一旦next方法的返回对象的done属性为true，for...of循环就会中止，且不包含该返回对象

``` js
function* foo() {
  yield 1;
  yield 2;
  yield 3;
  yield 4;
  yield 5;
  return 6;
}

for (let v of foo()) {
  console.log(v);
}
// 1 2 3 4 5
```

## Generator.prototype.throw()

throw方法抛出的错误要被内部捕获，前提是必须至少执行过一次next方法

``` js
function* gen() {
  try {
    yield 1;
  } catch (e) {
    console.log('内部捕获');
  }
}

var g = gen();
g.throw(1);
// Uncaught 1
```

throw方法被捕获以后，会附带执行下一条yield表达式。也就是说，会附带执行一次next方法。

``` js
var gen = function* gen(){
  try {
    yield console.log('a');
  } catch (e) {
    // ...
  }
  yield console.log('b');
  yield console.log('c');
}

var g = gen();
g.next() // a
g.throw() // b
g.next() // c
```

## Generator.prototype.return()

Generator 函数返回的遍历器对象，还有一个return方法，可以返回给定的值，并且终结遍历 Generator 函数

## yield* 

yield*表达式，用来在一个 Generator 函数里面执行另一个 Generator 函数

``` js
function* bar() {
  yield 'x';
  yield* foo();
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  yield 'a';
  yield 'b';
  yield 'y';
}

// 等同于
function* bar() {
  yield 'x';
  for (let v of foo()) {
    yield v;
  }
  yield 'y';
}

for (let v of bar()){
  console.log(v);
}
// "x"
// "a"
// "b"
// "y" 
```

## Generator的this

Generator 函数总是返回一个遍历器，ES6 规定这个遍历器是 Generator 函数的实例，也继承了 Generator 函数的prototype对象上的方法

> Generator内部没有this对象

``` js
function* g() {
  this.a = 11;
}

let obj = g();
obj.next();
obj.a // undefined
```

> Generator 函数也不能跟new命令一起用，会报错。

``` js
function* F() {
  yield this.x = 2;
  yield this.y = 3;
}

new F()
// TypeError: F is not a constructor
```

改造Generator函数使其可以拥有this对象并使用new关键字

``` js
function* gen() {
  this.a = 1;
  yield this.b = 2;
  yield this.c = 3;
}

function F() {
  return gen.call(gen.prototype);
}

var f = new F();

f.next();  // Object {value: 2, done: false}
f.next();  // Object {value: 3, done: false}
f.next();  // Object {value: undefined, done: true}

f.a // 1
f.b // 2
f.c // 3
```

## Generator与协程

可以并行执行、交换执行权的线程（或函数），就称为协程

## 应用

### 异步操作的同步化表达

。。。