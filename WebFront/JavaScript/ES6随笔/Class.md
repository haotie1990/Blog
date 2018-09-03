# Class

## 定义

1. 类的数据类型就是函数，类本身就指向构造函数

``` js
class Point {
  // ...
}

typeof Point // "function"
Point === Point.prototype.constructor // true
```

2. 类的所有方法都定义在类的prototype属性上面

``` js
class Point {
  constructor() {
    // ...
  }

  toString() {
    // ...
  }

  toValue() {
    // ...
  }
}

// 等同于

Point.prototype = {
  constructor() {},
  toString() {},
  toValue() {},
};
```

3. 类的内部所有定义的方法，都是不可枚举的（non-enumerable）
4. 类的属性名，可以采用表达式

## constructor

constructor方法是类的默认方法，通过new命令生成对象实例时，自动调用该方法。一个类必须有constructor方法，如果没有显式定义，一个空的constructor方法会被默认添加。

类必须使用new调用，否则会报错

## Class表达式

``` js
const MyClass = class Me {
  getClassName() {
    return Me.name;
  }
};
```

上面代码使用表达式定义了一个类。需要注意的是，这个类的名字是MyClass而不是Me，Me只在 Class 的内部代码可用，指代当前类。

采用 Class 表达式，可以写出立即执行的 Class

``` js
let person = new class {
  constructor(name) {
    this.name = name;
  }

  sayName() {
    console.log(this.name);
  }
}('张三');

person.sayName(); // "张三"
```

## this的指向

类的方法内部如果含有this，它默认指向类的实例。但是，必须非常小心，一旦单独使用该方法，很可能报错。

``` js
class Logger {
  printName(name = 'there') {
    this.print(`Hello ${name}`);
  }

  print(text) {
    console.log(text);
  }
}

const logger = new Logger();
const { printName } = logger;
printName(); // TypeError: Cannot read property 'print' of undefined
```

解决方法：
1. 构造函数中绑定this
2. 使用箭头函数
3. 使用Proxy，获取方式时自动绑定实例

## 继承

Class 可以通过extends关键字实现继承，这比 ES5 的通过修改原型链实现继承，要清晰和方便很多

子类必须在constructor方法中调用super方法，否则新建实例时会报错

``` js
class Point { /* ... */ }

class ColorPoint extends Point {
  constructor() {
  }
}

let cp = new ColorPoint(); // ReferenceError
```

在子类的构造函数中，只有调用super之后，才可以使用this关键字，否则会报错。

1. super作为函数调用时，代表父类的构造函数。作为函数时，super()只能用在子类的构造函数之中，用在其他地方就会报错
2. super作为对象时，在普通方法中，指向父类的**原型对象**；在静态方法中，指向父类。由于super指向父类的原型对象，所以定义在父类实例上的方法或属性，是无法通过super调用的
3. 在子类普通方法中通过super调用父类的方法时，方法内部的this指向当前的子类实例
4. 使用super的时候，必须显式指定是作为函数、还是作为对象使用，否则会报错。