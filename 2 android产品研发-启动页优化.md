上一篇文章中我们介绍加单说明了一下android的编码规范，这里我是强烈建议大家在团队合作中约定编码规范的，哪怕是一个并不是十分规范的规范总比没有规范好得多，尤其是团队产品的研发，对产品的持续迭代过程中你会越发的意识到编码规范对产品迭代的好处，当然了，这里并不是要求大家一定按照文中给出的编码规范作为团队中使用的编码规范，而是希望大家在团队合作中能够约定出自身的编码规范，哪怕其并不是十分的规范。

而这片文章中我们主要介绍一下启动页的优化，关于启动页的优化是UE方面的内容了，一个很慢的启动页很容易让用户觉得受不了，进而“逃离”App的，所以若想产品有更好的用户体验，做一些启动页的优化是一个不错的选择。这里我们简单介绍一下我在实践中对启动页是如何优化的。

 公司产品使用的数据统计产品是友盟，最近产品经理在看友盟数据的时候，发现App在启动页跳出率很高，启动页的平均启动时间为2.8s，如下图：
 ![image](http://img.blog.csdn.net/20160531110002721)

纳尼，2.8s才启动App，这是杂么回事啊，优化吧...

没办法，2.8s启动启动页时间确实有点长，而且用户在首页的跳出率也确实有点高（这里普及一下跳出率的概念：用户从当前页面离开应用的情况，说白了就是启动页的启动时间过长，不少用户都等不了了，直接从启动页离开App了）。

那么在优化启动也的时候我们应该逐步的分析启动页的逻辑，熟悉了启动页的执行逻辑才能更好的优化启动页的加载速度。


**- 启动页面网络请求优化**

（1）我们在启动页做了什么？
我们的启动页面主要用于展示启动页面，加载网络配置信息；
具体而言：

- 加载App中h5页面的url地址

- 加载请求用户信息

- 加载首页地图页面需要展示的图标

- 延时三秒钟跳转首页面，这时候无论是否加载到前面的网络请求信息，都会跳转到首页面

（2）那些可以优化的地方？
这样看来由于我们在启动页面有网络请求，所以主动的延时3秒钟跳转首页，所以造成了我们的启动页面加载过慢，所以我们首要的优化地方就是在网络请求与延时跳转这方面。

（3）如何优化？
现在的逻辑是无论配置信息是否拉取成功都需要在3秒钟的时候跳转主页面，可以这样考虑，若这三个配置信息拉取的时间小于3秒钟其实不必要等到3秒钟在跳转主界面；思路已经出来了，剩下的就是具体实现了。。

（4）启动页面网络请求优化的具体实践

```
timer = new Timer();
            timerTask = new TimerTask() {
                @Override
                public void run() {
                    if (isGoToComplete) {
                        L.i("等待2s完成，取消timer任务...");
                        cancelTimer();
                        return;
                    }
                    if (isLoadStartQueryComplete && isLoadUserInfoComplete) {
                        // 已经显示过引导图
                        L.i("StartActivity中网络请求完成，执行跳转逻辑.....");
                        handler.sendEmptyMessage(eventWhat);
                        L.i("StartActivity中网络请求完成，取消timer任务...");
                        cancelTimer();
                    }
                }
            };
            timer.schedule(timerTask, 0, 100);
```

在这里我们创建一个timer任务，每隔100ms执行一次，判断这两个拉取网络配置信息的异步消息是否已经完成，若已经完成的话则发送handler消息跳转主界面，若2秒钟之内都没有完成拉取任务，则直接跳转同时取消timer任务。

简单来说就是启动页面最长的等待时间为2秒钟，若两秒钟之内启动页面的网络请求信息还没有完成也不在等待直接跳转主页面，而若在2秒钟之内启动页面的网络请求执行完毕，则直接跳转不必等到2秒钟之后再跳转，这样我们启动页面就根据网络请求的情况大大缩短了加载时间。

**- 启动页面特效优化**

现如今几乎所有的安卓app在启动的时候都会有一个类似于开机画面的东西，往往是一个开机界面，上面写着这个应用程序的提示文字，动画等等，并且这个开解界面往往实现了渐变，loading等效果，这样启动页面的耗时比较长的话，用户也不会觉得时间很长，难以忍受，当然了这种方式只是在视觉效果上造成“启动页面加载速度很快”的效果的。

这里我们简单的实现启动页面的动画效果的实现：

```
//渐变展示启动屏
        AlphaAnimation alphaAnimation = new AlphaAnimation(0.3f,1.0f);
        alphaAnimation.setDuration(2000);
        contentView.startAnimation(alphaAnimation);
        alphaAnimation.setAnimationListener(new AnimationListener() {
            @Override
            public void onAnimationEnd(Animation arg0) {
                L.i("启动页面动画执行完毕...");
            }
            
            @Override
            public void onAnimationRepeat(Animation animation) {
				L.i("启动页面动画重复执行...");
			}
			
            @Override
            public void onAnimationStart(Animation animation) {
				L.i("启动页面动画开始执行...")
			}
                                                                          
        });
```
而这里的contentView就是我们启动页Activity的主View，所以这里的执行效果就是我们的启动页Activity的界面的透明度逐渐增加，这样在用户看来由于有了一层loading效果，加载速度显得很快。


**- 启动页面无黑屏**

这里同时也介绍一下android app启动页面黑屏的问题，android开发app启动时若没有做特殊处理的话，会出现一瞬间的黑屏现象，网上针对这种情况一般有两种解决方式：

（1）App启动页面为图片或者是drawable文件

```
<item name="android:windowBackground">@drawable/start_background</item>
```
设置设置整个App的style文件，设置android:windowBackground属性为图片或者是drawable文件，这样App在启动过程中就是显示的是启动页面的图片或者是是drawable文件;

（2）App启动页面为Layout文件

这时候无法为App的style设置background，退而求其次主流的方式是设置白色背景

```
<item name="android:windowBackground">@color/c10</item>
```
而这里的@color/c10就是我们定义的白色色值，这样经过处理之后App启动时就不会出现黑屏的效果了。

**- Application启动速度优化**
由于我们的应用启动过程首先会执行Application的创建于执行其生命周期方法（onCreate方法），具体可参见我的android应用进程启动流程：<a href="http://blog.csdn.net/qq_23547831/article/details/51119333"> android源码解析之（十一）-->应用进程启动流程</a>

所以提高app的启动速度，加快Application的执行时间也是一个很重要的方面，这里我暂时总结了几条原则：

- 尽量不将一些业务逻辑放于Application中；

- Application尽量不以静态变量的方式保存应用数据；

- 若App的大小不是特别大无需使用dex分包方案；

- 在Application中关于文件，数据库的操作尽量

**- 启动页面屏蔽返回按键**

一般的我们的App中都会在启动页面执行一些网络操作，初始化配置等，所以这时候我们是不希望用户通过按下返回按键退出App，因而我们需要在启动页中屏蔽返回按键，这里简单的介绍一下具体的实现：

```
/**
     * startActivity屏蔽物理返回按钮
     *
     * @param keyCode
     * @param event
     * @return
     */
    @Override
    public boolean onKeyDown(int keyCode, KeyEvent event) {
        if (keyCode == KeyEvent.KEYCODE_BACK) {
            return true;
        }
        return super.onKeyDown(keyCode, event);
    }
```
这里我们重写启动页面Activity的onKeyDown方法，首先判断用户按下的是否是返回按键，若是的话则直接返回true，屏蔽返回按键的后续执行逻辑。关于返回按键的执行流程，可参考我的android系统返回按键执行流程：<a href="http://blog.csdn.net/qq_23547831/article/details/51513771"> android源码解析（二十九）-->应用程序返回按键执行流程</a>

以上就是个人总结的启动页中一些要点或者是需要注意的地方，个人能力不足有不对的地方欢迎指正。

另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>

