本文我们将讲解一下关于Android开发过程中常见的内存泄露场景与检测方案。Android系统为每个应用程序分配的内存是有限的，当一个应用中产生的内存泄漏的情况比较多时，这就会导致应用所需要的内存超过这个系统分配的内存限额，进而造成了内存溢出而导致应用崩溃。在实际的开发过程中我们由于对程序代码的不当操作随时都有可能造成内存泄露。

**（1）什么是内存泄露**

当一个对象已经不需要再使用了，本该被回收时，而有另外一个正在使用的对象持有它的引用从而导致它不能被回收，这导致本该被回收的对象不能被回收而停留在堆内存中，这就产生了内存泄漏。

**（2）系统分配的应用内存大小**

ActivityManager的getMemoryClass()获得内用正常情况下内存的大小
ActivityManager的getLargeMemoryClass()可以获得开启largeHeap最大的内存大小

```
ActivityManager activityManager = (ActivityManager)context.getSystemService(Context.ACTIVITY_SERVICE);
activityManager.getMemoryClass();
activityManager.getLargeMemoryClass();
```
需要指出的是这里获取的内存大小是JVM为进程分配的内存大小，而当我们的应用中存在多个进程的时候，该应用理论上的内存大小限制：

- 应用内存 = 进程内存大小 * 进程个数

所以当我们应用需要较大内存的时候也可以考虑通过多进程的方式进而获取更多的系统内存。

这样获取到的应用内存大小就是应用所能获取到的最大内存大小，当应用需要更多内存以支持其运行的时候，系统无法为其分配更多的内存，这样就造成了OOM的异常。


**（3）内存泄露的常见场景**


- 非静态内部类，静态实例化

```
/**
 * 自定义实现的Activity
 */
public class MyActivity extends AppCompatActivity {

	/**
	 * 静态成员变量
	 */
    public static InnerClass innerClass = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_my);

        innerClass = new InnerClass();
    }

    class InnerClass {

        public void doSomeThing() {
        }
    }
}
```
这里内部类InnerClass隐式的持有外部类MyActivity的引用，而在MyActivity的onCreate方法中调用了

```
innerClass = new InnerClass();
```
这样innerClass就会在MyActivity创建的时候是有了他的引用，而innerClass是静态类型的不会被垃圾回收，MyActivity在执行onDestory方法的时候由于被innerClass持有了引用而无法被回收，所以这样MyActivity就总是被innerClass持有而无法回收造成内存泄露。

- 不正确的使用Context对象造成内存泄露

```
/**
 * 自定义单例对象
 */
public class Single {
    private static Single instance;
    private Context context;
    private Object obj = new Object();
    
    private Single(Context context) {
        this.context = context;
    }
	
	/**
	 * 初始化获取单例对象
	 */
    public static Single getInstance(Context context) {
        if (instance == null) {
	        synchronized(obj) {
				if (instance == null) {
					instance = new Single(context);
				}
			}
        }
        return instance;
    }
}
```

我们通过懒汉模式创建单例对象，并且在创建的时候需要传入一个Context对象，而这时候如果我们使用Activity、Service等Context对象，由于单例对象的生命周期与进程的生命周期相同，会造成我们传入的Activity、Service对象无法被回收，这时候就需要我们传入Application对象，或者在方法中使用Application对象，上面的代码可以改成：

```
/**
 * 自定义单例对象
 */
public class Single {
    private static Single instance;
    private Context context;
    private Object obj = new Object();
    
    private Single(Context context) {
        this.context = context;
    }
	
	/**
	 * 初始化获取单例对象
	 */
    public static Single getInstance(Context context) {
        if (instance == null) {
	        synchronized(obj) {
				if (instance == null) {
					instance = new Single(context.getApplication());
				}
			}
        }
        return instance;
    }
}
```
这样就不会有内存泄露的问题了。


- 使用Handler异步消息通信

在日常开发中我们通常都是这样定义Handler对象：

```
/**
 * 定义Handler成员变量
 */
Handler handler = new Handler() {  
        @Override  
        public void handleMessage(Message msg) {  
            dosomething();  
  
        }  
    };  
```

但是这样也存在着一个隐藏的问题：在Activity中使用Handler创建匿名内部类会隐式的持有外部Activity对象的引用，当子线程使用Handler暂时无法完成异步任务时，handler对象无法销毁，同时由于隐式的持有activity对象的引用，造成activity对象以及相关的组件与资源文件同样无法销毁，造成内存泄露。
好吧，那么如何解决这个问题呢？具体可以参考：<a href="http://blog.csdn.net/qq_23547831/article/details/46881941">Android中使用Handler造成内存泄露的分析和解决</a>

- 使用资源文件结束之后未关闭

