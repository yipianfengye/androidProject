上一篇文章中我们介绍了android开发中经常会涉及到但又常常被忽视掉的开发者模式。主要讲解了包括如何打开手机的开发者模式，开发者模式中各个菜单的意义和作用，如何清除手机App数据，以及清除手机App数据具体清除那些数据等知识点，具体关于android中开发者模式的知识，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51809497"> android产品研发（十六）-->开发者选项</a>

本文将介绍android中hybird开发相关的知识点。hybird开发实际上是混合开发的意思，这里的混合是H5开发与Native开发混合的意思。下面的文章中我们将逐个介绍一下hybird开发的概念、hybird开发的优势、android中如何实现hybird开发、简单的hybird开发的例子，以及在产品实践中对hybird开发的应用，希望通过本篇文章的介绍让您能够对android中的hybird开发有一个基本的认识。

**一：hybird开发的概念**

在具体介绍hybird开发之前，我们先看一下什么是hybird开发，在这里我们先引用一下百度百科中对hybird开发的定义：

> Hybrid App（混合模式移动应用）是指介于web-app、native-app这两者之间的app，兼具“Native App良好用户交互体验的优势”和“Web App跨平台开发的优势”。

从定义中我们可以看到hybird开发其实就是在App开发过程中既使用到了web开发技术也使用到了native开发技术，通过这两种技术混合实现的App就是我们通常说的hybird app，而通过这两种技术混合开发就是hybird开发。

好吧，我们已经知道hybird开发的具体含义，那么一个问题就产生了，既然我们已经有了native开发了为何还需要hybird开发呢？它有什么好处么？答案是肯定的，下面我们就来看一下为何需要hybird开发方式。

**二：为何需要hybird开发**

下面我们简单看一下Native开发中存在的弊端以及使用Hybird开发方式的好处，通过对比你就能知道了hybird开发的优势，当然了，这里不是推崇使用hybird开发方式，native也有native开发的优势，hybird开发也有hybird开发的劣势，这里只是简单的看一下hybird相对于native开发的优势。

- 使用Native开发的方式人员要求高，只是一个简单的功能就需要IOS程序员和Android程序员各自完成；

- 使用Native开发的方式版本迭代周期慢，每次完成版本升级之后都需要上传到App Store并审核，升级，重新安装等，升级成本高；

- 使用Hybird开发的方式简单方便，同一套代码既可以在IOS平台使用，也可以在android平台使用，提高了开发效率与代码的可维护性；

- 使用Hybird开发的方式升级简单方便，只需要服务器端升级一下就好了，对用户而言完全是透明了，免去了Native升级中的种种不便；

通过对比可以发现hybird开发方式现对于native实现主要的优势就是更新版本快，代码维护方便，当然了这两个优点也是我们推崇使用hybird开发app的主要因素。知道了hybird开发的好处之后，我们如何在android中实现hybird开发呢?下面我们就将介绍这个问题。

**三：android中如何实现Bybird开发**

其实在android开发中使用hybird模式开发app，也是有两种方案的：

- 使用第三方hybird框架

- 自己使用webview加载

通过这两种方案实现Hybird开发各有利弊，具体如下：

- 使用PhoneGap、AppCan之类的第三方框架，其实现的原理是以WebView作为用户界面层，以Javascript作为基本逻辑，以及和中间件通讯，再由中间件访问底层API的方式，进行应用开发。相当于为我们封装了webview与相应的native组件；

- 使用webview控件加载H5网页的内容，其中客户端的webview只是作为一个加载H5页面的壳子，具体的实现效果是由H5实现的，这个需要Native程序员和H5程序员一起合作完成；

- 使用第三方框架的方式的好处是许多功能已经被集成好了，只需要简单的调用即可，但是这种方式集成度高，不容易定制化处理，而且性能上也是一个打的问题；

- 使用webview加载H5页面，定制化程度高，问题可控，但是相对与第三方框架集成度不够高，但是其已经可以满足我们日常的开发功能需要了，目前还是比较推荐使用这种方式实现Hybird开发；

下面我们就看一下如何在android系统中通过webview实现对H5页面的加载操作。

**四：Hybird开发简单实现**

- 在AndroidManifest.xml中定义网络请求权限

```
<uses-permission android:name="android.permission.INTERNET"/>
```
注意这个权限是必须的，因为加载webview页面一般而言经常是网络上的H5页面，这时候的网络请求权限就是必须的了，好多时候测试webview加载网络H5页面失败，找了半天不知道是什么原因，最后才发现是网络权限没有添加...

