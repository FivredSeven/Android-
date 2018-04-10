# Android-
日常学习进阶知识积累
tracywu

#### Android中ANR的情况，怎么出现，怎么解决
`https://blog.csdn.net/qq_14906597/article/details/51800486`
KeyDispatchTimeout(5 seconds) –主要类型按键或触摸事件在5秒内无响应
BroadcastTimeout(10 seconds) –BroadcastReceiver在10秒内无法处理完成
ServiceTimeout(20 seconds) –Service在20秒内无法处理完成

仔细检查主线程的耗时操作，移动到子线程去处理。

检查trace.txt文件 调用栈  iowait?block?memoryleak?

#### Android中OOM，怎么出现，怎么解决。
`https://blog.csdn.net/hudfang/article/details/51781997`
1.Context在页面销毁的时候没有及时回收，导致泄露。
2.static修饰的静态变量引用了资源耗费比较多的实例
  解决方案：第一、将线程的内部类，改为静态内部类。并且注意第二条。
           第二、在线程内部采用弱引用保存Context引用。

3.数据库的cursor使用完没有调用close()。
4.调用registerReceiver()，页面销毁后未及时调用unregisterReceiver().
5.io操作未关闭InputStream/OutputStream。
6.bitmap泄露
  解决方案：第一、及时销毁
          第二、设置一定的采样率
          第三、软引用

#### 内存泄露和内存溢出
`https://blog.csdn.net/u013495603/article/details/50696170`
内存溢出 out of memory
内存泄露 memory leak
memory leak持续积累最终会导致out of memory！

泄露场景：
* 单例造成的内存泄露   单例长期存在，如果持有了生命周期较短的context，页面销毁时容易造成泄露。
* 匿名内部类/非静态内部类和异步线程：
* Handler 造成的内存泄漏
* static 声明的变量容易造成泄露
* 资源未关闭造成的内存泄漏

总结

对 Activity 等组件的引用应该控制在 Activity 的生命周期之内； 如果不能就考虑使用 
getApplicationContext 或者 getApplication，以避免 Activity 
被外部长生命周期的对象引用而泄露。
尽量不要在静态变量或者静态内部类中使用非静态外部成员变量（包括context 
)，即使要使用，也要考虑适时把外部成员变量置空；也可以在内部类中使用弱引用来引用外部类的变量。
对于生命周期比Activity长的内部类对象，并且内部类中使用了外部类的成员变量，可以这样做避免内存泄漏： 
将内部类改为静态内部类

静态内部类中使用弱引用来引用外部类的成员变量

Handler 的持有的引用对象最好使用弱引用，资源释放时也可以清空 Handler 里面的消息。比如在 Activity onStop 
或者 onDestroy 的时候，取消掉该 Handler 对象的 Message和 Runnable.

在 Java 的实现过程中，也要考虑其对象释放，最好的方法是在不使用某对象时，显式地将此对象赋值为 null，比如使用完Bitmap 
后先调用 recycle()，再赋为null,清空对图片等资源有直接引用或者间接引用的数组（使用 array.clear() ; 
array = null）等，最好遵循谁创建谁释放的原则。

正确关闭资源，对于使用了BraodcastReceiver，ContentObserver，File，游标 
Cursor，Stream，Bitmap等资源的使用，应该在Activity销毁时及时关闭或者注销。

保持对对象生命周期的敏感，特别注意单例、静态对象、全局性集合等的生命周期。

#### android 启动一个应用程序可以从桌面入口进去，也可以从别的应用跳转进去，有什么区别
没有本质区别，从桌面进入比较单一，只能进入到manifest里面注册的启动activity，从其他应用跳转可以传递各种数据，可以打开任意页面。

#### aidl是什么，简要说明一下
`https://blog.csdn.net/fwt336/article/details/52587133`
`https://blog.csdn.net/u011974987/article/details/51243539`
跨进程通信协议，最常见的就是通过service来实现
从AIDL的功能来看，它主要的应用场景就是IPC。虽然同一个进程中的client-service也能够通过AIDL定义接口来进行通信，但这并没有发挥AIDL的主要功能。 概括来说：
如果不需要IPC，那就直接实现通过继承Binder类来实现客户端和服务端之间的通信。
如果确实需要IPC，但是无需处理多线程，那么就应该通过Messenger来实现。Messenger保证了消息是串行处理的，其内部其实也是通过AIDL来实现。
在有IPC需求，同时服务端需要并发处理多个请求的时候，使用AIDL才是必要的

#### 自定义View的流程（View的绘制流程）
测量：onMeasure()决定View的大小；
布局：onLayout()决定View在ViewGroup中的位置；
绘制：onDraw()决定绘制这个View。

自定义View的步骤：
1. 自定义View的属性；
2. 在View的构造方法中获得自定义的属性；
3. 重写onMeasure()； --> 并不是必须的，大部分的时候还需要覆写
4. 重写onDraw()；

`https://blog.csdn.net/asdf717/article/details/52585469`
![](http://upload-images.jianshu.io/upload_images/764699-4f64498fa1b87f29.jpg?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

measure过程的核心方法: measure() - onMeasure() - setMeasuredDimension().  

如何对自定义View进行控制
1. 如果想控制View在屏幕上的渲染效果，就在重写onDraw()方法，在里面进行相应的处理。
2. 如果想要控制用户同View之间的交互操作，则在onTouchEvent()方法中对手势进行控制处理。
3. 如果想要控制View中内容在屏幕上显示的尺寸大小，就重写onMeasure()方法中进行处理。
4. 在 XML文件中设置自定义View的XML属性。
5. 如果想避免失去View的相关状态参数的话，就在onSaveInstanceState() 和 onRestoreInstanceState()方法中保存有关View的状态信息。





