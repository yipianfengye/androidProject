本文我们将讲解Android开发中常常涉及到的MVC/MVP/MVVM等模式的基本概念。许多童鞋对Android开发中涉及到的MVC、MVP、MVVM这三种模式不是太清楚，我认为无论是MVC、MVP亦或者是MVVM都是一种代码组织方式，通过这种代码组织方式能够让代码更有层次感，各个层次主要负责各自的工作，这样降低了整个项目的代码逻辑耦合度与可读性。


下面对MVC、MVP、MVVM等设计模式逐一的做一下说明：

**MVC开发模式：**

MVC，即Model层，View层，Control层，在JAVAEE中MVC是一种经典的开发模型，下面是引用的一段对其的说明：

> MVC全名是Model View Controller，是模型(model)－视图(view)－控制器(controller)的缩写，一种软件设计典范，用一种业务逻辑、数据、界面显示分离的方法组织代码，将业务逻辑聚集到一个部件里面，在改进和个性化定制界面及用户交互的同时，不需要重新编写业务逻辑。其中M层处理数据，业务逻辑等；V层处理界面的显示结果；C层起到桥梁的作用，来控制V层和M层通信以此来达到分离视图显示和业务逻辑层。

简单来讲就是：

- 视图（View）：用户界面

- 控制器（Controller）：业务逻辑

- 模型（Model）：数据保存


**MVC它的数据流转：**

![这里写图片描述](http://img.blog.csdn.net/20160226102237411)
（盗用了一下网上的图...）

- 用户操作界面，View接受指令，View 传送指令到 Controller，也就是从View层到Control层的箭头所示

- Controller 完成业务逻辑后，要求 Model 改变状态，也就是从Control层到Model层的箭头所示

- Model 将新的数据发送到 View，用户得到反馈，也就是从Model层到View层的箭头所示

**Android中MVC模式的体现：**

其实Android开发的主要流程都是MVC模式的，比如我们常见的Activity+Layout+Model展示业务逻辑的模式，其中：

Activity - 对应着Controller层，主要是控制层，用于实现业务逻辑

Layout - 对应着View层，主要用于展示页面

Model - 对应着Model层，主要是保存数据

**MVC模式的优势：**

- 使用MVC模式降低了程序中的耦合度，使应用程序视图层与Model层分离，减少了代码之间的相互影响；

- 由于使用MVC模式降低代码耦合度，因此可以很方便的扩展现有程序；

- 不同代码模块职责划分明确，有利于代码的维护与升级；


**MVP开发模式：**

MVP开发模式是MVC模式一种进阶，MVP和MVC模型的主要区别是model层与View层不再发生关系而是通过Presenter层作为中间的枢纽。并且各个部分之间都是双向关联的；

简单来讲就是：

- 视图（View）：用户界面

- 控制层（Presenter）：业务逻辑（负责与View层和Model层双向交互）

- 模型（Model）：数据保存

**MVP它的具体数据流转是这样的：**

![这里写图片描述](http://img.blog.csdn.net/20160226102451834)
（盗用了一下网上的图...）

- 用户操作界面，View接收指令，View传送指令到Presenter层，也就是从View层到Presenter层的箭头所示

- Presenter完成业务逻辑后，要求Model改变状态，也就是从Presenter层到Model层的箭头所示

- Model状态改变之后将结果返回给Presenter层，然后Presenter层在将结果反馈到View层，也就是从Model层到Presenter层，从Presenter层到View层的箭头所示

在MVP里，Presenter完全把Model和View进行了分离，主要的程序逻辑在Presenter里实现。而且，Presenter与具体的View是没有直接关联的，而是通过定义好的接口进行交互，从而使得在变更View时候可以保持Presenter的不变。

**MVP模式的优势：**

- MPV开发模式与MVC开发模式有的优势相似，都是降低了代码的耦合度

- 使用MVP模式View层与Model层不在相互关联，可以更高效地使用模型，因为所有的交互都发生在一个地方——Presenter内部


**MVVM它的具体数据流转是这样的：**

MVVM与MVP是相类似的，唯一的区别是，它采用双向绑定（data-binding）：View的变动，自动反映在 ViewModel，反之亦然。

简单来讲就是：

- 视图（View）：用户界面

- 控制层（VM）：业务逻辑（负责与View层和Model层双向交互）

- 模型（Model）：数据保存

![这里写图片描述](http://img.blog.csdn.net/20160226102636272)
（盗用了一下网上的图...）

- 用户操作界面，View接收指令，View传送指令到Presenter层

- ViewModel完成业务，改变Model层数据

- Model状态改变之后将结果返回该ViewModel层，然后ViewModel层自动更新View层显示



注：google提供的官方data binding框架采用的就是MVVM模型


参考文章：
<a href="http://www.cnblogs.com/lori/p/3501764.html">MVVM大话开篇</a>
<a href="http://www.ruanyifeng.com/blog/2015/02/mvcmvp_mvvm.html">MVC，MVP 和 MVVM 的图示</a>


<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">Android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">Android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">Android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">Android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">Android产品研发（十九）-->Android studio中的单元测试</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080">Android产品研发（二十）-->代码Review</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868453">Android产品研发（二十一）-->Android中的UI优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868496">Android产品研发（二十二）-->Android实用调试技巧</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51953926">Android产品研发（二十三）-->Android中保存静态秘钥实践</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/52671165"> Android产品研发（二十四）-->内存泄露场景与检测</a>
