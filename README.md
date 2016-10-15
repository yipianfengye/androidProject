# AndroidProject

<img src="http://ac-mhke0kuv.clouddn.com/ff054257bc88c000d0b5.png?imageView/1/w/220/h/120/q/100/format/png" width="1000" height="250">

<br>最近的android产品研发系列主要讲解的是android产品研发过程中涉及到的技术，技巧，实践等。前面我们讲解了android源码系列的文章&nbsp;&nbsp;可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/50696046">Android源码解析-->总结（持续更新中）</a>，源码系列的文章东西比较多比较复杂，并且一些东西还没有讲完，这里已经更新了30篇了，后续的东西一定会更新的。考虑一直讲源码系列可能看的比较累，这里就有了产品研发系列的文章。本个系列的文章主要是讲解android产品研发过程中一些需要注意的技术技巧与实践。其主要面对产品研发，对App稳定性，友好型，兼容性要求较高的App。

下面就是我准备讲解的一些产品研发系列的内容：
（其中超链接字体的文章是我已经写完的部分，其他的是我还没写但是打算写的东西，这些东西大概覆盖了android产品研发过程中涉及到的各个方面，当然了有可能后续也会有所补充）

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013"><font color="red">Android产品研发（一）-->实用开发规范</font></a>
> 本文中选择将开发规范作为这个系列的第一篇文章，就是个人感觉产品研发过程中，开发规范真的很重要，很重要，非常重要（重要的事情说三遍），一个好的开发规范可以让团队中的人对他人的代码更熟悉，新人也可以更好的了解产品的业务逻辑。开发规范并不是一个死的一成不变的，每个团队可能都有自己的开发规范，只要是适合团队的开发规范就是最好的开发规范...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277"><font color="red">Android产品研发（二）-->启动页优化</a>
> 这篇文章中我们主要介绍一下启动页的优化，关于启动页的优化是UE方面的内容了，一个很慢的启动页很容易让用户觉得受不了，进而“逃离”App的，所以若想产品有更好的用户体验，做一些启动页的优化是一个不错的选择。这里我们简单介绍一下我在实践中对启动页是如何优化的...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974"><font color="red">Android产品研发（三）-->基类Activity</a>
> 在实际的android产品研发中，一般的我们在写Activity的时候都会继承于一个基类Activity，该Activity是所有的Activity的基类。在该基类中我们主要用于重写一些共有的逻辑。好处是显而易见的对于一些Activity的共有逻辑我们不必要在每个Activity中都重新写一遍，只需要在基类Activity中写一遍就好了...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066"><font color="red">Android产品研发（四）-->减小Apk大小</a>
> 随着移动技术的深入发展，各种炫酷效果的更新，在我们追求UI与UE的同时一个不如忽视的问题逐渐暴露出来，那就是apk文件越来越大，可能有的童鞋会说现在都是wifi环境，apk文件增大几M不是什么大不了的问题，这其实也是有一定道理的，但是作为开发人员的我们这绝不是我们认为可以忽略这个问题的理由。优化Apk大小也是优化我们App体验的一个重要方面，虽然可能它不是那么的重要...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261"><font color="red">Android产品研发（五）-->多渠道打包</a>
> 国内的Android开发者还是很苦逼的，由于众所周知的原因，google play无法在国内打开（翻墙的就不在考虑之内了），所以android系的应用市场，群雄争霸。后果就是国内存在着有众多的应用市场，产品在不同的渠道可能有这不同的统计需求，为此android开发人员需要为每个应用市场发布一个安装包，这里就引出了android的多渠道打包...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51581491"><font color="red">Android产品研发（六）-->Apk混淆</a>
> 本文主要讲解Apk的混淆，这里的混淆分为两种代码混淆和资源文件混淆。实际的产品研发中为了防止自己的劳动成果被别人窃取，混淆代码能有效防止apk文件被反编译，进而查看源代码。说来惭愧，作为互联网创业公司的我们也确实对竞品Apk反编译研究过，如果Apk混淆之后确实对理解源码的业务流程造成了困扰，这也从侧面说明了Apk混淆的重要性...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51587927"> <font color="red">Android产品研发（七）-->Apk热修复</a>
> 去年一整年android社区中刮过了一阵热修复的风，各大厂商，逼格大牛纷纷开源了热修复框架，恩，产品过程中怎么可能没有bug呢？重新打包上线？成本太高用户体验也不好，咋办？上热修复呗。好吧，既然要开始上热修复的功能，那么就得调研一下热修复的原理。下面我将分别讲述一下热修复的原理，各大热修复框架的比较，以及自身产品中热修复功能的实践

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51598041"><font color="red">Android产品研发（八）-->App数据统计</a>
> 文本将要介绍一下android产品中另一项基础功能-数据统计。App数据统计的意义在于通过统计用户的行为方式有针对性的更新展示算法，根据用户的行为习惯更新产品功能等等，具体而言当我们开发好App后就会把它发到应用市场上，但是目前有很多的应用市场(如，豌豆荚，应用宝，安卓市场等)，那么问题来了，假如我们想统计我们开发的应用的下载次数，就必须使用统计吧，而且它不仅可以统计我们的应用的下载量，启动次数，还可以统计页面访问量、查看程序的bug等等，所以相对于项目而言产品由于存在着持续的迭代与用户体验，所以做好数据统计工作是一项必不可少的工作...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51612429"><font color="red">Android产品研发（九）-->App网络传输协议</a>
> 本文将要介绍的是App的网络传输协议。App与服务器交互就会涉及到信息的交换，而信息的交互就必然需要一套完整的数据协议。这里首先需要明确一点的是什么是网络传输协议呢？这里首先套用一段百度百科的定义...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51655330"><font color="red">Android产品研发（十）-->不使用静态变量保存数据</a>
> 本文讲解的其实并不是一个技术方面，而是一个android产品研发过程中的技巧：尽量不使用静态变量保存核心数据。这是为什么呢？这是因为android系统中的应用进程并不是安全的，包括application对象、静态变量在内的进程级别变量并不会一直呆着内存里面，它会被kill掉，它真的有可能会被kill掉，真的真的，重要的事情说三遍。与大家普遍的看法不同之处在于，当进程被干掉之后，实际上app不会重新开始启动。Android系统会创建一个新的Application对象，然后启动上次用户离开时的activity以造成这个app从来没有被kill掉得假象。而这时候静态变量等数据由于进程已经被杀死而被初始化，所以就有了我们的不推荐在静态变量（包括Application中保存全局数据静态数据）的观点...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51685310"><font color="red">Android产品研发（十一）-->使用scheme实现页面跳转</a>
> 本文讲解的是一种App内页面跳转协议，这里的跳转包括应用内跳转、H5与Native跳转，服务器通知客户端如何跳转等。在讲解应用内跳转协议之前我们先讲解一下H5与Native相互跳转的相关知识点...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047"><font color="red">Android产品研发（十二）-->App长连接实现</a>
> 本文中我们将讲解一下App的长连接实现。一般而言长连接已经是App的标配了，推送功能的实现基础就是长连接，当然了我们也可以通过轮训操作实现推送功能，但是轮训一般及时性比较差，而且网络消耗与电量销毁比较多，因此一般推送功能都是通过长连接实现的...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389"><font color="red">Android产品研发（十三）-->App轮训操作</a>
> 本文将讲解app端的轮询请求服务，一般而言我们经常将轮询操作用于请求服务器。比如某一个页面我们有定时任务需要时时的从服务器获取更新信息并显示，比如当长连接断掉之后我们可能需要启动轮询请求作为长连接的补充等，所以这时候就用到了轮询服务...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773"><font color="red">Android产品研发（十四）-->App升级与更新</a>
> 本文将讲解app的升级与更新。一般而言用户使用App的时候升级提醒有两种方式获得：一种是通过App Store获取、一种是打开应用之后提醒用户更新升级，而更新操作一般是在用户点击了升级按钮之后开始执行的，这里的升级操作也分为两种形式：一般升级、强制升级...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528"><font color="red">Android产品研发（十五）-->内存对象序列化</a>
> 本文将讲解android中数据传输中需要了解的数据序列化方面的知识，我们知道android开发过程中不同Activity之间传输数据可以通过Intent对象的put**方法传递，对于java的八大基本数据类型(char int float double long short boolean byte)传递是没有问题的，但是如果传递比较复杂的对象类型（比如对象，比如集合等），那么就可能存在问题，而这时候也就引入了数据序列化的概念...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497"><font color="red">Android产品研发（十六）-->开发者选项</a>
> 本文主要介绍Android开发中常常涉及到但又不是被人重视知识点：开发者选项。主要涉及到如何打开开发者模式，开发者选项中有哪些操作菜单以及各自的作用，如何清除手机数据，清除手机数据具体清除那些数据等等...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985"><font color="red">Android产品研发（十七）-->Hybird开发</a>
> 本文将介绍android中hybrid开发相关的知识点。hybrid开发实际上是混合开发的意思，这里的混合是H5开发与Native开发混合的意思。下面的文章中我们将逐个介绍一下hybrid开发的概念、hybrid开发的优势、android中如何实现hybrid开发、简单的hybrid开发的例子，以及在产品实践中对hybrid开发的应用，希望通过本篇文章的介绍让您能够对android中的hybrid开发有一个基本的认识...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139"><font color="red">Android产品研发（十八）-->webview问题集锦</a>
> 本文中我们将介绍一下android中webview在使用过程中会遇到的一些问题。这些问题主要是webview在使用过程中我已经趟过的坑，希望通过这篇文章的介绍能够帮助大家更好的使用webview...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451"><font color="red">Android产品研发（十九）-->android studio中的单元测试</a>
> 本文我们将讲解如何在android studio中进行单元测试。在android开发项目中，经常会进行测试操作，而一次又一次的运行模拟器，浪费了大量时间，降低了工作效率降低，虽然最新的android studio中提供了instance run功能，来提高android studio的编译速度，但是我们还是需要了解android studio的单元测试功能，其可以很方便的为我们提供功能性测试，所以如果项目中有用到测试数据的时候，可以先进行单元测试，如果可以正常输出数据了，然后再到UI中执行，这样会提高一些工作效率...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080"><font color="red">Android产品研发（二十）-->代码Review</a>
> 本文我们将讲解android中的代码Review。良好的产品开发迭代过程中，代码Review是一个必不可少的步骤，通过代码Review能够提高产品质量，增强团队成员之间的沟通，提高开发效率。所以团队开发活动中定时进行代码Review就显得很有必要了...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868453"><font color="red">Android产品研发（二十一）-->android中的UI优化</a>
> 本文我们将讲解一下android UI优化方面的知识。android系统的优化分为好多方面：比如性能优化，UI优化，资源文件优化等等，这里我们先暂时讲解android UI优化方面的知识点...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868496"><font color="red">Android产品研发（二十二）-->android实用调试技巧</a>
> 本文我们将讲解android中的调试技巧。程序调试，是将编制的程序投入实际运行前，用手工或编译程序等方法进行测试，修正语法错误和逻辑错误的过程。这是保证计算机信息系统正确性的必不可少的步骤。在android开发过程中熟练的使用调试技巧是一个很重要的方面。android的调试技巧包括熟练使用android中的日志API，自定义android日志框架，通过gradle配置调试日志，android studio的调试技巧等等。通过对本文的学习我们能够对android中调试技巧有一个大概的了解...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/51953926"><font color="red">Android产品研发（二十三）-->android中保存静态秘钥实践</a>
> 本文我们将讲解一个android产品研发中可能会碰到的一个问题：如何在App中保存静态秘钥以及保证其安全性。许多的移动app需要在app端保存一些静态字符串常量，其可能是静态秘钥、第三方appId等。在保存这些字符串常量的时候就涉及到了如何保证秘钥的安全性问题。如何保证在App中静态秘钥唯一且正确安全，这是一个很重要的问题，公司的产品中就存在着静态字符串常量类型的秘钥，所以一个明显的问题就是如何生成秘钥，保证秘钥的安全性？

