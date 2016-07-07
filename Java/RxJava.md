# RxJava

## RxJava介绍

如果想了解RxJava，那么必须先要知道ReactiveX，ReactiveX是Reactive Extensions的缩写，Rx是一个函数库，可以让开发者利用可观察序列和各种操作符来编写异步和基于事件的程序，而ReactiveX.io给的定义则是，Rx是一个使用可观察数据流进行异步编程的编程接口，ReactiveX结合了观察者模式、迭代器模式和函数式的变成精华。

RxJava是ReactiveX在JVM上的一个实现。

## 创建Observable（被观察者/事件源）

在RxJava中，一个实现了Observer接口的对象订阅（subscribe）一个可观察对象（Observable）的实例，订阅者（Subscriber）对可观察对象（Observable）发射的任何数据或数据序列做出响应。其中Observable代表被观察者，它决定什么时候触发以及触发怎样的事件。RxJava使用操作符`create()`创建一个最基本的Observale。

```java
Observable<String> observable = Observable.create(new Observable.OnSubscribe<String>() {
    @Override
    public void call(Subscriber<? super String> subscriber) {
        subscriber.onNext("hello world");
        subscriber.onCompleted();
    }
});
```
出了除了`create()`方法，还有其他一些方法来创建Observable。
* [`just()`](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Just.md)：将一个或多个对象转换成发射这个或这些对象的Observable
```java
Observable<String> observable = Observable.just("hello world");
Observable<String> observable1 = Observable.just("hello", "world");
```    
* [`from()`](https://github.com/mcxiaoke/RxDocs/blob/master/operators/From.md)：将一个Iterable，一个Future，或者一个数组转换成一个Observable
```java
String[] array = {"hello", "world"};
Observable<String> observable = Observable.from(array);
```
* [`interval()`]()：创建一个按照给定的时间间隔发射整数序列的Observable
```java
// 每隔500毫秒发射一个整数
Observable<Long> observable = Observable.interval(500, TimeUnit.MILLISECONDS);
// 延迟500毫米发射一个整数，此后每隔250毫秒发射一个整数
Observable<Long> observable1 = Observable.interval(500, 250, TimeUnit.MILLISECONDS);
```
* [`timer()`]()：创建一个在给定的延迟时间之后发射单个数据的Observable
```java
Observable<Long> observable = Observable.timer(500, TimeUnit.MILLISECONDS);
```

## 操作数据（变换/过滤/组合）

Rx最强大的地方就在于其各种操作符，可以对Observable发射的数据或数据序列进行诸如变换、过滤、组合灯操作。

* [`map()`](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Map.md)：对序列的每一项都应用一个函数来变换Observable发射的数据
```java
Observable.just("hello world.")
	.map(new Func1<String, String>() {

		@Override
		public String call(String t) {
			// TODO Auto-generated method stub
			return t.toUpperCase();
		}
	})
	.subscribe(new Action1<String>() {

		@Override
		public void call(String t) {
			// TODO Auto-generated method stub
			System.out.println(t);
		}
	});
```
输出结果：`HELLO WORLD.`

* [`flatmap()`/`concatMap()`]()：使用一个指定的函数对原始Observable发射的每一项数据执行变换操作，这个函数返回一个本身也发射数据的Observable，然后`flatmap`（`concatMap`）合并这些Observable发射的数据，最后将合并的结果当做它自己的数据序列发射。这个操作符很有用，例如，当你有一个这样的Observable：它发射一个数据序列，这些数据本身包含Observable成员或者可以变换为Observable，因为你可以创建一个新的Observable发射这些次级Observable发射的数据的完整集合。

* [`filter()`](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Filter.md)：只发射通过了测试的数据
```java
Observable.just(1, 2, 3, 4, 5, 6, 7, 8, 9, 10)
	.filter(new Func1<Integer, Boolean>() {

		@Override
		public Boolean call(Integer t) {
			// TODO Auto-generated method stub
			return t%2 == 0;
		}
	})
	.subscribe(new Action1<Integer>() {

		@Override
		public void call(Integer t) {
			// TODO Auto-generated method stub
			System.out.println(t);
		}
	});
```
输出结果：`2 4 6 8 10`

* [`merge()`](https://github.com/mcxiaoke/RxDocs/blob/master/operators/Merge.md)：可以将多个Observable的输出合并
```java
Observable.merge(
	Observable.just(1,3,5,7,9),
	Observable.just(2,4,6,8,10))
	.subscribe(new Action1<Integer>() {

		@Override
		public void call(Integer t) {
			// TODO Auto-generated method stub
			System.out.println(t);
		}
	});
```
输出结果：`1 3 5 7 9 2 4 6 8 10`

## 创建Observer（观察者/事件处理）

当有了事件源后，就可以创建观察者Observer，它决定事件触发的时候讲有怎样的行为。

```java
Observer<String> observer = new Observer<String>() {

    // 事件队列完结，当不再有onNext发出时，需要触发onCompleted方法作为标志
	@Override
	public void onCompleted() {
		// TODO Auto-generated method stub

	}

    // 事件队列异常，在事件处理过程中出现异常，onError会被触发，同时队列终止
	@Override
	public void onError(Throwable e) {
		// TODO Auto-generated method stub

	}

    // 事件处理
	@Override
	public void onNext(String t) {
		// TODO Auto-generated method stub

	}
};
```
除了`Observer`接口外，还有一个实现了`Observer`接口的抽象类`Subscriber`，抽象类`Subscriber`对接口进行了扩展。一是增加了一个`onStart()`方法，它会在`Observable.subscribe`订阅刚开始，而事件还未发送之前被调用，可以做一些准备工作，例如数据清零等。二是增加了`unsubscribe`方法，它是`Subscriber`所实现的另一个接口`Subscription`的方法，用于取消订阅，这个方法被调用后，`Subscriber`不再接收事件。

```java
Subscriber<String> subscriber = new Subscriber<String>() {

	@Override
	public void onStart() {
		// TODO Auto-generated method stub
		super.onStart();
	}

	@Override
	public void onCompleted() {
		// TODO Auto-generated method stub

	}

	@Override
	public void onError(Throwable e) {
		// TODO Auto-generated method stub

	}

	@Override
	public void onNext(String t) {
		// TODO Auto-generated method stub

	}
};

subscriber.unsubscribe();
```

有时候，我们只需要关心事件处理或者事件处理过程中干的异常，而不关心什么时候事件队列结束。RxJava则为此提供了一些列的ActionX接口，它可以包装不同的无返回值的方法。可以使用Action来替代Subscriber。

```java
Action1<String> onNext = new Action1<String>() {

	@Override
	public void call(String t) {
		// TODO Auto-generated method stub

	}
};

Action1<Throwable> onError = new Action1<Throwable>() {

	@Override
	public void call(Throwable t) {
		// TODO Auto-generated method stub

	}
};

Action0 onCompleted = new Action0() {

	@Override
	public void call() {
		// TODO Auto-generated method stub

	}
};

Observable.just("hello").subscribe(onNext);

Observable.just("hello").subscribe(onNext, onError);

Observable.just("hello").subscribe(onNext, onError, onCompleted);
```

## 线程变换

在不指定线程的情况下，RxJava遵循的是线程不变原则，即：在哪个线程调用`subscribe`就在哪个线程生成事件，在哪个线程生产事件，就在哪个线程消费事件。如果想给Observable添加多线程功能，就要使用`Scheduler`调度器。

调度器种类：

|调度器类型|功能|
|:-----|:----------|
|Shedulers.computation()|用于计算任务，如事件循环和回调处理，不要用于IO操作，默认线程数等于处理器的数量|
|Schedulers.io()|用于IO密集型任务，如异步阻塞IO操作，这个调度器的线程池会根据需要增长|
|Schedulers.from(Executor)|使用指定的Executor作为调度器|
|Schedulers.immediate()|在当前线程立即开始执行任务|
|Schedulers.newThread()|为每一个任务创建一个新的线程|
|Schedulers.trampoline()|当其他排队的任务完成后，在当前线程排队开始执行|

有了这几个`Scheduler`后，就可以使用`subscribeOn`和`observeOn`两个方法来对线程控制。`subscribeOn`方法指定事件产生的线程，`observeOn`方法指定事件消费的线程。

```java
Observable.create(new OnSubscribe<String>() {

		@Override
		public void call(Subscriber<? super String> subscriber) {
			// TODO Auto-generated method stub
			System.out.println(Thread.currentThread().getName()+":call");
			subscriber.onNext("hello world.");
		}
	})
	.subscribeOn(Schedulers.computation())// 指定Observable线程
	.map(new Func1<String, String>() {

		@Override
		public String call(String t) {
			// TODO Auto-generated method stub
			System.out.println(Thread.currentThread().getName());
			return t.toUpperCase();
		}
	})
	.observeOn(Schedulers.newThread()) // 指定Subscriber线程
	.subscribe(new Action1<String>() {

		@Override
		public void call(String t) {
			// TODO Auto-generated method stub
			System.out.println(Thread.currentThread().getName()+":"+t);
		}
	});
```
输出结果：
```
RxComputationScheduler-1:call
RxNewThreadScheduler-2
RxNewThreadScheduler-1:HELLO WORLD.
```

# 参考
* [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
* [是时候学习 RxJava 了](http://android.jobbole.com/83416/)
* [ReactiveX/RxJava文档中文版](https://github.com/mcxiaoke/RxDocs)
