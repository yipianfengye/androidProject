上一篇文章中我们讲解了如何在android studio中进行单元测试。实际开发过程中有一些功能性的需求，比如测试工具类，测试数据存储等测试工作，如果还是通过重复执行apk文件的编译，安装，运行等会浪费大量的时间，而这些功能与android的开发环境无太大的关系，我们完全可以使用单元测试来执行。android studio中默认是支持进行单元测试的，并提供了获取Context等系统对象的API，我们可以通过其系统提供的API获取Context等对象，进而测试相应的功能。
具体更多关于android studio中进行单元测试的知识点，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51868451">android studio中的单元测试</a>

本文我们将讲解android中的代码Review。良好的产品开发迭代过程中，代码Review是一个必不可少的步骤，通过代码Review能够提高产品质量，增强团队成员之间的沟通，提高开发效率。所以团队开发活动中定时进行代码Review就显得很有必要了。


**一：什么是代码Review？**

review字面的意思是再次查看，代码review就是代码再次查看评审的意思。这里我们首先看一下百科中对代码Review中的定义：

> 代码评审是指在软件开发过程中，通过对源代码进行系统性检查的过程。通常的目的是查找系统缺陷，保证软件总体质量和提高开发者自身水平。 Code Review是轻量级代码评审，相对于正式代码评审，轻量级代码评审所需要的各种成本要明显低的多，如果流程正确，它可以起到更加积极的效果。正因如此，轻量级代码评审经常性得被引入到软件开发过程中。

我理解的代码review就是在android开发过程中，每个迭代周期编码结束之后测试之前，团队成员之间相互查看对方的代码，不一定是逐行逐句的查看，主要是对核心方法，编程思路有一个大概的评审查看。

**二：代码Review的好处**

- 通过代码Review可以提高产品代码的质量

- 通过代码Review可以增强团队成员之间的沟通

- 通过代码Review能够有效的提前发现代码中存在的缺陷与BUG，降低线上出现事故的概率

- 通过代码Review可以提高团队成员的编程能力，不同成员之间对功能设计思路的重构可以很好的提高团队成员的个人专业技能

**三：为何需要代码Review**

- 由于团队成员之间既有新成员又有老成员，既有大神也有菜鸟，为了提高产品代码质量，所以团队成员在产品迭代周期中固定时间进行代码Review有利于提高全队成员对产品代码的理解与个人能力的提高。

- 通过代码Review，团队成员对产品的每个模块都有了认识，对App上出现的错误信息立即反馈与修复。

**四：代码Review实践**

团队成员每周都会有固定的时间进行代码Review，每次Review的时间大概在半小时至一小时之间。

主要流程：

- 每个人介绍各自的功能需求，实现的主要逻辑，核心代码等；

- 团队成员提出问题，其他实现思路等；

- 讨论不同实现思路的方式以及优劣势；


**五：android代码lint检查**

除了队员之间的代码Review，还可以通过android 代码lint的方式review代码。android studio默认已经提供了强大的lint检查工具，通过其我们可以很方便的发现代码中存在的问题，修正可能出现的bug等。

- 通过android studio编译工具执行lint检查操作