- 在Layout布局文件中定义Webview控件

```
<WebView 
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:id="@+id/webView"
    />
```
这里的WebView控件就是android原生的webview控件了，其和普通的android控件的使用没有什么不同都是在布局文件中定义，然后在Activity代码中获取并执行初始化操作等等。

- 在代码中获取Webview控件加载本地或者网络H5资源

加载本地H5页面
```
/**
 * 加载本地H5资源文件
 */
webView = (WebView) findViewById(R.id.webView);
webView.loadUrl("file:///android_asset/example.html");
```

加载网络H5页面

```
/**
 * 加载网络H5资源
 */
webView = (WebView) findViewById(R.id.webView);
webView.loadUrl("http://baidu.com");
```
可以发现在获取到webview组件之后直接执行一个loadUrl方法传入一个url地址就可以了，这样在activity页面中就可以展示出webview页面了，契合普通的网页效果没什么不同，这里需要说明的是，webview不但能够加载网页地址，同样的也可以加载html代码，本地html资源等等，相对来说功能还是很强大的。

当然了以上只是最最简单的webview使用的例子，下面我们可以为我们的webview对象设置各种参数：

- 为Webview控件设置参数

```
WebSettings webSettings = h5Fragment.mWebView.getSettings();
        webSettings.setJavaScriptEnabled(true);
        webSettings.setLoadWithOverviewMode(true);
        webSettings.setAllowFileAccess(false);
        webSettings.setUseWideViewPort(false);
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
        webSettings.setDatabaseEnabled(false);
        webSettings.setAppCacheEnabled(false);
        webSettings.setBlockNetworkImage(true);
```
这里的WebSettings就是webview的设置参数对象，我们是通过它为webview设置各种参数值的，见名知意，看见名字我们就知道各个set方法的意思了。比如设置webview中的html页面js代码是否可用，是否可以访问系统文件，H5缓存是否可用，是否立即加载网页图片等等。

- 为Webview控件设置WebChromeClient

WebChromeClient对象是webview的关于页面效果回调方法的实现对象，主要用于实现webview页面上一些效果的回调，我们可以看一下其中实现的一些回调方法：

```
/**
 * 自定义实现WebChromeClient对象
 */
public class MWebChromeClient extends WebChromeClient{

	/**
	 * 当webview加载进度变化时回调该方法
	 */
    @Override
    public void onProgressChanged(WebView view, int newProgress) {
        super.onProgressChanged(view, newProgress);
    }

	/**
	 * 当加载到H5页面title的时候回调该方法
	 */
    @Override
    public void onReceivedTitle(WebView view, String title) {
        super.onReceivedTitle(view, title);
    }

	/**
	 * 当接收到icon的时候回调该方法
	 */
    @Override
    public void onReceivedIcon(WebView view, Bitmap icon) {
        super.onReceivedIcon(view, icon);
    }

	/**
	 * 当H5页面调用js的Alert方法的时候回调该方法
	 */
    @Override
    public boolean onJsAlert(WebView view, String url, String message, JsResult result) {
        return super.onJsAlert(view, url, message, result);
    }

	/**
	 * 当H5页面调用js的Confirm方法的时候回调该方法
	 */
    @Override
    public boolean onJsConfirm(WebView view, String url, String message, JsResult result) {
        return super.onJsConfirm(view, url, message, result);
    }

	/**
	 * 当H5页面调用js的Prompt方法的时候回调该方法
	 */
    @Override
    public boolean onJsPrompt(WebView view, String url, String message, String defaultValue, JsPromptResult result) {
        return super.onJsPrompt(view, url, message, defaultValue, result);
    }
}
```
上面的WebChromeClient中我们重写了其中的几个字方法，我们已经在方法中添加了注释标明了各个方法的调用时机，而且通过方法名我们也不难发现各个方法的具体作用，这里就不在具体的介绍了。

- 为Webview主要设置WebviewClient

