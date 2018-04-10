# Android-Advanced-Notes
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

#### bitmap的内存计算公式（如何压缩）
`https://www.cnblogs.com/zj2012zy/p/5331302.html`
当图片以格式ARGB_8888存储时的计算方式
占用内存=图片长*图片宽*4字节
图片长 = 图片原始长 (设备DPI/文件夹DPI) 
图片宽 = 图片原始宽(设备DPI/文件夹DPI)
举例验证如下：
图片大小 200 * 320，设备为红米dpi为320，属于xhdpi设备。

验证一 图片放在hdpi，下面为代码输出结果：
DD/MainActivity(13014): dpi: 320    bitmap ByteCount: 456036
图片长 = （320 / 240） * 200  = 266.67
图片宽 = （320 / 240 ）* 320 = 426.67
占用内存 = 266.67 * 426.67 * 4 = 455116 与 实际值大致相同

验证二：图片放xxhdpi下，下面为代码输出结果：
D/MainActivity(13014): dpi: 320    bitmap ByteCount: 113316
图片长 = （320 / 480 ） * 200 = 133.33
图片宽 = （320 / 480 ） * 320 = 213.33
占用内存 = 133.33 * 213.33 * 4 = 113774 与 实际值大致相同。
`https://blog.csdn.net/u010652002/article/details/72676723`
<table border="1">
<tr>
<td>density</td>
 <td>1</td>
 <td>1.5</td>
 <td>2</td>
 <td>3</td>
 <td>3.5</td>
 <td>4</td>
</tr>
<tr>
<td>densityDpi</td>
 <td>160</td>
 <td>240</td>
 <td>320</td>
 <td>480</td>
 <td>560</td>
 <td>640</td>
</tr>
</table>
如何压缩：
* 使用 inSampleSize。
这个方法主要用在图片资源本身较大，或者适当地采样并不会影响视觉效果的条件下，这时候我们输出地目标可能相对较小，对图片分辨率、大小要求不是非常的严格
BitmapFactory.Options options = new Options();
options.inSampleSize = 2;
Bitmap bitmap = BitmapFactory.decodeResource(getResources(), resId, options);
* 使用矩阵
Matrix matrix = new Matrix();
matrix.preScale(2, 2, 0f, 0f);
//如果使用直接替换矩阵的话，在Nexus6 5.1.1上必须关闭硬件加速
canvas.concat(matrix);
canvas.drawBitmap(bitmap, 0,0, paint);
大图小用用采样，小图大用用矩阵
Matrix matrix = new Matrix();
matrix.postScale(2, 2, 0, 0);
imageView.setImageMatrix(matrix);
imageView.setScaleType(ScaleType.MATRIX);
imageView.setImageBitmap(bitmap);
* 合理选择Bitmap的像素格式
ALPHA8 没必要用，因为我们随便用个颜色就可以搞定的。
ARGB4444 虽然占用内存只有 ARGB8888 的一半，不过已经被官方嫌弃，失宠了。。『又要占省内存，又要看着爽，臣妾做不到啊T T』。
ARGB8888 是最常用的，大家应该最熟悉了。
RGB565 看到这个，我就看到了资源优化配置无处不在，这个绿色。。（不行了，突然好邪恶XD），其实如果不需要 alpha 通道，特别是资源本身为 jpg 格式的情况下，用这个格式比较理想。
* 高能：索引位图(Indexed Bitmap)
* 尽可能替换图片，使用自定义view来替换。

#### Handler机制
`https://www.jianshu.com/p/7f2fcb43f8d9`
Android中的消息机制主要就是指Handler的消息机制，Handler相信大家已经非常熟悉了，它可以将一个任务切换到Handler所在的线程中去执行，开发中，当我们在子线程做了一些操作后需要更新UI，由于Android不允许在子线程中访问UI控件，所以我们一般都会使用handler来实现。

Handler的机制需要MessageQueue、Looper和Message的支持。他们在消息机制中各扮演了不同的角色
Handler：负责消息的发送和接收处理
MessageQueue：消息队列，一个消息存储单位，经常需要进行增减，内部使用的是单链表的结构
Looper：消息循环。会不停地从MessageQueue中取消息，如果有新消息就会立刻处理，否则就一直阻塞在那里
Message：消息载体

#### 图片加载框架对比
`https://blog.csdn.net/u013134722/article/details/56676078`

在正式对比前，先了解几个图片缓存通用的概念：

RequestManager：请求生成和管理模块；
Engine：引擎部分，负责创建任务（获取数据），并调度执行；
GetDataInterface：数据获取接口，负责从各个数据源获取数据。比如 MemoryCache 从内存缓存获取数据、DiskCache 从本地缓存获取数据，下载器从网络获取数据等。
Displayer：资源（图片）显示器，用于显示或操作资源。 比如 ImageView，这几个图片缓存都不仅仅支持 ImageView，同时支持其他 View 以及虚拟的 Displayer 概念。
Processor 资源（图片）处理器， 负责处理资源，比如旋转、压缩、截取等。
以上概念的称呼在不同图片缓存中可能不同，比如 Displayer 在 ImageLoader 中叫做 ImageAware，在 Picasso 和 Glide 中叫做 Target。

