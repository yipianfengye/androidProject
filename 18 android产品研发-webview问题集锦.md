上一篇文章中我们介绍了hybrid开发相关的知识。重点介绍了hybrid开发的概念，hybrid开发的作用，android中如何实现hybrid开发，android中实现hybrid开发的例子，以及产品开发中hybrid开发实践等，通过对以上这些概念的介绍我们对hybrid开发应该已经有了大概的了解，更多具体的内容可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->Hybrid开发</a>

本文中我们将介绍一下android中webview在使用过程中会遇到的一些问题。这些问题主要是webview在使用过程中我已经趟过的坑，希望通过这篇文章的介绍能够帮助大家更好的使用webview。

下面是本文主要介绍的一些知识点，后续使用过程中可能会有更新。

- webview的性能优化

- webview注入cookie信息

- webview退出activity异常

- webview中native与js交互

- webview下载文件

- 腾讯X5浏览服务

最近App中相当一部分的页面内容使用的是webview。而使用webview加载页面一个需要注意的地方就是性能，所以最近也研究了一下webview的性能优化问题。

**webview的性能问题**

在讲解webview的性能问题之前，我们先来了解一下android webview的缓存机制。

- Android WebView缓存机制

WebView中存在着两种缓存：网页数据缓存（网页数据，url等）、H5缓存（H5代码缓存数据）

不同的缓存数据会保存在不同的文件目录下，这里引用一下其他blog的说法：