```
/**
 * 自定义实现WebViewClient类
 */
public class MWebViewClient extends WebViewClient {

	/**
	 * 在webview加载URL的时候可以截获这个动作, 这里主要说它的返回值的问题：
	 *	1、返回: return true;  webview处理url是根据程序来执行的。 
	 *	2、返回: return false; webview处理url是在webview内部执行。 
	 */
    @Override
    public boolean shouldOverrideUrlLoading(WebView view, String url) {
        
    }

	/**
	 * 在webview开始加载页面的时候回调该方法
	 */
    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        super.onPageStarted(view, url, favicon);

    }

	/**
	 * 在webview加载页面结束的时候回调该方法
	 */
    @Override
    public void onPageFinished(WebView view, String url) {
        super.onPageFinished(view, url);
    }

	/**
	 * 加载页面失败的时候回调该方法
	 */
    // 该方法为android23中新添加的API，android23中会执行该方法
    @TargetApi(21)
    @Override
    public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
        
    }

	/**
	 * 加载页面失败的时候回调该方法
	 */
    /**
     * 在android23中改方法被onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) 替代
     * 因此在android23中执行替代方法
     * 在android23之前执行该方法
     * @param view
     * @param errorCode
     * @param description
     * @param failingUrl
     */
    @Override
    public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
        
    }
}
```
这里我们只是暂时看一下WebViewClient中的几个比较重要的方法，shouldOverrideUrlLoading方法，onPageStarted方法，onPageFinished方法，onReceivedError方法等，相关的方法说明已经有注释了，这里就不在做过多的说明了。好了介绍完了相关的API之后我们来看一下我们在产品中关于Hybird开发的实践。

- 友友用车中关于Hybird开发的实践

Hybird这么高逼格的东西友友用车怎么能不涉及呢？在我们的产品开发中也使用到了Webview，并封装了自己的Webview库，下面我们就看一下友友用车中关于Hybird开发的实践。

（1）定义H5Activity类，用于展示H5页面

```
/**
 * 自定义实现的H5Activity类，主要用于在页面中展示H5页面，整个Activity只有一个Fragment控件
 */
public class H5Activity extends BaseActivity {

    public H5Fragment h5Fragment = null;

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_h5);
        h5Fragment = new H5Fragment();
        getSupportFragmentManager().beginTransaction().replace(R.id.mfl_content_container, h5Fragment).commit();
    }
}
```

（2）在H5Fragment中具体实现对H5页面的加载操作