* 共同优点
1. 使用简单 
都可以通过一句代码可实现图片获取和显示。
2. 可配置度高，自适应程度高 
图片缓存的下载器（重试机制）、解码器、显示器、处理器、内存缓存、本地缓存、线程池、缓存算法等大都可轻松配置。
自适应程度高，根据系统性能初始化缓存配置、系统信息变更后动态调整策略。 
比如根据 CPU 核数确定最大并发数，根据可用内存确定内存缓存大小，网络状态变化时调整最大并发数等。
3. 多级缓存 
都至少有两级缓存、提高图片加载速度。
4. 支持多种数据源 
支持多种数据源，网络、本地、资源、Assets 等
5. 支持多种 Displayer 
不仅仅支持 ImageView，同时支持其他 View 以及虚拟的 Displayer 概念。
6. 其他小的共同点包括支持动画、支持 transform 处理、获取 EXIF 信息等。

* Fresco 优点
1. 图片存储在安卓系统的匿名共享内存, 而不是虚拟机的堆内存中, 图片的中间缓冲数据也存放在本地堆内存, 所以, 应用程序有更多的内存使用,不会因为图片加载而导致oom, 同时也减少垃圾回收器频繁调用回收Bitmap导致的界面卡顿,性能更高.
2. 渐进式加载JPEG图片, 支持图片从模糊到清晰加载
3. 图片可以以任意的中心点显示在ImageView, 而不仅仅是图片的中心.
4. JPEG图片改变大小也是在native进行的, 不是在虚拟机的堆内存, 同样减少OOM
5. 很好的支持GIF图片的显示
缺点:
1. 框架较大, 影响Apk体积
2. 使用较繁琐

* ImageLoader 优点
(1) 支持下载进度监听
(2) 可以在 View 滚动中暂停图片加载 
通过 PauseOnScrollListener 接口可以在 View 滚动中暂停图片加载。
(3) 默认实现多种内存缓存算法这几个图片缓存都可以配置缓存算法，不过 ImageLoader 默认实现了较多缓存算法，如 Size 最大先删除、使用最少先删除、最近最少使用、先进先删除、时间最长先删除等。
(4) 支持本地缓存文件名规则定义
缺点:
 缺点在于不支持GIF图片加载,  缓存机制没有和http的缓存很好的结合, 完全是自己的一套缓存机制

* Picasso 优点
(1) 自带统计监控功能 
支持图片缓存使用的监控，包括缓存命中率、已使用内存大小、节省的流量等。
(2) 支持优先级处理 
每次任务调度前会选择优先级高的任务，比如 App 页面中 Banner 的优先级高于 Icon 时就很适用。
(3) 支持延迟到图片尺寸计算完成加载
(4) 支持飞行模式、并发线程数根据网络类型而变 
手机切换到飞行模式或网络类型变换时会自动调整线程池最大并发数，比如 wifi 最大并发为 4， 4g 为 3，3g 为 2。 
这里 Picasso 根据网络类型来决定最大并发数，而不是 CPU 核数。
(5) “无”本地缓存 
无”本地缓存，不是说没有本地缓存，而是 Picasso 自己没有实现，交给了 Square 的另外一个网络库 okhttp 去实现，这样的好处是可以通过请求 Response Header 中的 Cache-Control 及 Expired 控制图片的过期时间。
缺点
 于不支持GIF, 并且它可能是想让服务器去处理图片的缩放, 它缓存的图片是未缩放的, 并且默认使用ARGB_8888格式缓存图片, 缓存体积大.

* Glide 优点
(1) 图片缓存->媒体缓存 
Glide 不仅是一个图片缓存，它支持 Gif、WebP、缩略图。甚至是 Video，所以更该当做一个媒体缓存。
(2) 支持优先级处理
(3) 与 Activity/Fragment 生命周期一致,LifecycleListener来监听生命周期变化，支持 trimMemory 
Glide 对每个 context 都保持一个 RequestManager，通过 FragmentTransaction 保持与 Activity/Fragment 生命周期一致，并且有对应的 trimMemory 接口实现可供调用。
(4) 支持 okhttp、Volley 
Glide 默认通过 UrlConnection 获取数据，可以配合 okhttp 或是 Volley 使用。实际 ImageLoader、Picasso 也都支持 okhttp、Volley。
(5) 内存友好 
① Glide 的内存缓存有个 active 的设计 
从内存缓存中取数据时，不像一般的实现用 get，而是用 remove，再将这个缓存数据放到一个 value 为软引用的 activeResources map 中，并计数引用数，在图片加载完成后进行判断，如果引用计数为空则回收掉。
② 内存缓存更小图片 
Glide 以 url、viewwidth、viewheight、屏幕的分辨率等做为联合 key，将处理后的图片缓存在内存缓存中，而不是原始图片以节省大小
③ 与 Activity/Fragment 生命周期一致，支持 trimMemory
④ 图片默认使用默认 RGB565 而不是 ARGB888 
虽然清晰度差些，但图片更小，也可配置到 ARGB_888。
其他：Glide 可以通过 signature 或不使用本地缓存支持 url 过期

