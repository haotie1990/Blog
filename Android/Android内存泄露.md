# Android内存泄露

## Java的内存回收机制

### 1. Java虚拟机内存模型

Java虚拟机在运行Java程序时会把内存区域分成几个数据区来存储不同的数据，几个数据区如下：

![](../Image/jvm_memory_1.png)

* 程序计数器

程序计数器是一块较小的内存空间，可以看做是当前线程所执行字节码的行号指示器，其是线程私有的，同时也是唯一不存在OOM的区域。

* Java虚拟机栈

虚拟机栈是Java方法执行的内存模型：每个方法在执行的同时，都会创建一个栈帧，用于存储局部变量表、操作数栈、动态链接、方法出口灯信息。我们平常所指的栈内存和堆内存，其中的“栈”就是指代虚拟机栈，或者说是虚拟机栈中的局部变量表。局部变量表所需的内存空间在编译期完成分配，当进入一个方法时，这个方法所需要在栈帧中分配多的的局部变量空间是完全确定的，在方法运行期间不会改变局部变量表的大小。

* 本地方法栈

本地方法栈为虚拟机使用到Native方法服务

* Java堆

Java堆在虚拟机启动时创建，Java堆的唯一目的就是存放对象和数组。Java堆也是垃圾收集器管理的主要区域，因此也称为“GC堆”。Java堆物理上不需要连续内存，只需要逻辑上连续即可。如果堆中没有内存可以完成实例分配，并且也无法扩展时，将会抛出`OutOfMemoryError`异常。

* 方法区

方法区用于存储已被虚拟机加载的的类的信息、常量、静态变量、即时编译器编译后的代码等数据。方法区还有一个别名叫做Non-Heap（非堆），用于与Java对区分。对于HotSpot虚拟机来说，方法区又习惯称为“永久代”，代这只是针对HotSpot虚拟机来说的，其他虚拟机在实现上没有这个概念。相对而言，垃圾收集行为在这个区域比较少出现，但并非不会收集，这个区域的内存回收目标主要针对常量池的回收和对类型的卸载上。

> 运行时常量池也是方法区的一部分，用于存放编译器生成的各种字面量和符号引用。

### 总结

* 局部变量的基本数据类型和引用存储于栈中，引用的实体存储于堆中。
* 成员变量全部存储于堆中（包括基本数据类型，引用和引用的对象实体）


### 2. Java如何管理内存

Java的内存管理就是对象的分配和释放问题。在Java中，程序员需要通过关键字new为每个对象申请内存空间（基本类型除外），所有对象都在堆中分配空间。这块内存空间不再被任何变量引用时，这块内存就变成了垃圾，等待垃圾回收机制进行回收。

#### 2.1 对象在内存中的状态

当一个对象在堆内存中运行时，根据它被引用变量所引用的状态，可以把他所处的状态分成三种：

* 可达状态：当一个对象被创建后，若有一个以上的变量引用它，则这个对象在程序中处于可达状态，程序可以用过引用变量调用该对象的属性和方法。
* 可恢复状态：如果程序中某个对象不再有任何应用变量引用它，他就进入可恢复状态。在这种状态下，垃圾回收机制准备回收该对象所占用的内存。在回收该对象之前，系统会调用所有可恢复状态的对象的`finalize()`方法进行资源清理。如果系统在调用`finalize()`方法时重新让一个引用变量引用该对象，则这个对象会再次变为可达状态。
* 不可达状态：当对象与所有引用变量的关联都被切断，且系统已经调用所有对象的finalize()方法后依然没有使该对象变成可达状态，那么这个对象将永久性的失去引用，最后变成不可达状态。只有当一个对象处于不可达状态时，系统才会真正回收该对象所占用的资源。

#### 2.2 Java内存泄露

内存泄露是指无用对象持续占有内存或无用对象的内存得不到及时释放，从而造成内存空间的浪费。这些对象会满足两个特点，首先，这些对象是可达的；其次，这些对象是无用的。这些对象不会被GC回收，然而却占用内存。

为什么会造成Java内存泄露？长生命周期的对象持有短生命周期对象的引用就很可能发生内存泄露，尽管短生命周期对象已经不再需要，但因为长生命周期的对象持有它的引用导致不能被GC回收，就导致内存泄露。

#### 2.3 Java内存泄露的场景

1.静态集合类引起内存泄露

像`HashMap`、`Vector`等的使用最容易出现内存泄露，这些静态变量的生命周期和应用程序一致，他们所引用的所有的对象`Object`也不能被释放，因为他们也将一直被`Vector`等引用着。

```java
Static Vector v = new Vector(10);
for (int i = 1; i<100; i++)
{
    Object o = new Object();
    v.add(o);
    o = null;
}
```
在这个例子中，循环申请`Object`对象，并将所申请的对象放入一个`Vector`中，如果仅仅释放引用本身（o=null），那么`Vector`仍然引用该对象，所以这个对象对GC来说是不可回收的。因此，如果对象加入到`Vector`后，还必须从`Vector` 中删除，最简单的方法就是将`Vector`对象设置为null。

