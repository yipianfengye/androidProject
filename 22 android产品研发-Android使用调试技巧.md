上一篇文章中我们讲解了android UI优化方面的知识。我们讲解了android中的include、marge、ViewStub标签，在使用这些标签时可以简化我们的布局文件，优化组件绘制流程；讲解了android中的过度绘制相关知识点，通过优化我们的App过度绘制可以提高App的UI绘制流程与性能；我们还讲解了App中一些UI优化的小tips。更多关于android UI优化方面的知识可以参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51868453">android产品研发（二十一）-->android中的UI优化</a>

本文我们将讲解android中的调试技巧。程序调试，是将编制的程序投入实际运行前，用手工或编译程序等方法进行测试，修正语法错误和逻辑错误的过程。这是保证计算机信息系统正确性的必不可少的步骤。在android开发过程中熟练的使用调试技巧是一个很重要的方面。android的调试技巧包括熟练使用android中的日志API，自定义android日志框架，通过gradle配置调试日志，android studio的调试技巧等等。通过对本文的学习我们能够对android中调试技巧有一个大概的了解。

**android中的打印API：**

我们首先来看一下android中提供的打印日志API：

```
android.util.Log.java
```
这个类比较常用的打印日志的方法有5个,这5个方法都会把日志打印到AndroidMonitor中，android中日志分为五个级别：

- Log.v(tag,message);        //verbose模式,打印最详细的日志 

- Log.d(tag,message);        //debug级别的日志 

- Log.i(tag,message);        //info级别的日志 

- Log.w(tag,message);        //warn级别的日志 

- Log.e(tag,message);        //error级别的日志 

其中tag和message分别是两个String值.从android开发帮助文档中来看,tag和message的定义分别是: 

> tag：Used to identify the source of a log message. It usually identifies the class or activity where the log call occurs. 

> msg：The message you would like logged. 

可看出tag用来标记log消息的源头的。而message则是这条log的内容. 

- 一个简单的Log API例子

Log demo代码：

```
/**
 * 按钮点击事件，打印各个级别的日志
 */
button6.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                Log.v(TAG, "this is a verbose log!!!");
                Log.d(TAG, "this is a debug log!!!");
                Log.i(TAG, "this is a info log!!!");
                Log.w(TAG, "this is a warning log!!!");
                Log.e(TAG, "this is a error log!!!");
            }
        });
```
好吧，我们点击按钮，看一下AndroidMonitor中的打印结果：

```
07-17 19:22:16.202 7530-7530/uuch.com.android_activityanim V/MainActivity: this is a verbose log!!!
07-17 19:22:16.202 7530-7530/uuch.com.android_activityanim D/MainActivity: this is a debug log!!!
07-17 19:22:16.202 7530-7530/uuch.com.android_activityanim I/MainActivity: this is a info log!!!
07-17 19:22:16.202 7530-7530/uuch.com.android_activityanim W/MainActivity: this is a warning log!!!
07-17 19:22:16.203 7530-7530/uuch.com.android_activityanim E/MainActivity: this is a error log!!!
```

通过这种android自身Log API打印日志的方式，是最常见的一种打印日志的，调试代码的方式，基本上所有的项目中你可能都会遇到这种日志代码，前面的一篇文章中我也分析了Log日志的部分源码，具体可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/50963006">android源码解析之（六）-->Log</a>

但是呢你会发现这时候打印的日志格式比较简陋，比如出现异常的时候我们想快速定位到代码在哪，这时候就比艰难了，所以下面我们讲一下自定义日志框架。

**自定义android打印框架MLog：**

前面我们发现使用Android原生的Log API打印的日志格式比较简陋，那么可不可以定制化的显示一些友好型的日志信息呢？答案是肯定的，在前面的文章中我介绍了一个自定义的日志框架MLog，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51707796">github项目解析（五）-->android日志框架</a>

关于MLog框架的使用方式，实现过程等已在文章中做过简单的介绍，这里就看一下其打印的日志格式：

