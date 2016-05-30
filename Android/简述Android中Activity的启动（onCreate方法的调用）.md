#####**先写在前边，这是第一次写博客**

***
写过Java程序的人都知道，每一个Java都有一个main方法作为程序的入口，所以在Android应用程序上也存在一个这样的main方法，一个Android的应用程序都有一个MainActiviy，也许很多人认为主Activity就是一个程序的入口，但实际上真正的入口在AcivityThread类中。

本篇不会详细介绍整个Activity启动过程中的详细函数方法或者其逻辑，只是简单的梳理其过程中的相关类和关键的方法。

[ActivityThread-->main](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.main%28java.lang.String%5B%5D%29)
ActivityThread类中main方法就是一个应用程序的主入口，当启动一个应用，会先由Zygote进程孵化出新的进程后，会执行AcivityThread的main方法。在main方法中的关键是ActivityThread的attach方法将其绑定到ActivityManagerService中。

[ActivityThread-->attach](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.attach%28boolean%29)
在attach方法中只关注非system的部分，获取ActivityManagerService调用其attachApplication方法将ActivityThread添加到AMS中。

[ActivityManagerService-->attachApplication](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/com/android/server/am/ActivityManagerService.java#ActivityManagerService.attachApplication%28android.app.IApplicationThread%29)
在attachApplication方法中主要调用AMS的attachApplicationLocked方法。

[ActivityManagerService-->attachApplicationLocked](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/com/android/server/am/ActivityManagerService.java#ActivityManagerService.attachApplicationLocked%28android.app.IApplicationThread%2Cint%29)
在attachApplicationLocked方法中我们只关心其中两个重要的方法bindApplication方法和attachApplicationLocked(ProcessRecord)。其中bindApplication方法参数非常多，主要作用就是讲ApplicationThread绑定到AMS，我们不做关注。主要看mStackSupervisor.attachApplicationLocked(ProcessRecord)。

[ActivityStackSupervisor-->attachApplicationLocked](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/com/android/server/am/ActivityStackSupervisor.java#ActivityStackSupervisor.attachApplicationLocked%28com.android.server.am.ProcessRecord%29)
ActivityStackSupervisor类中的attachApplicationLocked方法中真正关心的是其realStartActivityLocked方法，进入启动Activity的逻辑中。

[ActivityStackSupervisor-->realStartActivityLocked](http://grepcode.com/file/repository.grepcode.com/java/ext/com.google.android/android/5.1.1_r1/com/android/server/am/ActivityStackSupervisor.java#ActivityStackSupervisor.realStartActivityLocked%28com.android.server.am.ActivityRecord%2Ccom.android.server.am.ProcessRecord%2Cboolean%2Cboolean%29)
realStartActivityLocked方法很长，主要都是做启动Activity的准备，当准备完成后就会调用ApplicationThread的scheduleLaunchActivity方法准备去启动Activity。

[ApplicaionThread-->schedulLaunchActivity](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.ApplicationThread.scheduleLaunchActivity%28android.content.Intent%2Candroid.os.IBinder%2Cint%2Candroid.content.pm.ActivityInfo%2Candroid.content.res.Configuration%2Candroid.content.res.CompatibilityInfo%2Ccom.android.internal.app.IVoiceInteractor%2Cint%2Candroid.os.Bundle%2Candroid.os.PersistableBundle%2Cjava.util.List%2Cjava.util.List%2Cboolean%2Cboolean%2Candroid.app.ProfilerInfo%29)
schedulLaunchActivity方法很简单对启动信息进行准备然后构造ActivityClientRecord对象，最后通过sendMessage方法发送一个LAUNCH_ACTIVITY的消息。在ActivityThread的内部有一个继承Handler的内部类H，消息将由内部类H处理。

[H-->LAUNCH_ACTIVITY](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.H.handleMessage%28android.os.Message%29)
内部类H处理消息调用handleLaunchActivity方法

[ActivityThread-->handleLaunchActivity](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.handleLaunchActivity%28android.app.ActivityThread.ActivityClientRecord%2Candroid.content.Intent%29)
handleLaunchActivity方法的逻辑比较简单，重点是调用了performLaunchActivity方法。

[ActivityThread-->performLaunchActivity](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/ActivityThread.java#ActivityThread.performLaunchActivity%28android.app.ActivityThread.ActivityClientRecord%2Candroid.content.Intent%29)
performLaunchActivity方法开始了真正处理具体的Activity启动。其中Instrumentationcal的callActivityOnCreate方法就会执行对Activity onCreate的调用。

[Instrumentation-->callActivityOnCreate](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/Instrumentation.java#Instrumentation.callActivityOnCreate%28android.app.Activity%2Candroid.os.Bundle%2Candroid.os.PersistableBundle%29)
在callActivityOnCreate方法中调用Activity的performCreate方法

[Activity-->performCreate](http://grepcode.com/file/repo1.maven.org/maven2/org.robolectric/android-all/5.0.0_r2-robolectric-1/android/app/Activity.java#Activity.performCreate%28android.os.Bundle%2Candroid.os.PersistableBundle%29)
```java
final void performCreate(Bundle icicle){
	onCreate();
	mActivityTransitionState.readState(icicle);
	performCreateCommon();
}
```
 至此所有的调用流程都已经完成

