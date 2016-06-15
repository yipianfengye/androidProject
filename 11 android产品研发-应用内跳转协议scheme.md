上一篇文章中我们讲解了在Android App的实际开发中，尽量不在静态变量、全局变量中保存数据，这是因为App的进程可能是不安全的，在部分手机中其有可能被系统杀死，从而造成静态全局变量重新初始化。而这时候App当前页面的Activity还会被保存在内存中，从而造成App并没有被重启的假象，但是这只是显示的页面没有被杀死，而进程实际上是被重新启动了的。这时候在使用已被重新初始化的静态变量就会发生一些不可预知的错误，具体关于不在静态变量中保存数据的问题，可以参考这里的：<a href="http://blog.csdn.net/qq_23547831/article/details/51655330"> android产品研发（十）-->不使用静态变量保存数据</a>

而本文讲解的是一种App内页面跳转协议，这里的跳转包括应用内跳转、H5与Native跳转，服务器通知客户端如何跳转等。

在讲解应用内跳转协议之前我们先讲解一下H5与Native相互跳转的相关知识点。现在越来越多的App采用了Native + H5方式开发，其中Native与H5页面如何交互？google提供了一个公共的方式：js与native互调，即js可以调用Native方法，Native同样也可以调用js方法；

**但是这种交互方式存在着不少问题：**
1、Java 调用 js 里面的函数、效率并不是很高、估计要200ms左右吧、做交互性很强的事情、这种速度很难让人接受、而js去调Java的方法、速度很快、50ms左右、所以尽量用js调用Java方法
2、Java 调用 js 的函数、没有返回值、调用了就控制不到了
3、Js 调用 Java 的方法、返回值如果是字符串、你会发现这个字符串是 native 的、转成 locale 的才能正常使用、使用 toLocaleString() 函数就可以了、不过这个函数的速度并不快、转化的字符串如果很多、将会很耗费时间
4、网页中尽量不要使用jQuery、执行起来需要5-6秒、最好使用原生的js写业务脚本、以提升加载速度、改善用户体验
5、android4.2以下的系统存在着webview的js对象注入漏洞...（不清楚的可以google）

基于这种种的原因，我们并未采用这种方式用于Native与webview交互，而是采用scheme + cookie的方式；

这里的scheme是一种页面内跳转协议，主要用于支持一下几种场景：

- 服务器下发跳转路径，客户端根据服务器下发跳转路径跳转相应的页面；

- H5页面点击锚点，根据锚点具体跳转路径App端跳转具体的页面；

- App端收到服务器端下发的PUSH通知栏消息，根据消息的点击跳转路径跳转相关页面

下面我将简单介绍一下scheme的基本概念以及以上三种场景下scheme的具体应用。

## URL scheme 概述

### URL scheme 的作用

客户端应用可以向操作系统注册一个 URL scheme，该 scheme 用于从浏览器或其他应用中启动本应用。通过指定的 URL 字段，可以让应用在被调起后直接打开某些特定页面，比如车辆详情页、订单详情页、消息通知页、促销广告页等等。也可以执行某些指定动作，如订单支付等。也可以在应用内通过 html 页来直接调用显示 app 内的某个页面。

### URL scheme 的格式

客户端自定义的 URL 作为从一个应用调用另一个的基础，遵循 RFC 1808 (Relative Uniform Resource Locators) 标准。这跟我们常见的网页内容 URL 格式一样。

一个普通的 URL 分为几个部分，`scheme`、`host`、`relativePath`、`query`。

比如：`http://www.baidu.com/s?rsv_bp=1&rsv_spt=1&wd=NSurl&inputT=2709`，这个URL中，`scheme` 为 `http`，`host` 为 `www.baidu.com`，`relativePath` 为 `/s`，`query` 为 `rsv_bp=1&rsv_spt=1&wd=NSurl&inputT=2709`。

一个应用中使用的 URL 例子（该 URL 会调起车辆详情页）：`uumobile://mobile/carDetail?car_id=123456`，其中 `scheme` 为 `uumobile`，`host` 为 `mobile`，`relativePath` 为 `/carDetail`，`query` 为 `car_id=123456`。


