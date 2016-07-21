# androidProject

最近的android产品研发系列主要讲解的是android产品研发过程中涉及到的技术，技巧，实践等。前面我们讲解了android源码系列的文章&nbsp;&nbsp;可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/50696046">android源码解析-->总结（持续更新中）</a>，源码系列的文章东西比较多比较复杂，并且一些东西还没有讲完，这里已经更新了30篇了，后续的东西一定会更新的。考虑一直讲源码系列可能看的比较累，这里就有了产品研发系列的文章。本个系列的文章主要是讲解android产品研发过程中一些需要注意的技术技巧与实践。其主要面对产品研发，对App稳定性，友好型，兼容性要求较高的App。

下面就是我准备讲解的一些产品研发系列的内容：
（其中红色字体的文章是我已经写完的部分，其他的是我还没写但是打算写的东西，这些东西大概覆盖了android产品研发过程中涉及到的各个方面，当然了有可能后续也会有所补充）

<a href="http://blog.csdn.net/qq_23547831/article/details/51534013"><font color="red">android产品研发（一）-->实用开发规范</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51541277"><font color="red">android产品研发（二）-->启动页优化</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51546974"><font color="red">android产品研发（三）-->基类Activity</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51559066"><font color="red">android产品研发（四）-->减小Apk大小</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51569261"><font color="red">android产品研发（五）-->多渠道打包</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51581491"><font color="red">android产品研发（六）-->Apk混淆</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51587927"> <font color="red">android产品研发（七）-->Apk热修复</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51598041"><font color="red">android产品研发（八）-->App数据统计</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51612429"><font color="red">android产品研发（九）-->App网络传输协议</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51655330"><font color="red">android产品研发（十）-->不使用静态变量保存数据</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51685310"><font color="red">android产品研发（十一）-->使用scheme实现页面跳转</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51690047"><font color="red">android产品研发（十二）-->App长连接实现</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51719389"><font color="red">android产品研发（十三）-->App轮训操作</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51764773"><font color="red">android产品研发（十四）-->App升级与更新</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51779528"><font color="red">android产品研发（十五）-->内存对象序列化</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51809497"><font color="red">android产品研发（十六）-->开发者选项</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51812985"><font color="red">android产品研发（十七）-->Hybird开发</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51820139"><font color="red">android产品研发（十八）-->webview问题集锦</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51868451"><font color="red">android产品研发（十九）-->android studio中的单元测试</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51833080"><font color="red">android产品研发（二十）-->代码Review</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51868453"><font color="red">android产品研发（二十一）-->android中的UI优化</a>
<a href="http://blog.csdn.net/qq_23547831/article/details/51868496"><font color="red">android产品研发（二十二）-->android实用调试技巧</a>

<br>android产品研发之代码保存静态秘钥；
<br>android产品研发之AS打包apk，aar，jar包；
<br>android产品研发之Https请求；
<br>android产品研发之拦截App请求；
<br>android产品研发之定位服务；
<br>android产品研发之持续集成；
<br>android产品研发之MVP框架；
<br>android产品研发之常用框架；
<br>android产品研发之git使用；
<br>android产品研发之产品加固与加密；
<br>android产品研发之屏幕适配
<br>android产品研发之Fragment化
<br>android产品研发之基础组件SDK化
<br>android产品研发之React Native开发
<br>android产品研发之RxAndroid
<br>android产品研发之内存泄露情景与检测
<br>android产品研发之性能优化
<br>android产品研发之控件MD化

作为IT人员我还是比较强调做产品而不是做项目的，因为做项目都是跟着项目走许多东西做完了也就做完了，没有深入进去，没有持续的迭代与优化，相当于做一件事做了N遍，这样对个人很难有技能上的提高。持续的迭代一个产品不但能够在深度也能在广度上提高自己，如果可以的话强烈建议大家持续的迭代某一个产品。

这里多说几句，这里只是在技术上关于android产品研发的一些tip，在产品上我们同样的是需要有自己的思考。为了让App更好用，更好看，更简单，多多的站在用户的角度上思考，这不但是产品经理的任务，同样也是我们程序员需要做的。只有在和用户交流的时候，你才会发现，你觉得很好的东西用户可能根本不会用，有时候，专业术语导致用户完全不理解。按钮很明显但用户完全没看到，为什么?因为用户的注意力被其他功能给扰乱了，这些问题都是产品的复杂造成的。说这么多就是想跟大家说做产品需要的时候更多的需要从用户的角度考虑问题，而不是站在你的角度想用户的问题，有时候你所想的问题可能并不是用户想要的。