**自定义按钮点击事件：**

- 在Application中初始化MLog框架

```
/**
 * 传入为true，则表示执行打印操作，否则不显示日志
 */
MLog.init(true);
```
尽量可以在Application的onCreate方法中执行，这是App的进程起始方法。

- 在源代码中执行打印日志的操作

```
/**
 * 自定义按钮的点击事件
 */
button6.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View v) {
                MLog.v(TAG, "this is a verbose log!!!");
                MLog.d(TAG, "this is a debug log!!!");
                MLog.i(TAG, "this is a info log!!!");
                MLog.w(TAG, "this is a warning log!!!");
                MLog.e(TAG, "this is a error log!!!");
            }
        });
```
在AndroidMoitor中打印的日志格式和内容：
![image](http://img.blog.csdn.net/20160717194025444)

这样我们在点击日志信息的时候就能够跳转到这条日志的代码打印位置，是不是很方便？而且如果觉得日志格式不是很好的话，还可以定制化展示奥。

比如：
![image](http://img.blog.csdn.net/20160717194345590)

怎么样？是不是很好看了？

具体如何实现自定义Log框架以及自定义实现Log的显示样式可以参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51707796">github项目解析（五）-->android日志框架</a>

大多时候我们的App都有测试环境和正式环境两种环境的，当切换环境的时候要求我们在测试环境打印日志，则正式环境屏蔽日志，通过代码也可以实现，但是比较麻烦，有没有一个比较简单的方法呢？答案是肯定的，可以通过配置gradle的方式配置在测试环境中打印日志，在正式环境中屏蔽日志打印操作。


**gradle中配置正式测试打印框架：**

我们知道在android开发过程中为了调试代码经常在代码中添加一些日志信息，但是正式环境是不需要这些日志信息的，而且过得日志打印操作也会对App的性能有影响。

一个比较好的办法就是在App的测试环境中打印日志信息，在正式环境中屏蔽日志信息，那么如何实现呢？通过代码么？通过代码也是可以实现的，但是这样显得太原始了，其实android studio的gradle插件已经提供了这样的功能。

那么如何通过gradle配置日志打印信息呢？

```
buildTypes {
        debug {
            // 显示Log
            buildConfigField "boolean", "LOG_DEBUG", "true"
            //混淆
            minifyEnabled false
            //Zipalign优化
            zipAlignEnabled true
            // 移除无用的resource文件
            shrinkResources true
            //加载默认混淆配置文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //签名
            signingConfig signingConfigs.debug
        }
        release {
            // 不显示Log
            buildConfigField "boolean", "LOG_DEBUG", "false"
            //混淆
            minifyEnabled true
            //Zipalign优化
            zipAlignEnabled true
            // 移除无用的resource文件
            shrinkResources true
            //加载默认混淆配置文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //签名
            signingConfig signingConfigs.relealse
        }
    }
```
在android studio的module的gradle配置文件中，在buildTypes节点下可以配置自定义参数，这里我们在debug版本中定义LOG_DEBUG为true，在release版本中定义LOG_DEBUG为false。这样在编译的时候就会在gradle的编译类BuildConfig中生成成员变量:

```
LOG_DEBUG
```

- 若是正式环境则LOG_DEBUG的值为false

- 若是测试环境则LOG_DEBUG的值为true

所以这时候可以通过LOG_DEBUG变量的值控制日志是否打印。
```
/**
 * 在Application的onCreate方法中初始化MLog日志框架
 * 并根据apk环境判断是否显示日志信息
 */
if (BuildConfig.LOG_DEBUG == true) {
	MLog.init(true);
} else {
	MLog.init(false);
}
```
我们实现的MLog框架的init方法

- 若传入的值为true，则表示执行日志打印操作，也就是可以显示日志信息。

- 若传入的值为false，则表示不执行日志打印操作，也就是不显示日志信息。

这样我们就实现了在测试环境打印日志，在正式环境中屏蔽日志的操作。


**android studio的调试技巧：**

写代码的过程中不可避免有Bug，通常情况下除了日志最直接的调试手段就是debug；那么你的调试技术停留在哪一阶段呢？仅仅是下个断点单步执行吗？你是否知道求值调试，条件断点，日志断点，方法断点，异常断点等调试技巧么？下面我们将介绍一下android studio中的调试技巧。

**调试基础**

一般来说我们有两种办法调试一个debuggable的apk；

- 设置好断点，然后用debug模式编译安装这个app；

- 执行attach debugger to android process，即将debug进程添加到当前进程中。

一般比较常使用的debug方式是attach debugger to android process:

![image](http://img.blog.csdn.net/20160718145459117)

在执行按钮右侧两个的带有小虫子的按钮其实就是Attach Debugger to android process按钮。

我们可以在启动apk之后，直接下断点，然后attach process到指定进程，条件触发之后就可以直接进入调试模式。

**断点调试**

断点调试是最基本操作，基恩的断点调试有以下的几个常用的调试命令：

- F5跳转内部执行

- F6跳转下一步

- F8跳转到下一个断点

**如何添加断点？**

直接在android studio中的java源代码左侧，想调试某一行代码的话，直接单击若左侧出现了一个小红圈，则说明断点已经添加好了：
![image](http://img.blog.csdn.net/20160718150241495)

当代码处于debug模式，并且执行到此处的时候就会卡住在这里。

**执行断点调试**

执行attach debugger to android process，当代码执行到含有断点的时候就可以将程序卡到断点的代码了：
![image](http://img.blog.csdn.net/20160718150451731)

这时候我们可以看到在该行代码中可以看到相应的局部变量值，执行F6走到下一步，执行F8，调试到下一个断点。

**添加观察变量**
在调试模式下，选择变量，并右击：
![image](http://img.blog.csdn.net/20160718150735060)

这时候选择Add to watches，就可以将变量添加到观察列表了。

**表达式调试：Evaluate Expression**

这个调试功能非常的实用，也是我最喜欢的功能，使用ctrl + u快捷键可以弹窗表达式调试弹窗：
![image](http://img.blog.csdn.net/20160718151943887)

这个功能非常实用，可以在断点处直接进入一个求值环境，在这里你可以执行任何你感兴趣的表达式可以在表达式调试弹窗中写任何java表达式，比如：
![image](http://img.blog.csdn.net/20160718152045815)

其他的比如在断点处有一个对象object，如果你要查看它的某个属性很简单，在Debug窗口就能看到，但是如果你想要执行它的某个方法看看结果是什么呢？借助这个可以实现。当然它的功能远不止这么多，相当于直接进入了一个 REPL环境，非常实用。

**条件断点**

假设你的断点在一个列表的循环里面，可是你只对这个列表的某一个元素感兴趣，只想在遇到这个元素的时候才断下来；你是一直人肉 F9 直到满足条件吗？条件断点就是满足这种需求的，顾名思义，在特定条件下的断点。使用起来也非常简单，在你的断点上鼠标右键会出现一个小窗口，写上条件即可。
![image](http://img.blog.csdn.net/20160718152336957)

**其他断点调试方式**

除了这里的求值断点，条件断点，还有日志断点，异常断点，方法断点等，具体的可自行google哈。


**总结：**

- android默认提供了Log API实现对日志的打印功能；

- 可以实现自定义日志框架，定制化显示日志样式，定制化配置是否显示日志等，可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/51707796">android日志框架</a>

- 可以通过配置gradle，让测试环境中显示日志信息，在正式环境中不显示日志信息；

- android studio提供了断点调试功能，包含了求值调试，条件断点，日志断点，方法断点，异常断点等等；

<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">android产品研发（十九）-->android studio中的单元测试</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080">android产品研发（二十）-->代码Review</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868453">android产品研发（二十一）-->android中的UI优化</a>

