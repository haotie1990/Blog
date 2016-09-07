# Activity的生命周期和启动模式

## Activity的生命周期

Activity的生命周期分两种情况，一种是典型情况下的生命周期，另一种是异常情况下的生命周期。典型情况即使指在有用户参与的情况下Activity生命周期所发生的改变；而异常情况下是指Activity被系统销毁或者设备Configuration发生改变导致Activity被销毁重建的生命周期。

下图是一个完整的Activity生命周期切换流程图

![](../Image/activity_lifecycle.png)

|生命周期|状态表示|可做操作|
|:-----|:-------|:----------|
|onCreate|Activity被创建|做初始化工作，例如setContentView加载布局资源，初始化数据|
|onRestart|Activity正在重新启动|--|
|onStart|Activity正在被启动，其已经显示但未进入前台无法交互|--|
|onResume|Activity可见并可以和用户交互|开启动画，初始化设备连接，例如相机等|
|onPause|Activity正在停止|保存数据，停止动画，但不能耗时否则会影响下一个Activity的显示|
|onStop|Activity即将停止切对用户不可见|做一些回收工作|
|onDestroy|Activity被销毁|做最后的资源回收工作|

几种情况下的生命周期回调：

* Activity第一次启动：onCreate->onStart->onResume
* 用户打开新的Activity或回到主页：onPause->onStop，如果新的Activity的主题是透明主题，则当前Activity不会回调onStop
* 回到原Activity：onRestart->onStart->onResume
* 用户按下Back键退出：onPause->onStop->onDestroy

> NOTE：不能在onPause中做重量级的操作，因为必须onPause执行完后新的Activity才能onResume

Activity生命周期发生变化，还有一种情况就是异常的生命周期，主要原因就是资源相关的系统配置（Configuration）发生改变导致Activity被杀死并重新创建。在这种情况下，系统会调用onSaveInstanceState来保存当前Activity的状态，我们可以在此方法中我们可以将一些信息以键值对的形式保存在Bundle对象的参数中。当Activity被重新创建后，系统会调用onRestoreInstanceState函数，并把Activity销毁是onSaveInstanceState函数所保存的Bundle对象作为参数船体给onRestoreInstanceState函数和onCreate函数，我们可以在这两个任意函数中从Bundle参数对象中取出之前保存的数据并恢复。

![](../Image/image3-sml.png)

> NOTE：Activity在系统配置改变情况下需要重建时，系统也会为我们保存当前Activity的视图结构，例如文本框的用户输入、ListView的滚动位置等。

当然也可以避免系统配置改变时Activity被烧毁而重新创建，只需要给Activity指定configChanges属性。以下是介个主要的属性：

|属性|功能|
|:-----|:------------|
|locale|设备本地位置发生了改变，一般指系统语言|
|keybordHidden|键盘的访问性发生了改变，比如用户调出了键盘|
|orientation|屏幕方向发生了改变，比如旋转了手机屏幕|
|screenSize|屏幕的尺寸信息发生了改变，当旋转屏幕时，屏幕的尺寸信息会改变，这个选项比较特殊，它和编译选项有关，当编译选项中的minSdkVersion和targetSdkVersion均小于13时，Activity不会销毁重启，否则会销毁重启|

所以一般情况下，我们采用在AndroidManifest.xml中如下配置防止系统配置发生改变导致Activity销毁重启：
```
android:configChanges="orientation|keybordHidden|screenSize"
```

此时如果系统配置发生改变，将会回调`onConfigurationChanged(Configuration)`方法。

## Activity的启动模式和返回栈

正常情况下，我们多次启动同一个Activity的时候，系统会创建多个实例并把他们放入任务栈（任务栈是指在执行特定作业时与用户交互的一系列Activity，这些Activity按照各自的打开顺序排列在堆栈即返回栈中）中。但有时候，我们希望多次启动同一个Activity时，系统只创建一个实例，这个时候就需要通过设置Activity的启动模式来实现。

|启动模式|功能|
|:-----|:------------|
|standard|系统默认启动模式，每次启动一个Activity都会重新创建一个新的实例，不管这个实例是否已经存在。在这种模式下，谁启动了这个Activity，那么这个Activity就运行在启动它的按个Activity所在的任务栈中|
|singleTop|栈顶复用模式，如果新的Activity已经位于任务栈的栈顶，那么次Activity不会被重新创建，同时它的onNewIntent方法会被回调|
|singleTask|栈内复用模式，这是一种单例模式，在这种模式下，如果Activity声明的taskAffinity任务栈存在，这个时候检查此任务栈是否有此Activity的实例，如果存在实例，就把此Activity上面的的Activity销毁并将此Activity移至栈顶，如果实例不存在，就创建Activity并压入栈。如果不存在指定的任务栈，就重新创建任务栈，然后创建Activity实例并压入栈|
|singleInstance|单例模式，这是一种加强的singleTask模式，但是具有此模式的Activity只能单独位于一个任务栈中。即其他Activity启动此模式的Activity都会单独创建任务栈，由此Activity启动的任何Activity也均在单独的任务栈中打开|

下图为“singleTask”模式的Activity的任务栈状态：

![](../Image/diagram_backstack_singletask_multiactivity.png)

以上的启动模式需要在AndroidManifest文件中配置Activity的android:launchMode属性，同时也可以在启动的Activity的时候，通过Intent的setFlags方法设置设置Flags来设定Activity的启动模式。常用的有如下Flags：

* FLAG_ACTIVITY_NEW_TASK：类似“singleTask”
* FLAG_ACTIVITY_SINGLE_TOP：类似“singleTop”
* FLAG_ACTIVITY_CLEAR_TOP：具有此标记的Activity，在它启动时，同一个任务栈中所有位于它上面的Activity都要出栈
* FLAG_ACTIVITY_EXCLUDE_FROM_RECENTS：具有此标记的Activity不会出现在历史Activity的列表中

## Activity启动模式的使用

**singleTop**

适合那些接收到通知而启动显示内容的页面，例如：某个新闻客户端的新闻内容页面，因为如果收到10新闻推送，打开十个页面既耗费资源用户在返回时也很头疼。

**singleTask**

适合作为程序的主入口，例如浏览器的主页，不管从多少个应用打开浏览器，之后会启动主页一次，其余每次都是走onNewIntent，并且会清空主页面前面的页面

**singleInstance**

独立功能页面，例如闹铃的提醒页面，我们首先进入闹钟设置界面设置闹钟，然后按HOME键回到主页，随后又进入微信聊天，此时闹钟提醒启动，我们关掉闹钟返回到微信聊天页面。因为闹钟提醒页面所在的返回栈只有一个Activity，退出后此返回栈为空则返回到微信了解页面。如果是singleTask则会返回到闹钟设置页面。

# 参考

* [任务和返回栈](https://developer.android.com/guide/components/tasks-and-back-stack.html)
* [Android开发艺术探索]()