###Scheme定义Activity
1）在androidmanifest.xml中定义scheme

```
<!-- scheme协议 -->
        <activity
            android:name=".UI.translate.NativeAppActivity"
            android:label="@string/app_name">

            <!-- 要想在别的App上能成功调起App，必须添加intent过滤器 -->
            <intent-filter>

                <!-- 协议部分，随便设置 -->
                <data android:scheme="uumobile" />
                <!-- 下面这几行也必须得设置 -->
                <category android:name="android.intent.category.DEFAULT" />
                <category android:name="android.intent.category.BROWSABLE" />

                <action android:name="android.intent.action.VIEW" />
            </intent-filter>
        </activity>
```
这样我们便定义了能够接受scheme请求的activity实例，当网页或者是android代码发送这种规则scheme的请求的时候就能够吊起NativeAppActivity了。

2）当然就是实现NativeAppActivity

```
/**
 * Created by admin
 */
public class NativeAppActivity extends Activity{
    public String tag = "NativeAppActivity";
    public Activity mContext = null;

    public void onCreate(Bundle b)
    {
        super.onCreate(b);
        mContext = this;
        Uri uri = getIntent().getData();
        if (uri != null)
        {
            List<String> pathSegments = uri.getPathSegments();
            String uriQuery = uri.getQuery();
            Intent intent；
            if (pathSegments != null && pathSegments.size() > 0) {
                // 解析SCHEME
                if (someif) {
                  dosomething();
                }
                else {
                    // 若解析不到SCHEME，则关闭NativeAppActivity；
                    finish();
                }
            } else {
                finish();
            }
        } else {
            finish();
        }
    }

}
```
NativeAppActivity这个类中主要用于实现对scheme的解析，然后做出相应的动作，比如请求scheme跳转登录页面，我们可以这样定义

```
uumobile：//appname/gotoLogin
```
然后我们解析出scheme如果是这样的结构就跳转登录页面。。。

这里简单说一下，我们可以通过Intent对象获取调用的scheme的host等信息

```
this.getIntent().getScheme();//获得Scheme名称  
this.getIntent().getDataString();//获得Uri全部路径 
```

3）通过服务器下发跳转路径跳转相应页面

```
startActivity(new Intent(Intent.ACTION_VIEW, Uri.parse("uumobile://yongche/123123123")));
```

这里的"uumobile://yongche/123123123"就是服务器下发的跳转路径，当我们执行startActivity的时候就会调起NativeAppActivity，然后我们通过在NativeAppActivity解析scheme的内容，跳转相应的页面。

4）通过在H5页面的锚点跳转相应的页面

```
@Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        //解析scheme
        if (url.indexOf(H5Constant.SCHEME) != -1) {
            try {
                Uri uri = Uri.parse(url);
                String[] urlSplit = url.split("\\?");
                Map<String, String> queryMap = new HashMap<String, String>();
                String h5Url = null;
                if (urlSplit.length == 2) {
                    queryMap = H5Constant.parseUriQuery(urlSplit[1]);
                    h5Url = queryMap.get(H5Constant.MURL);
                }
                // 跳转NativeAppActivity解析
                {
                    // 若设置刷新，则刷新页面
                    if (queryMap.containsKey(H5Constant.RELOADPRE) && "1".equals(queryMap.get(H5Constant.RELOADPRE))) {
                        h5Fragment.isNeedFlushPreH5 = true;
                    }
                    Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                    h5Activity.startActivityForResult(intent, H5Constant.h5RequestCode);
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
            return true;
        }
        // 打电话
        else if (url.indexOf("tel://") != -1) {
            final String number = url.substring("tel://".length());
            Config.callPhoneByNumber(h5Activity, number);
            return true;
        } else if (url.indexOf("tel:") != -1) {
            final String number = url.substring("tel:".length());
            Config.callPhoneByNumber(h5Activity, number);
            return true;
        }
        // 其他跳转方式
        else {
            view.loadUrl(url);
            //如果不需要其他对点击链接事件的处理返回true，否则返回false
            return false;
        }
    }
```
可以发现我们为Webview设置了WebViewClient，并重写了WebViewClient的shouldOverrideUrlLoading方法，然后我们解析锚点的url，并根据解析的内容调起NativeAppActivity的scheme Activity，然后在NativeAppActivity中解析scheme的内容并跳转相应的页面。

