# Android中的线程操作

Android沿用了Java的线程模型，其中的线程也分为主线程和子线程，主线程也叫UI线程。主线程的作用是运行四大组件以及处理它们和用户的交互，而子线程（也叫工作线程）的作用则是执行耗时任务，如果网络请求、IO操作等。所以有时候我们为了处理一些耗时的操作，必须单独创建一个线程去执行相应的任务。

## Android中的线程形态

Android中的线程异步方案除了传统的Thread以外，还包括AsyncTask、HandlerThread以及IntentService，将依次介绍它们。

### Java中的线程

Java使用Thread类代表线程，所有的线程对象都必须是Thread类或者其子类的实例。

#### 1. 继承Thread类创建线程

通过继承Thread类来创建并启动多线程：

1. 定义Thread类的子类，并重写该类的run()方法，该run()方法体代表了线程需要完成的任务。
2. 创建Thread子类的实例，即创建了线程对象
3. 调用线程对象的start()方法来启动该线程

```java
public class ThreadTest {

	static class WorkThread extends Thread{
		@Override
		public void run() {
			// TODO Auto-generated method stub
            // 在此方法中执行线程任务
            for (int i = 0; i < 100; i++) {
				System.out.println(i);
			}
		}
	}
	public static void main(String[] args) {
		// TODO Auto-generated method stub
        // 创建并启动线程
		WorkThread workThread = new WorkThread();
		workThread.start();
	}

}
```

#### 2. 实现Runnable接口创建线程

实现Runnable接口来创建并启动线程：

1. 定义Runnable接口的实现类，并重写该接口的run()方法，该run()方法的方法体同样是该线程的线程执行体
2. 创建Runnable实现类的实例，并以此实例作为Thread的target来创建Thread对象，该Thread对象才是真正的线程对象。

```java
public class RunnableTest {

	static class WorkThread implements Runnable{
		@Override
		public void run() {
			// TODO Auto-generated method stub
			System.out.println(Thread.currentThread().getName());
		}
	}

	public static void main(String[] args) {
		// TODO Auto-generated method stub
		WorkThread workThread = new WorkThread();
		new Thread(workThread).start();
	}

}
```

#### 3. 使用Callable和Future创建线程

Thread和Runnable的run()方法的返回值是void，它做的事只是单纯的去执行run()方法中的代码，而Java提供了Callable接口，可看做Runnable接口的增强版。Callable接口提供了一个call()方法作为线程执行体，call()方法可以有返回值，也可以声明抛出异常。但Callable接口不是Runnable接口的子接口，所以Callable对象不能作为Thread的target。而且call()方法有一个返回值——call()方法并不能直接调用，它是作为线程执行体被调用的。因此Java还提供了Future接口来代表Callable接口里call()方法的返回值，并未Future接口提供了一个FutureTask实现类，该实现类同时实现了Runnable接口——可以作为Thread类的target。这样Callable和FutureTask配合起来就可以实现多线程。

这其实是一个很有用的功能，因为多线程相比单线程更难、更复杂的一个重要原因是因为多线程充满着未知，某个线程是否执行？某个线程执行了多久？某个线程执行的时候我们期望的数据是否已经完成？无法得知，我们只能等待这个线程任务执行完成。而Callable+FutureTask却可以获取线程运行结果，也可以在等待事件太长没有获取到结果的情况下取消该线程的任务。

Callable+FutureTask实现多线程：

1. 创建Callable接口的实现类，并实现call()方法，该call()方法即作为线程的执行体。
2. 创建Callable实现类的实例，并使用FutureTask类来包装Callable对象，该FutureTask封装了Callable对象call()方法的返回值
3. 使用FutureTask对象实例作为Thread的target创建并启动线程
4. 调用FutureTask的get()方法来获得子线程的执行结果，该方法将导致程序阻塞，必须等到子线程结果后才会得到返回值。

```java
public class CallableTest {

	static class WorkThread implements Callable<Integer>{

		@Override
		public Integer call() throws Exception {
			// TODO Auto-generated method stub
			// 线程执行体
			int sum = 0;
			for (int i = 1; i <= 100; i++) {
				sum += i;
			}
			return sum;
		}
	}

	public static void main(String[] args) throws InterruptedException, ExecutionException {
		WorkThread workThread = new WorkThread();
		FutureTask<Integer> futureTask = new FutureTask<>(workThread);
		// 将futureTask作为target创建Thread对象并启动线程
		new Thread(futureTask).start();
		// 将阻塞直到线程执行完成并返回结果
		System.out.println(futureTask.get());
	}
}
```

Runnable接口和Callable的性质类似，那么比较一个Thread和Runnable实现多线程的区别：
* Java不支持多继承，如果继承Thread类实现多线程，则将失去继承他来的机会。因为实现Runnable接口还可以继承其他类
* 实现Runnable接口，可以是多个线程Thread共享同一个target对象，适合多个线程来处理同一个资源的情况

### AsyncTask

AsyncTask是一种轻量级的异步任务，它可以在线程池中执行后台任务，然后把执行的进度和最终结果传递给主线程并在主线程中更新UI，从实现上来说，AsyncTask封装了Thread和Handler，通过AsyncTask可以更加方便的执行后台任务以及在主线程中访问UI。