> 当我们加载Html时候，会在我们data/应用package下生成database与cache两个文件夹:
我们请求的Url记录是保存在webviewCache.db里，而url的内容是保存在webviewCache文件夹下。
![这里写图片描述](http://img.blog.csdn.net/20160705155834541)


webview中也是可以设置缓存是否可用的，一般是通过WebSettings对象设置，下面我们就来看一下WebSettings对象的使用。

- android中webview组件有几个重要的方法

```
WebSettings webSettings = mWebView.getSettings();
webSettings.setJavaScriptEnabled(true);
webSettings.setLoadWithOverviewMode(true);
webSettings.setAllowFileAccess(false);
webSettings.setUseWideViewPort(false);
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
webSettings.setDatabaseEnabled(false);
webSettings.setAppCacheEnabled(false);
webSettings.setBlockNetworkImage(true);
// 设置WebView的Client
mWebView.setWebViewClient(new MWebViewClient(this));
// 设置可现实js的alert弹窗
mWebView.setWebChromeClient(new WebChromeClient());
```

可以看到我们可以使用WebSettings对象设置缓存是否可用，缓存DB是否可用等。我们需要首先确保这里设置了缓存可用，才可以继续设置使用何种缓存策略。

下面我们来看一下webview的五种缓存模式：
LOAD_CACHE_ONLY:  不使用网络，只读取本地缓存数据
LOAD_DEFAULT:  根据cache-control决定是否从网络上取数据。
LOAD_CACHE_NORMAL: API level 17中已经废弃, 从API level 11开始作用同LOAD_DEFAULT模式
LOAD_NO_CACHE: 不使用缓存，只从网络获取数据.
LOAD_CACHE_ELSE_NETWORK，只要本地有，无论是否过期，或者no-cache，都使用缓存中的数据。

- 几种缓存方式的实现

（1）使用LOAD_CACHE_ELSE_NETWORK缓存模式，这样需要在APP退出的时候清除webview缓存，但是这样做有一个弊端就是如果当前App已经是打开状态，网页内容有更新的话不会看到；

（2）使用LOAD_DEFAULT这种缓存方式，数据从缓存中获取还是从网络中获取根据H5页面的参数判断，这样做的好处是可以动态的处理更新内容；

- 设置缓存

```
mWebView.getSettings().setJavaScriptEnabled(true); 
     mWebView.getSettings().setRenderPriority(RenderPriority.HIGH);  mWebView.getSettings().setCacheMode(WebSettings.LOAD_DEFAULT);  //设置 缓存模式 
        // 开启 DOM storage API 功能 
        mWebView.getSettings().setDomStorageEnabled(true); 
        //开启 database storage API 功能 
        mWebView.getSettings().setDatabaseEnabled(true);  
        String cacheDirPath = getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME; 
//      String cacheDirPath = getCacheDir().getAbsolutePath()+Constant.APP_DB_DIRNAME; 
        Log.i(TAG, "cacheDirPath="+cacheDirPath); 
        //设置数据库缓存路径 
        mWebView.getSettings().setDatabasePath(cacheDirPath); 
        //设置  Application Caches 缓存目录 
        mWebView.getSettings().setAppCachePath(cacheDirPath); 
        //开启 Application Caches 功能 
        mWebView.getSettings().setAppCacheEnabled(true); 
```

- 退出App时清除缓存

```
//清理Webview缓存数据库 
        try { 
            deleteDatabase("webview.db");  
            deleteDatabase("webviewCache.db"); 
        } catch (Exception e) { 
            e.printStackTrace(); 
        } 
           
        //WebView 缓存文件 
        File appCacheDir = new File(getFilesDir().getAbsolutePath()+APP_CACAHE_DIRNAME); 
        Log.e(TAG, "appCacheDir path="+appCacheDir.getAbsolutePath()); 
           
        File webviewCacheDir = new File(getCacheDir().getAbsolutePath()+"/webviewCache"); 
        Log.e(TAG, "webviewCacheDir path="+webviewCacheDir.getAbsolutePath()); 
           
        //删除webview 缓存目录 
        if(webviewCacheDir.exists()){ 
            deleteFile(webviewCacheDir); 
        } 
        //删除webview 缓存 缓存目录 
        if(appCacheDir.exists()){ 
            deleteFile(appCacheDir); 
        }
```

- 其他的缓存策略

网页在加载的时候暂时不加载图片，当所有的HTML标签加载完成时在加载图片具体的做法如下初始化webview的时候设置不加载图片

```
webSettings.setBlockNetworkImage(true);
```

然后在html标签加载完成之后在加载图片内容:
```
@Override
    public void onPageFinished(WebView view, String url) {
        super.onPageFinished(view, url);
        mWebView.getSettings().setBlockNetworkImage(false);   
    }
```
O(∩_∩)O哈哈~，这样做的好处就是可以给人的感觉网页加载速度很快...

 - 将网页内容中需要的js，css引用文件保存在App本地

加载网页内容时，在加载完成html后替换页面内容引用的地址改为本地的资源文件地址，这样可以直接加载本地的资源文件加快资源的访问速度，目前主流的新闻客户端访问webview时多采用这种方式。

好吧，讲解完了webview的性能优化问题之后我们在讲解一下如何在H5页面种入Cookie信息。


**H5页面种入Cookie问题**

app中存在webview控件，既可以通过Native与js代码交互的方式实现信息的交互也可以通过cookie的方式与实现Native与H5端的交互，查询的好多资料各种各样的实现方式都有，最终不断尝试基本实现了需求，现说明一下最终的实现方式；

```
/**
     * 客户端将cookie种入H5页面中，H5页面可以通过js代码实现对native种入cookie信息的读取操作
     */
    public static void synCookies(Context context, String url, String cookie) {
        CookieSyncManager.createInstance(context);
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.setAcceptCookie(true);

        Uri uri = null;
        String domain = "";
        try {
            uri = Uri.parse(URLDecoder.decode(url, "utf-8"));
            domain = uri.getHost();
        } catch (UnsupportedEncodingException e) {
            e.printStackTrace();
        }
        String cookieStr = cookieManager.getCookie(url);
        // 判断token是否发生变化，发生变化的话则更新cookie
        L.i("cookieEquce:" + cookie.equals(H5Constant.TOKEN + "="));
        if (!TextUtils.isEmpty(cookieStr) && cookieStr.contains(cookie) && !cookie.equals(H5Constant.TOKEN + "=")) {
            return;
        }
        // 更新domain(不再从UserInfo中获取，更改为从UrlInfo中获取)
        if (!TextUtils.isEmpty(URLConfig.getUrlInfo().getB4Domain())) {
            domain = URLConfig.getUrlInfo().getB4Domain();
        }
        List<String> uaList = SysConfig.getSystemUa(context);
        String md = ";domain=" + domain;
        // 添加经纬度信息到Cookie中
        cookieManager.setCookie(url, "lat=" + Config.lat + md);
        cookieManager.setCookie(url, "lng=" + Config.lng + md);
        cookieManager.setCookie(url, cookie + ";" + md);
        if (uaList != null && uaList.size() > 0) {
            for (String coo : uaList) {
                cookieManager.setCookie(url, coo + md);
            }
        }

        CookieSyncManager.getInstance().sync();
    }
```

其中CookieManager是cookie的管理对象，主要实现对网页cookie的注入与清除等工作。注入字符串的形式是：key=value;domain=url的形式（其中url为需要注入cookie的url链接地址）

那么如何移除cookie呢？

```
/**
     * 移除cookie
     */
    public static void removeCookies(Context context) {
        CookieSyncManager.createInstance(context);
        CookieManager cookieManager = CookieManager.getInstance();
        cookieManager.removeSessionCookie();

        CookieSyncManager.getInstance().sync();
    }
```
一般情况下都是在打开H5页面的时候种入Cookie信息，然后在离开H5页面的时候清除cookie信息。当然了通过cookie实现native与js的交互只是实现信息交互的一种方式，我们还可以通过js与java代码相互调用的方式实现相互交互，文章的后面会有介绍。

而下面我们再来讲解一下activity退出时webview报错的问题。


**Activity退出时webview报错的问题**

前段时间在调试代码的时候，有一段关于webview的代码，即退出Fragment的时候清除webview，这时候在其他手机上是没有问题的，但是在三星Grand2中报错，而其报错信息是：

```
java.lang.Throwable: Error: WebView.destroy() called while still attached!
at android.webkit.WebViewClassic.destroy(WebViewClassic.java:4173)
at android.webkit.WebView.destroy(WebView.java:707)
at com.youyou.uuelectric.renter.UI.web.H5Fragment.onDestroyView(H5Fragment.java:202)
at android.support.v4.app.Fragment.performDestroyView(Fragment.java:2167)
at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1141)
at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1248)
at android.support.v4.app.FragmentManagerImpl.moveToState(FragmentManager.java:1230)
at android.support.v4.app.FragmentManagerImpl.dispatchDestroy(FragmentManager.java:2079)
at android.support.v4.app.FragmentController.dispatchDestroy(FragmentController.java:235)
 at android.support.v4.app.FragmentActivity.onDestroy(FragmentActivity.java:326)
at android.support.v7.app.AppCompatActivity.onDestroy(AppCompatActivity.java:161)
at com.youyou.uuelectric.renter.UI.base.BaseActivity.onDestroy(BaseActivity.java:136)
at android.app.Activity.performDestroy(Activity.java:5543)
at android.app.Instrumentation.callActivityOnDestroy(Instrumentation.java:1134)
at android.app.ActivityThread.performDestroyActivity(ActivityThread.java:3637)
at android.app.ActivityThread.handleDestroyActivity(ActivityThread.java:3672)
at android.app.ActivityThread.access$1300(ActivityThread.java:168)
at android.app.ActivityThread$H.handleMessage(ActivityThread.java:1382)
at android.os.Handler.dispatchMessage(Handler.java:99)
at android.os.Looper.loop(Looper.java:176)
at android.app.ActivityThread.main(ActivityThread.java:5493)
at java.lang.reflect.Method.invokeNative(Native Method)
at java.lang.reflect.Method.invoke(Method.java:525)
at com.android.internal.os.ZygoteInit$MethodAndArgsCaller.run(ZygoteInit.java:1225)
at com.android.internal.os.ZygoteInit.main(ZygoteInit.java:1041)
at dalvik.system.NativeStart.main(Native Method)
```

程序也没有异常退出之类的动作，清除webview的代码是这样写的：

```
@Override
    public void onDestroyView() {
        super.onDestroyView();
        mWebView.removeAllViews();
        mWebView.destroy();
    }
```

这个错误大概的意思是：当你结束webview的时候，webview还依附在其父控件之下，应当在调用webview.destory()方法之前接触他们之间的依附关系，那么既然错误提示信息已经很明显了，我们就根据错误信息尝试着首先执行webview父控件的清除工作，然后在执行webview控件的清除操作，所以代码中应该这样实现：

```
@Override
    public void onDestroyView() {
        super.onDestroyView();
        swipeRefreshLayout.removeView(mWebView);
        mWebView.removeAllViews();
        mWebView.destroy();
    }
```
这样经过修改之后特定机型上关于webview的错误就不在了。

**webview中实现Native与js相互调用**

上面我们介绍的通过cookie实现android native与H5的信息交互只是一种方式，我们也可以通过java代码与js代码直接相互调用的方式实现android native与H5信息的相互，这里简单的介绍一下使用方式

- native代码调用H5的js代码

（1）在H5页面中添加一个js函数

```
<script type="text/javascript">
function uu_click(clicked_id) {
    alert(clicked_id);
}
```

（2）在Native中通过java代码调用
若这时候H5页面已经被加载到webview中,则可以通过java代码直接调用js函数：

```
h5Fragment.mWebView.loadUrl("javascript:uu_click" + "('" + clickId + "')");
```

是的，没错就是这么简单，这样就实现了java代码对js代码的调用，但是需要指出的是这里无法调用含有返回值的js函数，这里也算是小小的问题吧，但是话说回来，一般也没有这种需要调用js代码并获得返回值的场景吧。

- js代码调用java函数

（1）首先在java端编写能够被js代码调用的java函数

- native方法的实现
```
/**
 * 自定义实现的native函数，可被js代码调用
 */
class JsInteration {
	...
	@JavascriptInterface
        public void toastMessage(String message) {
            Toast.makeText(getActivity(), message, Toast.LENGTH_LONG).show();
        }
	...
}
```

（2）在native中注入本地方法，供js调用；

```
mWebView.addJavascriptInterface(new JsInteration(), "control");
```

（3）在js代码中调用java代码：

```
function reply_click(clicked_id {
	window.control.toastMessage(clicked_id)
}
```
以上就是webview中使用js代码与java代码相互调用的简单例子。


**webview下载文件**

在使用webview开发中还遇到一个问题，app中webview中存在下载链接，但是在手机浏览器中点击下载是没有问题的，在webview中怎么都不好使。查询了好久，原来是因为WebView默认没有开启文件下载的功能，如果要实现文件下载的功能，需要设置WebView的DownloadListener，通过实现自己的DownloadListener来实现文件的下载。

- 设置webview的setDownloadListener方法

```
mWebView.setDownloadListener(new DownloadListener() {
            @Override
            public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype, long contentLength) {
                L.i("tag", "this is a message!!!");
            }
        });
```

- 重写onDownloadStart回调方法，实现下载文件的逻辑

比如在浏览器中实现下载：

```
@Override  
        public void onDownloadStart(String url, String userAgent, String contentDisposition, String mimetype,  
                                    long contentLength) {  
            Uri uri = Uri.parse(url);  
            Intent intent = new Intent(Intent.ACTION_VIEW, uri);  
            startActivity(intent);  
        }  
```
这样我们的webview中如果包含了下载链接就可以通过打开浏览器的方式实现下载了。

注：个人感觉webview没有默认实现下载链接的功能更多的可能是考虑权限，安全方面的问题。

**腾讯X5浏览服务**

由于不同的手机版本问题，各个厂商的定制化操作，webview在不同的手机上表现有很大的不同，所以webview的适配工作在android机上显得比较重要，这里我们就简单的介绍一下腾讯的X5浏览服务，其相当于android上webview的第三方框架。

这里先暂时看一下腾讯X5服务的官方介绍，其官网是：<a href="http://x5.tencent.com/">腾讯浏览服务</a>，然后我们看一下X5浏览服务官网对其的描述。

> 腾讯浏览服务由QQ浏览器团队出品，致力于优化移动端webview体验的整套解决方案，使用QQ浏览器X5内核SDK和X5云端服务，解决移动端webview使用过程中出现的一切问题，优化用户的浏览体验，同时腾讯还将持续提供后续的更新和优化，为开发者提供最新最优秀的功能和服务。

> X5SDK是通过调用微信/手机QQ/空间的X5内核，解决系统webview兼容性差、加载速度慢、功能缺陷等问题，开发接入便捷，大小只有253K，仅需几行代码，即可解决一切令开发者们头疼的问题，为用户提供最优秀的浏览体验。
> 同时，QQ浏览器团队还将持续更新和优化X5内核，持续优化功能，并保证兼容各种web新特性。

- 腾讯X5浏览服务的优势：

1) 速度快：相比系统webView的网页加载速度有近30%的提升。
2) 省流量：云端优化技术使流量节省20%
3) 更安全：24小时安全问题解决机制
4) 更稳定：经过亿级用户的使用考验，CRASH率0.15%
5) 集成强大的视频播放器，支持各种视频格式直接打开
6) 适屏排版、字体设置等浏览增强功能的提供
7) Html5更完整支持。
8) 无系统内核的碎片化问题，更少的兼容性问题

那么现在那些App具体接入了X5服务了呢？
![这里写图片描述](http://img.blog.csdn.net/20160704212532338)

可以发现已经有不少的牛擦的App接入了X5服务（当然主要是腾讯系的，原因你懂的），其相对于原生的webview的最大优势是做了各种型号手机的适配工作，而且其也添加了不少有特色的小功能，对于使用webview开发App的同学而言，可以考虑一下集成X5服务哈。

**总结：**

- webview的有一定的性能问题，可以设置不同的缓存策略提高webview的加载速率

- webview与native可以通过cookie的方式实现信息交互

- webview下载文件，需要重写自己的native的DownloadListener类，以及其回调方法

- 实现Native与H5端信息的交互既可以使用cookie的方式也可以通过java代码与js代码实现

- 原生的webview在不同手机上可能存在不同的表现，建议可以使用腾讯的X5浏览服务屏蔽的不同手机的差别，添加了不少特色的小功能

- 其他关于webview的问题集锦，可参考：<a href="http://blog.csdn.net/t12x3456/article/details/13769731/">Android WebView常见问题及解决方案汇总</a>，总结的很全面哈

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
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->hybrid开发</a>