2.当集合里面的对象属性被修改引起内存泄露

```java
public static void main(String[] args)
{
    Set<Person> set = new HashSet<Person>();
    Person p1 = new Person("唐僧","pwd1",25);
    Person p2 = new Person("孙悟空","pwd2",26);
    Person p3 = new Person("猪八戒","pwd3",27);
    set.add(p1);
    set.add(p2);
    set.add(p3);

    System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:3 个元素!
    p3.setAge(2); //修改p3的年龄,此时p3元素对应的hashcode值发生改变

    set.remove(p3); //此时remove不掉，造成内存泄漏

    set.add(p3); //重新添加，居然添加成功
    System.out.println("总共有:"+set.size()+" 个元素!"); //结果：总共有:4 个元素!
    for (Person person : set)
    {
        System.out.println(person);
    }
}
```

3.监听器

在java 编程中，我们都需要和监听器打交道，通常一个应用当中会用到很多监听器，我们会调用一个控件的诸如`addXXXListener()`等方法来增加监听器，但往往在释放对象的时候却没有记住去删除这些监听器，从而增加了内存泄漏的机会。

4.各种连接

比如数据库连接，网络连接和IO连接，除非其显式的调用了其close()或disconnect()方法将其连接关闭，否则是不会自动被GC回收的。

5.非静态内部类和外部模块的引用

非静态内部类和匿名内部类都会隐式的持有其外部类的引用，如果内部类生命周期长过其外部类就会造成内存泄露。对于外部模块，如果调用了其方法，例如：`public void registerMsg(Object b)`这种调用传入了一个对象，很可能外部模块就保持了该对象的引用，这时候需要注意外部模块是否提供了相应的去除操作。

6.单例模式

不正确使用单例模式是引起内存泄漏的一个常见问题，单例对象在初始化后将在JVM的整个生命周期中存在（以静态变量的方式），如果单例对象持有外部的引用，那么这个对象将不能被JVM正常回收，导致内存泄漏。

## Android中内存泄露的范例

1.Handler

```java
public class SampleActivity extends Activity {

  private final Handler mLeakyHandler = new Handler() {
    @Override
    public void handleMessage(Message msg) {
      // ...
    }
  }

  @Override
  protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);

    // Post a message and delay its execution for 10 minutes.
    mLeakyHandler.postDelayed(new Runnable() {
      @Override
      public void run() { /* ... */ }
    }, 1000 * 60 * 10);

    // Go back to the previous Activity.
    finish();
  }
}
```

当一个Handler在主线程进行初始化之后，我们发送一个target为这个Handler的消息，实际上已经发送的消息包含一个Handler实例的引用。 由于在Java中，非静态内部类或匿名的内部类会持有其外部类的引用，因此其Handler对象会持有Activity的引用。分析一下上面的代码，当我们执行了Activity的finish方法，被延迟的消息会在被处理之前存在于主线程消息队列中10分钟，而这个消息中又包含了Handler的引用，而Handler是一个匿名内部类的实例，其持有外面的SampleActivity的引用，所以这导致了SampleActivity无法回收，进行导致SampleActivity持有的很多资源都无法回收，这就是我们常说的内存泄露。

2.单例模式使用Activity的Context

由于单例的静态特性使得其生命周期跟应用的生命周期一样长，如下面的单例，在创建时需要传入一个Context。如果此时传入的还Application的Context，因为Application的生命周期就是整个应用的生命周期，所以没有问题。如果传入的是Activity的Context，当这个Context对应的Activity退出时，由于该Context的引用被单例对象所持有，其整个生命周期等于整个应用的生命周期，所以当Activity退出时它的内存并不会被回收，这就造成了内存泄露。

```java
public class AppManager {
private static AppManager instance;
private Context context;

private AppManager(Context context) {
    this.context = context;
}

public static AppManager getInstance(Context context) {
    if (instance == null) {
        instance = new AppManager(context);
    }
    return instance;
}

}
```

3.匿名异步线程

上文提到过，匿名的内部类也会持有其外部的引用，如果此时这个匿名类还是一个异步线程，那么如果在线程中做耗时操作，当Activity退出时，其操作还未完成。就会导致Activity占用的内存无法释放，造成内存泄露。

# 参考

* [Java内存区域模型、对象创建过程、常见OOM
 ](http://blog.csdn.net/shakespeare001/article/details/51732155)
* [Android 内存泄漏总结](https://github.com/GeniusVJR/LearningNotes/blob/master/Part1/Android/Android%E5%86%85%E5%AD%98%E6%B3%84%E6%BC%8F%E6%80%BB%E7%BB%93.md)
* [Jvm内存模型](http://gityuan.com/2016/01/09/java-memory/)