<br><a href="http://blog.csdn.net/qq_23547831/article/details/52671165"><font color="red"> Android产品研发（二十四）-->内存泄露场景与检测</a>
> 本文我们将讲解一下关于Android开发过程中常见的内存泄露场景与检测方案。Android系统为每个应用程序分配的内存是有限的，当一个应用中产生的内存泄漏的情况比较多时，这就会导致应用所需要的内存超过这个系统分配的内存限额，进而造成了内存溢出而导致应用崩溃。在实际的开发过程中我们由于对程序代码的不当操作随时都有可能造成内存泄露...

<br><a href="http://blog.csdn.net/qq_23547831/article/details/52723149"><font color="red"> Android产品研发（二十五）-->Android开发MVC/MVVM/MVP框架</a>
> 本文我们将讲解Android开发中常常涉及到的MVC/MVP/MVVM等模式的基本概念。许多童鞋对Android开发中涉及到的MVC、MVP、MVVM这三种模式不是太清楚，我认为无论是MVC、MVP亦或者是MVVM都是一种代码组织方式，通过这种代码组织方式能够让代码更有层次感，各个层次主要负责各自的工作，这样降低了整个项目的代码逻辑耦合度与可读性...


<br>android产品研发之Https请求；
<br>android产品研发之拦截App请求；
<br>android产品研发之定位服务；
<br>android产品研发之持续集成；
<br>android产品研发之常用框架；
<br>android产品研发之git使用；
<br>android产品研发之产品加固与加密；
<br>android产品研发之屏幕适配
<br>android产品研发之Fragment化
<br>android产品研发之基础组件SDK化
<br>android产品研发之React Native开发
<br>android产品研发之RxAndroid
<br>android产品研发之性能优化
<br>android产品研发之控件MD化

作为IT人员我还是比较强调做产品而不是做项目的，因为做项目都是跟着项目走许多东西做完了也就做完了，没有深入进去，没有持续的迭代与优化，相当于做一件事做了N遍，这样对个人很难有技能上的提高。持续的迭代一个产品不但能够在深度也能在广度上提高自己，如果可以的话强烈建议大家持续的迭代某一个产品。

这里多说几句，这里只是在技术上关于android产品研发的一些tip，在产品上我们同样的是需要有自己的思考。为了让App更好用，更好看，更简单，多多的站在用户的角度上思考，这不但是产品经理的任务，同样也是我们程序员需要做的。只有在和用户交流的时候，你才会发现，你觉得很好的东西用户可能根本不会用，有时候，专业术语导致用户完全不理解。按钮很明显但用户完全没看到，为什么?因为用户的注意力被其他功能给扰乱了，这些问题都是产品的复杂造成的。说这么多就是想跟大家说做产品需要的时候更多的需要从用户的角度考虑问题，而不是站在你的角度想用户的问题，有时候你所想的问题可能并不是用户想要的。