在使用一些资源性对象比如(Cursor，File，Stream，ContentProvider等)往往都用了一些缓冲，我们在不使用的时候，应该及时关闭它们，以便它们的缓冲及时回收内存。它们的缓冲不仅存在于Java虚拟机内，还存在于Java虚拟机外。如果我们仅仅是把它的引用设置为null,而不关闭它们，往往会造成内存泄露。

因为有些资源性对象，比如SQLiteCursor(在析构函数finalize(),如果我们没有关闭它，它自己会调close()关闭)，如果我们没有关闭它，系统在回收它时也会关闭它，但是这样的效率太低了。因此对于资源性对象在不使用的时候，应该立即调用它的close()函数，将其关闭掉，然后再置为null.在我们的程序退出时一定要确保我们的资源性对象已经关闭。

```
/**
 * 初始化Cursor对象
 */
Cursor cursor = getContentResolver().query(uri...); 
if (cursor.moveToNext()) { 
	/**
	 * 执行自设你的业务代码
	 */ 
	 doSomeThing();
}
```

这时候我们应当在doSomeThing之后执行cursor的close方法，关闭资源对象。

```
/**
 * 初始化Cursor对象
 */
Cursor cursor = getContentResolver().query(uri...); 
if (cursor.moveToNext()) { 
	/**
	 * 执行自设你的业务代码
	 */ 
	 doSomeThing();
}

if (cursor != null) {
	cursor.close();
}
```

- Bitmap使用不当

bitmap对象使用的内存较大，当我们不再使用Bitmap对象的时候一定要执行recycler方法，这里需要指出的是当我们在代码中执行recycler方法，Bitmap并不会被立即释放掉，其只是通知虚拟机该Bitmap可以被recycler了。

当然了现在项目中使用的一些图片库已经帮我们对图片资源做了很好的优化缓存工作，是我们省去了这些操作。

- 一些框架使用了注册方法而未反注册

比如我们时常使用的事件总线框架-EventBus，具体的实现原理可参考：<a href="http://blog.csdn.net/lmj623565791/article/details/40920453">Android EventBus源码解析 带你深入理解EventBus</a>当我们需要注册某个Activity时需要在onCreate中：

```
EventBus.getDefault().register(this);
```
然后这样之后就没有其他操作的话就会出现内存泄露的情况，因为EventBus对象会是有该Activity的引用，即使执行了改Activity的onDestory方法，由于被EventBus隐式的持有了该对象的引用，造成其无法被回收，这时候我们需要在onDestory方法中执行：

```
EventBus.getDefault().unregister(this);
```

- 集合中的一些方法的错误使用

（1）比如List列表静态化，只是添加元素而不再使用时不清楚元素；
（2）map对象只是put，而无remove操作等等；


**（4）关于内存泄露检测的两个开源方案 **

在项目中使用到了两个开源的内存泄露检测库：

<a href="https://github.com/square/leakcanary">LeakCanary</a>
<a href="https://github.com/fengcunhan/BlockCanary">BlockCanary</a>

推荐使用一下这两个库检测一下项目，或许会有意想不到的收获（曾检测出一个主流第三方SDK的内存泄露BUG）。

关于LeakCanary，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/50536690">Android内存泄露监测之leakcanary</a>，大概讲解了一下LeakCanary的使用方式。

BlockCanary库的使用方式和LeakCanary类似，更多关于其使用方式的介绍可查看其github文档。

除了以上两个开源库之外，还可以考虑使用软引用的方式，更多关于Java引用类型的知识，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/46505287">Java中的四种引用</a>

**（5）关于屏蔽内存泄露的建议**

- 正确的保证内存对象的生命周期，就是尽量保证内存对象在其生命周期内创建于结束，比如Android中的“上帝对象Context”，要保证不同的场景下使用不同的Context对象，下面是一张Context对象的使用场景图：
![这里写图片描述](http://img.blog.csdn.net/20160926192227820)

- 对资源对象的使用要在使用完成之后保证调用其资源的关闭方法，而非仅仅是对资源引用的关闭操作；

- 静态化资源对象其生命周期就会变成与进程的生命周期相同，在使用静态化时一定要考虑清楚该对象静态化是否存在内存泄露的可能；

- 对Android开发中常见的内存泄露场景要做到了然于胸，了解一些Android中常见的内存泄露检测方法；

**总结：**

关于内存泄露其实主要记住一个原则就好：确保对象能够在正确的时机被回收掉。然后我们根据具体内存泄露的场景具体解决就好了。

<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">Android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">Android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">Android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">Android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">Android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">Android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">Android产品研发（十九）-->Android studio中的单元测试</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080">Android产品研发（二十）-->代码Review</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868453">Android产品研发（二十一）-->Android中的UI优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868496">Android产品研发（二十二）-->Android实用调试技巧</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51953926">Android产品研发（二十三）-->Android中保存静态秘钥实践</a>