```
/**
 * 具体实现H5页面加载Fragment，只有一个Webview控件
 */
public class H5Fragment extends BaseFragment implements SwipeRefreshLayout.OnRefreshListener {

    @BindView(R.id.sswipeRefreshLayout)
    public SwipeRefreshLayout swipeRefreshLayout;
    /**
     * H5页面 WebView
     */
    @BindView(R.id.mwebview)
    public WebView mWebView = null;
    @BindView(R.id.rl)
    public RelativeLayout rl;
    /**
     * 页面title
     */
    public String title = "";
    /**
     * 页面当前URL
     */
    public String currentUrl = "";
    /**
     * 判断网页是否加载成功
     */
    public boolean isSuccess = true;
    /**
     * 判断前一页H5是否需要刷新
     */
    public boolean isNeedFlushPreH5 = false;

    private BasePayFragmentUtils payFragmentUtils;

    public static final String KEY_DIALOG_WEB_VIEW = "dialog_webView";
    /**
     * 是否是弹窗中的WebView
     */
    private boolean isDialogWebView = false;

    View.OnClickListener errorOnClickListener = new View.OnClickListener() {
        @Override
        public void onClick(View view) {
            mProgressLayout.showLoading();
            isSuccess = true;
            reflush();
        }
    };

    @Override
    public void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        payFragmentUtils = new BasePayFragmentUtils(this, BasePayFragmentUtils.ORDER_TYPE_H5);

        Bundle bundle = getArguments();
        if (bundle != null && bundle.containsKey(KEY_DIALOG_WEB_VIEW)) {
            isDialogWebView = bundle.getBoolean(KEY_DIALOG_WEB_VIEW, false);
        }
    }

    @Override
    public View setView(LayoutInflater inflater, ViewGroup container, Bundle savedInstanceState) {
        View rootView = inflater.inflate(R.layout.fragment_h5, null);
        ButterKnife.bind(this, rootView);

        if (getActivity() instanceof H5Activity) {
            H5Activity h5Activity = (H5Activity) getActivity();
            mProgressLayout = h5Activity.mProgressLayout;
        }

        initView();
        initData();
        return rootView;
    }

    @Override
    public void onResume() {
        super.onResume();
        payFragmentUtils.onPayResume();
        if (H5Constant.isNeedFlush == true || isNeedFlushPreH5 == true) {
            H5Constant.isNeedFlush = false;
            isNeedFlushPreH5 = false;
            // 加载数据
            initData();
        }
    }

	/**
	 * 执行组件初始化的操作
	 */
    private void initView() {
        // 判断下拉刷新组件是否可用
        isSwipeEnable();
        // 初始化WebView组件
        H5FragmentUtils.initH5View(this);
        // 设置WebView的Client
        mWebView.setWebViewClient(new MWebViewClient(this));
        // 设置可现实js的alert弹窗
        mWebView.setWebChromeClient(new WebChromeClient());

        if (isDialogWebView) {
            mProgressLayout.setCornerResId(R.drawable.map_confirm_bg);
        }
    }

	/**
	 * 执行初始化加载数据的操作
	 */
    private void initData() {
        mProgressLayout.showLoading();
        // 设置title
        H5FragmentUtils.setTitle(this, title);
        // 获取请求URL
        currentUrl = H5FragmentUtils.getUrl(this, currentUrl);
        // 刷新页面
        reflush();
    }

    /**
     * 判断下拉刷新组件是否可用
     */
    private void isSwipeEnable() {
        if (getActivity() == null) {
            return;
        }

        if (isDialogWebView) {
            getActivity().getIntent().putExtra(H5Constant.CARFLUSH, false);
        }
        //判断滑动组件是否可用
        if (getActivity().getIntent().getBooleanExtra(H5Constant.CARFLUSH, true)) {
            swipeRefreshLayout.setEnabled(true);
            swipeRefreshLayout.setColorSchemeResources(R.color.c1, R.color.c1, R.color.c1);
            swipeRefreshLayout.setOnRefreshListener(this);
            mWebView.setOnTouchListener(new View.OnTouchListener() {
                @Override
                public boolean onTouch(View v, MotionEvent event) {
                    if (event.getAction() == MotionEvent.ACTION_DOWN) {
                        int downY = (int) event.getY();
                        if (downY <= DisplayUtil.screenhightPx / 3) {
                            swipeRefreshLayout.setEnabled(true);
                        } else {
                            swipeRefreshLayout.setEnabled(false);
                        }
                    }
                    return false;
                }
            });
        } else {
            swipeRefreshLayout.setEnabled(false);
        }
    }

    /**
     * 执行Webview的下拉刷新操作
     */
    @Override
    public void onRefresh() {
        //判断是否执行刷新动作
        reflush();
    }

    /**
     * 刷新当前页面
     */
    private void reflush() {
        if (Config.isNetworkConnected(mContext)) {
            if (!TextUtils.isEmpty(currentUrl)) {
                H5Cookie.synCookies(mContext, currentUrl, H5Cookie.getToken());
                mWebView.loadUrl(currentUrl);
            } else {
                swipeRefreshLayout.setRefreshing(false);
                mProgressLayout.showError(errorOnClickListener);
            }
        } else {
            swipeRefreshLayout.setRefreshing(false);
            mProgressLayout.showError(errorOnClickListener);
        }
    }

    @Override
    public void onDestroyView() {
        super.onDestroyView();
        /*swipeRefreshLayout.removeView(mWebView);
        mWebView.removeAllViews();
        mWebView.destroy();*/
    }
}
```

（3）初始化WebView组件

```
/**
     * 初始化组件WebView
     *
     * @param h5Fragment
     */
    public static void initH5View(H5Fragment h5Fragment) {
        if (h5Fragment == null || h5Fragment.getActivity() == null) {
            return;
        }

        if (h5Fragment.getActivity().getIntent().getBooleanExtra(H5Constant.SOFT_INPUT_IS_CHANGE_LAYOUT, false)) {
            h5Fragment.getActivity().getWindow().setSoftInputMode(WindowManager.LayoutParams.SOFT_INPUT_ADJUST_PAN | WindowManager.LayoutParams.SOFT_INPUT_STATE_HIDDEN);
        }
        // 设置H5页面默认能够长按复制
        /*if (!h5Fragment.getActivity().getIntent().getBooleanExtra(H5Constant.OPENLONGCLICK, false)) {

            h5Fragment.mWebView.setOnLongClickListener(new View.OnLongClickListener() {
                @Override
                public boolean onLongClick(View v) {
                    return true;
                }
            });
        }*/
        WebSettings webSettings = h5Fragment.mWebView.getSettings();
        webSettings.setJavaScriptEnabled(true);
        webSettings.setLoadWithOverviewMode(true);
        webSettings.setAllowFileAccess(false);
        webSettings.setUseWideViewPort(false);
        webSettings.setCacheMode(WebSettings.LOAD_NO_CACHE);
        webSettings.setDatabaseEnabled(false);
        webSettings.setAppCacheEnabled(false);
        webSettings.setBlockNetworkImage(true);
    }
```