（1）执行android studio --> Analyze --> Inspect code操作，打开代码检查框
<br>![image](http://img.blog.csdn.net/20160713181633868)

（2）在代码检查框中选择为整个工程执行lint检查？还是整个module或者是当前的源文件执行lint检查，这里为了简单起见，我们只为当前的源代码文件执行lint检查，然后执行确认即可
<br>![image](http://img.blog.csdn.net/20160713181822659)

（3）接下来就可以在我们的android studio查看lint检查结果了
<br>![image](http://img.blog.csdn.net/20160713182133203)

可以发现我们lint检查之后出现了许多检查结果，其中在uuelectricrenter项目下存在着92条检查信息，下面我们就分析一下检测结果。

**六：取消无用的lint检查**

- Android > Lint > Correntness

![image](http://img.blog.csdn.net/20160714145816992)

可以看到在Correctness栏目下列出了出现问题的条目，而在右侧则列出了出现问题的源码文件，位置，问题描述，建议方案等：

![image](http://img.blog.csdn.net/20160714150339576)

可以发现该lint问题是在MainMapFragment源码文件的188行，我们找到改源码文件的第188行，看一下源码是怎么样写的：

```
rootView = inflater.inflate(R.layout.fragment_main_map, null);
```
可以发现使用布局加载器的时候调用inflate方法第二个parentView参数我们传递的是null，这时候lint检查就会报错，当然了在程序中这样写是没有问题的，而我们以后不想lint检查的时候在检查出这个问题，那么怎么办呢？

选中lint检查条目 --> 右键 --> Disable inspection，这时候我们再次执行lint检查，发现就无法检测出这个问题了：
![image](http://img.blog.csdn.net/20160714150754724)

**七：修复lint检查异常**

在看一个lint检查结果：
![image](http://img.blog.csdn.net/20160714151006618)

好吧，这样看的不是太清楚，我们看一下lint描述：
![image](http://img.blog.csdn.net/20160714151039173)

问题描述是：在MainMapFragment的1635行，调用setText方法的时候没有使用string资源，那我们就看一下改行代码的实现：

```
mTvMsg.setText("步行" + result);
```
可以看到我们为TextView设置text字符串的时候直接硬编码写入了text字符串，所以这时候报了lint检查异常，这时候我们可以通过调用string资源的方式修改，然后再次执行lint检查操作：

![image](http://img.blog.csdn.net/20160714151344233)

可以发现这时候已经没有了刚刚的lint检查异常。


**八：如何设置lint检查**

android studio --> Perferences --> Deitor Inspections
![image](http://img.blog.csdn.net/20160714152436851)

一个简单的设置lint检查的例子

我们在android布局文件中为TextView设置text的时候可能直接硬编码写入字符串：

```
<TextView
        android:layout_width="match_parent"
        android:layout_height="wrap_content"
        android:text="这是测试Text"
        android:layout_marginTop="20dp"
        android:layout_marginLeft="20dp"
        android:layout_marginRight="20dp"
        android:layout_gravity="bottom"
        />
```
而这时候android studio并没有为我们提示什么错误信息，我们可以更改lint检查提示，让其一旦检测到有布局文件硬编码的情况就报错，还是和上面的步骤一样：
android studio --> perferences --> Editor --> Inspections --> Hadrcoded text
![image](http://img.blog.csdn.net/20160714152911993)

我们这时候修改器severity级别，由Warning更改为Error，这时候就可以看到错误信息了：
![image](http://img.blog.csdn.net/20160714153043178)


**九：其他相关的lint检查：**

<a href="http://blog.csdn.net/u010015108/article/details/51190725">Android Studio Lint 自动检查清除冗余资源</a>
<a href="http://www.cnblogs.com/cheerego/p/5175764.html">Android APK瘦身之Android Studio Lint (代码审查)</a>
<a href="http://tech.meituan.com/android_custom_lint.html">Android自定义Lint实践</a>

**十：关于lint检查的快捷键**

上文中我们讲解的是通过：
android studio --> Analyze --> Inspect code
的方式执行lint检查，android studio中还可以通过快捷键的方式直接打开lint检查对话框：

shift + control + A --> 输入Ins --> 选择Inspect Code
这样就可以直接打开lint检查对话框了：
![image](http://img.blog.csdn.net/20160714153943002)

**十一：android开发规范**

代码Review过程不单单是团队check，代码lint，一个好的开发规范也是很有必要的，这里推荐我的另一篇文章：<a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a> 个人感觉产品研发过程中，开发规范真的很重要，很重要，非常重要（重要的事情说三遍），一个好的开发规范可以让团队中的人对他人的代码更熟悉，新人也可以更好的了解产品的业务逻辑。开发规范并不是一个死的一成不变的，每个团队可能都有自己的开发规范，只要是适合团队的开发规范就是最好的开发规范。


**总结：**
本文我们讲解了android产品研发过程中代码Review的相关知识，有团队代码Review，android studio代码lint检查，android开发规范等相关知识。代码Review的方式不是非常重要，重要的是保持一个良好的代码Review流程，这样才能在不断的代码Review过程中提高产品代码质量，增强团队成员之间的沟通。

<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51685310">android产品研发（十一）-->应用内跳转scheme协议</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">android产品研发（十九）-->android studio中的单元测试</a>
