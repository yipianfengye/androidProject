上一篇文章中我们讲解了android中内存对象的序列化方式。由于android开发涉及到不同Activity的数据传递，对于基本数据类型数据的传递是没有问题的，但是一旦涉及到复杂数据类型，就需要将数据序列化以便传输，在文章中我们主要讲解了两种数据序列化的方式：实现Serializable接口和实现Parcelable接口，同时也比较了它们各自的优缺点和实现方式。具体关于内存对象序列化方面的知识可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>

本文主要介绍Android开发中常常涉及到但又不是被人重视知识点：开发者选项。主要涉及到如何打开开发者模式，开发者选项中有哪些操作菜单以及各自的作用，如何清除手机数据，清除手机数据具体清除那些数据等等。

一般而言，不同的手机开发者选项界面是不太相同的，这是由于手机的设置界面都被做了定制化处理，但是其基本的功能菜单都是类似的。下面我们就先来看一下如何打开手机的开发者模式。

**如何打开开发者选项菜单？**

不同的手机进入开发者选项的菜单可能不太一样，但是基本的大概的可能是：

- 设置 

- 关于手机 

- android版本号 

- 连续点击N次 

- 弹出进入开发者模式说明

经过上面的步骤，我们就打开了手机的开发者模式，在进入了开发者模式之后我们就可以在设置页面或者是设置里面的其他设置，高级设置等等菜单之中找找是否出现了开发者选项的菜单，若出现了开发者选项菜单我们就可以根据自己的需求选择性的打开各种控制开关了。


**开发者选项中提供了那些功能？**

知道了如何把手机进入开发者模式之后，在我们的日常开发过程中，不可避免的会使用到android开发者选项这一个功能，比如使用真机在android studio中调试App等等，那么开发者选项中到底有哪些功能呢？一下就是开发者选项中提供的功能呢列表：