（4）自定义实现WebviewClient对象

```
/**
 * 自定义实现WebviewClient类
 */
public class MWebViewClient extends WebViewClient {

    public H5Fragment h5Fragment = null;
    public Activity h5Activity = null;

    public MWebViewClient(H5Fragment h5Fragment) {
        this.h5Fragment = h5Fragment;
        if (h5Fragment.getActivity() == null) {
            h5Activity = Config.currentContext;
        } else {
            h5Activity = h5Fragment.getActivity();
        }
    }

	/**
	 * 拦截H5页面的a标签跳转，解析scheme协议
	 * 相当于放弃了a标签的使用，转而使用自定义的scheme协议
	 */
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
				// 解析SCHEME跳转
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

	/**
	 * H5页面刚刚开始被webview加载时回调该方法
	 */
    @Override
    public void onPageStarted(WebView view, String url, Bitmap favicon) {
        super.onPageStarted(view, url, favicon);

    }

	/**
	 * H5页面结束被加载时回调该方法
	 */
    @Override
    public void onPageFinished(WebView view, String url) {
        super.onPageFinished(view, url);
        h5Fragment.swipeRefreshLayout.setRefreshing(false);
        if (h5Activity.getTitle().toString().equals("找不到网页")) {
            h5Fragment.mProgressLayout.showError(h5Fragment.errorOnClickListener);
            return;
        }
        if (h5Fragment.isSuccess)
            h5Fragment.mProgressLayout.showContent();
        else
            h5Fragment.mProgressLayout.showError(h5Fragment.errorOnClickListener);

        h5Fragment.onLoadFinish(h5Fragment.isSuccess);
        if (h5Fragment.isSuccess) {
            h5Fragment.mWebView.getSettings().setBlockNetworkImage(false);
        }
    }

    // 该方法为android23中新添加的API，android23中会执行该方法
    @TargetApi(21)
    @Override
    public void onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) {
        if (Build.VERSION.SDK_INT >= 21) {
            if (request.isForMainFrame()) {
                h5Fragment.isSuccess = false;
                h5Fragment.mProgressLayout.showError(h5Fragment.errorOnClickListener);
            }
        }
    }

    /**
     * 在android23中改方法被onReceivedError(WebView view, WebResourceRequest request, WebResourceError error) 替代
     * 因此在android23中执行替代方法
     * 在android23之前执行该方法
     * @param view
     * @param errorCode
     * @param description
     * @param failingUrl
     */
    @Override
    public void onReceivedError(WebView view, int errorCode, String description, String failingUrl) {
        if (Build.VERSION.SDK_INT < 23) {
            h5Fragment.isSuccess = false;
            h5Fragment.mProgressLayout.showError(h5Fragment.errorOnClickListener);
        }
    }
}
```

（5）打开H5页面

```
Intent intent = new Intent(context, H5Activity.class);
        intent.putExtra(H5Constant.MURL, currentUrl);
        intent.putExtra(H5Constant.TITLE, title);
        context.startActivity(intent);
```

可以发现在产品实际开发过程中使用webview的页面都是整个的Activity页面，也就是说整个Activity页面只有一个webview控件，所以这时候页面的内容都是通过H5实现的。
然后当我们需要打开H5页面的时候可以通过服务器下发H5页面的url和title，并作为参数传递给H5Activity，然后打开该url所表示的网页。

同时我们使用Fragment用于实现加载H5页面的所以，所以以后当我们需要在其他地方使用加载H5页面的时候可以很方便的一直。

在MWebviewClient的shouldOverrideUrlLoading方法中我们拦截了所有的a标签跳转，转而实现我们自身的scheme协议，即a标签的跳转链接不再是常规的http链接，而是我们自定义的scheme协议，具体可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/51685310">android产品研发（十一）-->应用内跳转scheme协议</a>


**总结：**

- 本文中我们介绍了Hybird开发的概念，hybird开发的作用，android中如何实现hybird开发，android实现hybird的例子，产品中对hybird开发的实践

- 在定义webview的时候可以设置WebviewSettings，设置WebviewClient，设置WebChromeClient等参数对象

- 可以在WebviewClient的shouldOverrideUrlLoading方法中拦截a标签的跳转并执行相应的逻辑


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

