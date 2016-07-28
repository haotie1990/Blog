# Touch事件分发流程及处理

所谓的事件分发，其实就是对MotionEvent事件产生以后，系统需要把这个事件传递给一个具体的View，而这个传递过程就是分发过程。在这个分发过程中主要由三个重要的方法完成：`dispatchTouchEvent`、`onInterceptTouchEvent`、`onTouchEvent`。

* `public boolean dispatchTouchEvent(MotionEvent ev)`

用来进行事件分发。如果事件能都传递给当前View，那么此方法一定会被调动，返回结果受当前View的`onTouchEvent`方法和下级View的`dispatchTouchEvent`方法的影响。

* `public boolean onInterceptTouchEvent(MotionEvent event)`

在方法dispatchTouchEvent中被调用，用来判断是否拦截某个事件，如果当前View拦截了某个事件，那么在同一个事件序列当中，此方法不会再被调用。返回结果表示是否拦截某个事件。

* `public boolean onTouchEvent(MotionEvent event)`

在dispatchTouchEvent中被调用，用来处理Touch事件，返回结果表示是消耗当前事件，如果不消耗，则在同一个事件序列中，当前view无法再次接收到事件。

# 参考

* [Android开发艺术探索](Touch事件分发流程.md)
* [Managing Touch Events in a ViewGroup](https://developer.android.com/training/gestures/viewgroup.html)
* [图解 Android 事件分发机制](http://www.jianshu.com/p/e99b5e8bd67b)
* [可能是讲解Android事件分发最好的文章](http://www.jianshu.com/p/2be492c1df96)