![这里写图片描述](http://img.blog.csdn.net/20160114145314118)


**开发者选项中的具体功能**

这里以红米note2的开发者选项说明一下各个选项的具体功能：

- 开启开发者选项 
这是开发者选项的控制开发，打开这个才算开启了开发者选项，并且下面的选项功能才可以使用

- 提交错误报告
将本机上安卓系统的出错日志以及硬件设备信息发送给谷歌。一般是发送不到的，原因你懂的！所以开不开启都无所谓的。

- 不锁定屏幕
解释很清楚，充电时不会休眠，比如我们在使用手机调试程序的时候，一会手机就锁屏了，很麻烦，如果我们打开这个设置之后，无论什么时候我们的手机都不会在锁屏了，很方便

- 直接进入系统
很实用，就是开发过程中点击屏幕直接进入系统而不会锁屏

- 打开蓝牙数据包日志
这个选项会抓取所有的蓝牙数据包保存到一个文件中，在调试蓝牙程序的时候比较好用

- 进程统计信息
主要用于统计系统程序的后台信息
![这里写图片描述](http://img.blog.csdn.net/20160114150056836)
可以查看一些程序使用时长，内存占用等信息；

- USB调试
这是手机能够连接电脑的关键操作，只有开启了这个选项手机才能连接到电脑，并进行调试，很多时候我们的手机连接不到电脑都是因为我们打开了开发者模式，但是允许USB调试的开关没有打开，这时候重新打开USB调试，可能手机就能连接到电脑了

- 允许模拟位置
允许代码模拟位置，比如地图类应用需要测试在外地的使用情况，通过开启此项选项可以通过代码模拟位置

- 选择调试应用
设定需要调试的应用程序，以android studio为例，设定调试程序之后，Android monitor窗口的默认选择程序就是设定的调试程序。当然我们也可以在手机的开发者选项中选择需要调试的应用程序

- 显示触摸操作
可以在屏幕中显性的展示触摸的轨迹

- 指针位置
可以显示触摸的指针坐标点

- 显示边界布局
主要用于显示布局的边界，比如一个Activity显示界面中各种布局文件的边界等

- 窗口动画缩放
可以设置动画的缩放效果

- 动画程序时常缩放
可以设置动画程序播放时长

- 模拟辅助显示设备
小米手机中改选项可以模拟各种屏幕分辨路的显示效果

- 调试GPU过度绘制
主要用于显示在界面是否存在过度绘制的现象
一共有四种颜色：蓝色、绿色、淡红、深红。根据过度绘制的次数，依次递增。1x过度绘制是蓝色、2x是绿色、3x是淡红、4x是深红。具体关于android中过度绘制的问题，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/50521909">android中的过度绘制</a>

- 后台进程限制
主要用于限制后台进程的数量

- 系统内存优化级别
主要用于设置系统内存的优化级别


当然了以上介绍的这些选项是开发者选项中提供核心功能的菜单，此外还有一些其他选项，大家可以多了解一下。

**清除App数据**

下面我们将在开发者选项的基础上介绍一个其他方面的内容--清除App数据。

**什么是清除App数据？** 

手机在运行过程中会在手机端保存一些临时数据，配置数据，运行数据等，这些数据可能以配置文件，数据库文件等形式保存在手机端，android手机在设置页面提供了清除App数据的功能，可以通过这个功能实现对App保存数据的清除操作。

**如何进行清除App数据**

我们可以通过如下步骤实现对App数据的清除工作：

- 手机设置

- 应用管理

- 某一应用

- 清除数据

这样通过如上的操作步骤我们就将这个App的数据清除了，但是这样操作之后到底会清除App那些数据呢？

**清除App数据的类型**

- 这里新建一个项目com.chao.ttext，我们在项目数据目录:data/data/com.chao.ttext目录下创建缓存数据目录，具体目录结果如下所示：

```
data/data/com.chao.ttext # ls
lib 存放使用的包
files 存放应用程序自己保存的文件
databases 存放数据库数据
shared_prefs SP文件
cache 存放缓存数据
app_appcache H5缓存
app_databases webview缓存
app_geolocation 定位缓存
```

- 然后我们为每个目录添加一个新的空文件，这里暂时使用linux命令：touch，在每个目录中添加数据文件用于判断清除数据的结果：

```
/data/data/com.chao.ttext # touch lib/temp.txt
/data/data/com.chao.ttext # touch files/temp.txt
/data/data/com.chao.ttext # touch databases/temp.txt
/data/data/com.chao.ttext # touch shared_prefs/temp.txt
/data/data/com.chao.ttext # touch cache/temp.txt
/data/data/com.chao.ttext # touch app_appcache/temp.txt
/data/data/com.chao.ttext # touch app_databases/temp.txt
/data/data/com.chao.ttext # touch app_geolocation/temp.txt
```

- 继续的我们执行清除App数据的操作，即：打开设置-》应用管理-》ttext-》清除数据

- 最后我们查看一下执行了清除数据操作之后的数据目录即查看ttext数据目录下的数据情况：

```
/data/data/com.chao.ttext # ls
lib
```
然后进入lib目录查看temp.txt文件是否还存在，结果还是存在的。

**结论：清除数据会清除App数据目录下除lib文件以外的所有文件和目录。**


**总结：**

- 在android开发中常常会使用到开发者选项，可以通过设置关于手机android版本号连续点击的方式进入开发者选项

- 常见的手机无法连接电脑可能是USB调试开关没有打开的原因，可以尝试打开USB调试连接电脑

- 开发者选项中有一些比较实用的功能可能会在调试App的时候用到，比如：不锁屏，GPU调试，动画调试等等

- 清除App数据会清除App数据目录下除lib文件以外的所有文件和目录

- 清除App数据，会使App进程被杀死，也就是说执行了清除App数据的操作之后再次打开App都是重新打开一个新的进程


<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261">android产品研发（五）-->多渠道打包</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51581491">android产品研发（六）-->Apk混淆</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51587927">android产品研发（七）-->Apk热修复</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51598041">android产品研发（八）-->App数据统计</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51612429">android产品研发（九）-->App网络传输协议</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51655330">android产品研发（十）-->不使用静态变量保存数据</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51685310">android产品研发（十一）-->应用内跳转scheme协议</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>