AsyncTask是一个抽象的泛型，他提供了Params、Progress和Result这三个泛型参数，其中Params表示参数类型，Progress表示后台任务的执行进度类型，而Result则表示后台任务的放回结果类型。如果AsyncTask确定不需具体的参数，那么这三个泛型参数可以用Void代替。

AsyncTask提供了四个核心方法：

* onPreExecute()，在主线程中执行，在异步任务开始执行之前，自方法会被调用，做一些准备工作。
* doInBackground(Pramss...params)，在线程池中执行，此方法用于执行异步任务，params表示异步任务的输入参数。此方法中还可以通过publishProgress()方法来更新任务的进度，publishProgress()方法会调用onProgressUpdate()方法。此方法需要返回执行结果给onPostExecute()方法。
* onProgressUpdate(Progress...values)，在主线程中执行，当后台执行任务的执行进度发生变化此方法会被调用
* onPostExecute(Result result)，在主线程中执行，在异步任务执行完成后，此方法会被调用，其中result为后台任务的返回值，即doInBackground()的返回值。

此外AsyncTask还提供一个onCancelled()方法，它同样在主线程中执行，当异步任务被取消时，onCancelled()方法会被调用，同时onPostExecute方法则不会被调用。

```java
class WorkTask extends AsyncTask<String, Void, Bitmap>{

    @Override
    protected void onProgressUpdate(Void... values) {
    }

    @Override
    protected Bitmap doInBackground(String... params) {
        HttpURLConnection connection = null;
        try {
            URL url = new URL(params[0]);
            connection = (HttpURLConnection) url.openConnection();
            connection.setRequestMethod("GET");
            connection.setDoInput(true);
            connection.setReadTimeout(5000);
            connection.setConnectTimeout(5000);
            connection.connect();
            Bitmap bitmap = BitmapFactory.decodeStream(connection.getInputStream());
            return bitmap;
        } catch (MalformedURLException e) {
            e.printStackTrace();
        } catch (IOException e) {
            e.printStackTrace();
        }finally {
            if(connection != null){
                connection.disconnect();
            }
        }
        return null;
    }

    @Override
    protected void onPostExecute(Bitmap bitmap) {

    }

    @Override
    protected void onCancelled() {
    }
}
```

AsyncTask在具体使用过程中有一些限制：

* AsyncTask类必须在主线程中加载。这个过程在Android4.1级以上版本已经被希望完成。
* AsyncTask对象必须在主线程中创建
* execute()方法必须在主线程中调用
* 不能直接调用onPreExecute、doInBackground等方法
* 一个AsyncTask对象只能执行一次，即只能调用一次execute()方法，否则会抛异常
* 在Android1.6之前，AsyncTask是串行执行任务，Android1.6的时候AsyncTask开始采用线程池处理并行任务，但从Android3.0开始，为了避免AsyncTask带来的并发问题，AsyncTask又采用串行执行任务。但仍然可以使用executeOnExecutor()方法来并行执行任务。因为注意在串行状态下，不应该执行特别好使的任务，否则会造成任务对象阻塞。

### HandleThread

HandlerThread继承Thread类，并在run()方法中通过Looper.prepare()创建消息队列，并通过Looper.loop()开启消息循环。这样使消息循环执行在子线程中，在构建Handler时使用此HandlerThread的Looper，那么Handler的消息处理handleMessage()都会在子线程中执行。由于HandlerThread的run()方法是一个无限的循环，因为当明确不需要再使用HandlerThread的时候，可以通过它的quit()或quitSafely()方法来执行消息循环。

```java
public class MainActivity extends AppCompatActivity {

    WorkThread workThread = null;
    WorkHandler workHandler = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);

        //---

        workThread = new WorkThread("Work_Thread");
        workThread.start();
        workHandler = new WorkHandler(workThread.getLooper());
    }

    @Override
    protected void onDestroy() {
        workThread.quitSafely();
        super.onDestroy();
    }

    class WorkThread extends HandlerThread{

        public WorkThread(String name) {
            super(name);
        }
    }

    class WorkHandler extends Handler{

        public WorkHandler(Looper looper) {
            super(looper);
        }

        @Override
        public void handleMessage(Message msg) {
            // 执行在工作线程中
        }
    }
}
```

### IntentService

## 线程池的使用

## 第三方异步方案

### EventBus

### RxJava

# 参考

* [Android开发艺术探索第11章Android的线程和线程池]()
* [Android 线程的正确使用姿势](http://android.jobbole.com/82440/)
* [线程安全、数据同步之 synchronized 与 Lock](http://android.jobbole.com/83462/)
* [Android子线程真的不能更新UI么](http://www.cnblogs.com/lao-liang/p/5108745.html)
* [EventBus](http://greenrobot.org/eventbus/)
* [给 Android 开发者的 RxJava 详解](http://gank.io/post/560e15be2dca930e00da1083)
* [Android性能优化之使用线程池处理异步任务  
 ](http://blog.csdn.net/u010687392/article/details/49850803)
* [AsyncTask源码分析](https://github.com/white37/AndroidSdkSourceAnalysis/blob/master/article/AsyncTask%E5%92%8CAsyncTaskCompat%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90.md)