5）根据服务器下发通知栏消息，App跳转相应的页面

```
public class NotificationActivity extends Activity {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        L.i("接收到通知点击事件...");
        Intent realIntent = getIntent().getParcelableExtra(NotifeConstant.REAL_INTENT);
        // 解析scheme并跳转
        gotoRealScheme(this, realIntent);
    }


    /**
     * notification中跳转SCHEME，根据有效时间判断跳转URL地址
     *  跳转之后更具网络请求判断用户当前状态
     */
    private void gotoRealScheme(Context context, Intent realIntent) {
        if (realIntent == null || context == null) {
            finish();
            return;
        }
        try {
            L.i("开始解析通知中的参数...");
            long startShowTime = realIntent.getLongExtra(NotifeConstant.START_SHOW_TIME, 0);
            // 有效期时间，单位:s（秒）
            long validTime = realIntent.getLongExtra(NotifeConstant.VALID_TIME, 0);
            long currentTime = System.currentTimeMillis();
            String validActionUrl = realIntent.getStringExtra(NotifeConstant.VALID_ACTION_URL);
            String invalidActionUrl = realIntent.getStringExtra(NotifeConstant.INVALID_ACTION_URL);
            Intent schemeIntent;
            L.i("开始根据URL构建Intent对象...");
            if ((currentTime - startShowTime) / 1000L <= validTime) {
                schemeIntent = H5Constant.buildSchemeFromUrl(validActionUrl);
            } else {
                schemeIntent = H5Constant.buildSchemeFromUrl(invalidActionUrl);
            }
            if (schemeIntent != null) {
                // 设置当前页面为通知栏打开
                Config.isNotificationOpen = true;
                context.startActivity(schemeIntent);
                finish();
                //对通知栏点击事件统计
                MobclickAgent.onEvent(context, UMCountConstant.PUSH_NOTIFICATION_CLICK);
            } else {
                finish();
            }
        } catch (Exception e) {
            // 异常情况下退出当前Activity
            finish();
        }
    }
}
```
服务器下发的所有的通知都是先跳转这里的NotificationActivity，然后在这里执行跳转其他Activity的逻辑，而这里的H5Constant的buildSchemeFromUrl方法就是构造跳转页面Intent对象的，我们可以看一buildSchemeFromUrl方法的具体实现：

```
/**
     * 从scheme的url中构建出Intent，用于界面跳转
     *
     * @param url
     * @return
     */
    public static Intent buildSchemeFromUrl(String url) {
        if (url != null && url.indexOf(H5Constant.SCHEME) != -1) {
            Uri uri = Uri.parse(url);
            String[] urlSplit = url.split("\\?");
            Map<String, String> queryMap = new HashMap<String, String>();
            String h5Url = null;
            if (urlSplit.length == 2) {
                queryMap = H5Constant.parseUriQuery(urlSplit[1]);
                h5Url = queryMap.get(H5Constant.MURL);
            }
            Intent intent = new Intent(Intent.ACTION_VIEW, uri);
            if (!TextUtils.isEmpty(h5Url)) {
                intent.putExtra(H5Constant.MURL, h5Url);
            }
            return intent;
        }
        return null;
    }
```
这样我们就搞构造除了跳转NativeAppActivity的Intent对象，并将scheme字符串传递给了NativeAppActivity，这样在NativeAppActivity中就可以解析scheme字符串并执行相应的跳转逻辑了。



**总结：**
android中的scheme是一种非常好的实现机制，通过定义自己的scheme协议，可以非常方便跳转app中的各个页面；
通过scheme协议，服务器可以定制化告诉App跳转那个页面，可以通过通知栏消息定制化跳转页面，可以通过H5页面跳转页面等。


另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261">android产品研发（五）-->多渠道打包</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51581491">android产品研发（六）-->Apk混淆</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51587927"> android产品研发（七）-->Apk热修复</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51598041">android产品研发（八）-->App数据统计</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51612429">android产品研发（九）-->App网络传输协议</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51655330">android产品研发（十）-->不使用静态变量保存数据</a>
