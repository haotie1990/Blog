# Activity的生命周期和启动模式

## Activity的生命周期

Activity的生命周期分两种情况，一种是典型情况下的生命周期，另一种是异常情况下的生命周期。典型情况即使指在有用户参与的情况下Activity生命周期所发生的改变；而异常情况下是指Activity被系统销毁或者设备Configuration发生改变导致Activity被销毁重建的生命周期。

下图是一个完整的Activity生命周期切换流程图

![](../Image/activity_lifecycle.png)

|生命周期|状态表示|可做操作|
|:-----|:-------|:----------|
|onCreate|Activity被创建|做初始化工作，例如setContentView加载布局资源，初始化数据|
|onRestart|Activity正在重新启动|--|
|onStart|Activity正在被启动，其已经显示但为进入前台无法交互|--|
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
android:configChanges="orientation|keybordHidden|screenSize“
```

此时如果系统配置发生改变，将会回调`onConfigurationChanged(Configuration)`方法。