#### Eventbus的实现原理
`https://blog.csdn.net/lmj623565791/article/details/40920453`
主要用于事件的发布和订阅 类似观察者模式
register的时候会遍历当前类所有方法，找到onEvent开头的方法， 存在map里
在post event的时候 去之前的map里找到对应方法，反射调用。

#### Activity、Window、View三者的差别，fragment的特点？
`https://blog.csdn.net/qq_34378183/article/details/52785669`
Activity像一个工匠（控制单元），Window像窗户（承载模型），View像窗花（显示视图） LayoutInflater像剪刀，Xml配置像窗花图纸。
在Activity中调用attach，创建了一个Window
创建的window是其子类PhoneWindow，在attach中创建PhoneWindow
在Activity中调用setContentView(R.layout.xxx)
其中实际上是调用的getWindow().setContentView()
调用PhoneWindow中的setContentView方法
创建ParentView： 作为ViewGroup的子类，实际是创建的DecorView(作为FramLayout的子类）
将指定的R.layout.xxx进行填充 通过布局填充器进行填充【其中的parent指的就是DecorView】
调用到ViewGroup
调用ViewGroup的removeAllView()，先将所有的view移除掉
添加新的view：addView()

fragment 特点
Fragment可以作为Activity界面的一部分组成出现；
可以在一个Activity中同时出现多个Fragment，并且一个Fragment也可以在多个Activity中使用；
在Activity运行过程中，可以添加、移除或者替换Fragment；
Fragment可以响应自己的输入事件，并且有自己的生命周期，它们的生命周期会受宿主Activity的生命周期影响。

#### 描述一次网络请求的过程
从我们在浏览器的地址栏输入http://blog.csdn.net/seu_calvin后回车，到我们看到该博客的主页，这中间经历了什么呢？简单地回答这个问题，大概是经历了域名解析、TCP的三次握手、建立TCP连接后发起HTTP请求、服务器响应HTTP请求、浏览器解析html代码，同时请求html代码中的资源（如js、css、图片等）、最后浏览器对页面进行渲染并呈现给用户。下面分别介绍一下每个过程。

#### View事件分发与传递
dispatchTouchEvent() 
onInterceptTouchEvent()
onTouchEvent()

可以用如下伪代码理解这三个方法

    public boolean dispatchTouchEvent(MotionEvent ev) {
        boolean result = false;
        if (onInterceptTouchEvent(ev)) {
            result = onTouchEvent(ev);
        } else {
            result = child.dispatchTouchEvent(ev);
        }
        return result;
    }
    
#### 事件分发中onTouch和onTouchEvent的区别
`https://blog.csdn.net/dq1005/article/details/51585980`
1、onTouch方法是View的 OnTouchListener接口中定义的方法。
2、OnTouch()先执行，只有当OnTouch()返回为false时才触发OnTouchEvent()，反之onTouchEvent方法不会被调用。
3、点击事件是在OnTouchEvent()中执行的。因此，Touch事件先执行，click事件后执行。
4、内置诸如click事件的实现等等都基于onTouchEvent事件，假如onTouch返回true，这些事件将不会被触发。

#### 怎么可以让程序在后台不被杀死
`https://blog.csdn.net/xiaole0313/article/details/76342608`

保证service不被杀掉
* 创建两个service互相监听service状态，一个被杀后，另一个重启该被杀service
* onStartCommand方法，返回START_STICKY
* 提升service优先级
* 提升service进程优先级
* onDestroy方法里重启service
* Application加上Persistent属性
* 监听系统广播判断Service状态
* 将APK安装到/system/app，变身系统级应用
* 发送一个空的notification到通知栏，能提高应用的存活几率
* 接入小米推送，华为推送等
#### 动画框架的实现原理
* 帧动画
* 补间动画，平移，旋转，淡入淡出，缩放
* 属性动画


## 性能优化
Android 性能优化 (一)APK高效瘦身 
http://blog.csdn.net/whb20081815/article/details/70140063
Android 性能优化 (二)数据库优化 秒变大神
http://blog.csdn.net/whb20081815/article/details/70142033
Android 性能优化（三）布局优化 秒变大神 
http://blog.csdn.net/whb20081815/article/details/70147958
Android 性能优化（四）内存优化OOM 秒变大神
http://blog.csdn.net/whb20081815/article/details/70243105
Android 性能优化（五）ANR 秒变大神
http://blog.csdn.net/whb20081815/article/details/70245594
Android 性能优化（六） RelativeLayout和LinearLayout性能比较
http://blog.csdn.net/whb20081815/article/details/74465870
Android 性能优化<七>自定义view绘制优化 
http://blog.csdn.net/whb20081815/article/details/74474736

#### ContentProvider 
https://blog.csdn.net/carson_ho/article/details/76101093

#### MVC、MVP、MVVM

