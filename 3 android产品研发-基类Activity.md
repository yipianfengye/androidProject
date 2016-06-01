在上一篇文章中我们介绍了在android产品研发过程中，启动页的优化工作，比如启动页性能优化，启动页渐进动画效果，启动页屏蔽返回按键等等，而在本文中我们将要介绍一下在App产品研发中都会复写的基类Activity。

在实际的android产品研发中，一般的我们在写Activity的时候都会继承于一个基类Activity，该Activity是所有的Activity的基类。在该基类中我们主要用于重写一些共有的逻辑。好处是显而易见的对于一些Activity的共有逻辑我们不必要在每个Activity中都重新写一遍，只需要在基类Activity中写一遍就好了。 

下面我就讲解一下在产品研发过程中我对基类Activity的设计。

（一）基类Activity是如何使用的？
定义一个BaseActivity，让App中所有的Activity都继承于BaseActivity；

（二）基本Activity包含的内容

- 在BaseActivity的生命周期中复写友盟数据统计方法。
用过友盟数据统计的同学应该知道，为了统计每个页面的点击事件，页面访问路径，异常信息等我们需要在Activity的生命周期方法中添加友盟的API，这些都是一些相同的逻辑，难道我们需要在每个Activity中都重写这些友盟API么？答案当然是否定的，友盟官方也不建议我们这么做，我们可以在一个基类Activity的生命周期方法中重写这些友盟API，并让我们需要统计的页面继承于该基类Activity。
```
@Override
protected void onResume() {
	MobclickAgent.onResume(this);
}

@Override
protected void onPause() {
	MobclickAgent.onPause(this);
}

（其他生命周期方法暂略）
...
```
由于我们需要统计不同的Activity页面的数据，所以我们需要在各个Activity的生命周期中都要重写友盟的API，那么这里我们就完全可以将这种重复性的动作下发到BaseActivity中，而让所有的Activity都继承于BaseActivity，进而我们的Activity就在生命周期方法中重写了友盟的API。



- 在生命周期方法中保存Context对象，保存Activity栈

```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        mContext = this;
        Config.setActivityState(this);
        ...
    }

public static void setActivityState(Activity activity) {
        currentContext = activity;
        // 设置App只能竖屏显示
        activity.setRequestedOrientation(ActivityInfo.SCREEN_ORIENTATION_PORTRAIT);
        // 向自身维护的Acitivty栈（数据类型为List）中添加Activity
        UUApp.getInstance().addActivity(activity);
    }

@Override
    protected void onDestroy() {
        super.onDestroy();
        // 在BaseActivity中的Activity栈中移除Activity
        UUApp.getInstance().removeActivity(this);
    }
```
因为我们在Activity页面中经常需要使用“this”，但是到处都是用this显得不太好，这时候我们可以在BaseActivity中定义一个Actiivty类型的mContext成员变量并在BaseActivity中的onCreate方法中赋值为this，这样我们在子Activity中需要使用this的地方直接使用mContext成员变量就好了。

并且我们在内存中保存了一个Activity的自定义栈，正常的做法是在Activity的onCreate方法中执行入栈操作，在Activity的onDestory方法中执行出栈操作，由于这些都是共有的逻辑因此我们把这两个操作放到BaseActivity中。


- 在生命周期方法中对事件总线框架EventBus进行注册和反注册

使用过EventBus的同学应该知道其能够实现事件传递的核心机制就是将Activity对象“注册”到EventBus中，但是有注册也必须要有反注册的机制，因为Activity不是常驻内存的，Activity也有销毁的时候，这时候就需要从EventBus中“反注册”，一个比较好的方式就是在Activity
```
@Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        ...
        EventBus.getDefault().register(this);
	    ...
    }

@Override
    protected void onDestroy() {
        super.onDestroy();
        ...
        EventBus.getDefault().unregister(this);
        ...
    }

/**
     * event 更改显示登陆状态
     */
    public void onEventMainThread(BaseEvent event) {
        ...doSomething...
    }
```
这样我们就不必要单独在Activity中重写EventBus的注册和反注册逻辑了。

- 在onResume方法中执行更新登录用户票据更新的操作

在App端保存了用户的token信息，但是这里的token信息不是一直有效的，有效期为30天，所以这里就需要有一个更新token的操作，App现在的更新操作是判断token的有效期是否过了一半，即15天，若过了一半了，则在新打开的Activity时执行更新票据操作，所以这也是共有的逻辑，即在BaseActivity的onResume中执行更新票据的操作。
```
@Override
    protected void onResume() {
        super.onResume();
        /**
         * 验证用户票据是否失效，失效的话则默认执行更换票据操作
         *     startActivity中不需要更新票据，否则起始页有可能会弹出登陆提示框
         */
        if (UserConfig.isNeedUpdateTicket() && (!(this instanceof StartActivity) || !this.getClass().getName().equals(StartActivity.class.getName()))) {
            if (!UserConfig.isUpdateTicketing) {
                L.d("onResume 中更新用户票据");
                UserConfig.requestUpdateTicket();
            }
        }
        ...
    }
```

- 在setContentView方法中，重写加载布局文件的逻辑，统一App页面布局

```
@Override
    public void setContentView(int layoutResID) {
        View view = getLayoutInflater().inflate(R.layout.activity_base_layout, null);
        super.setContentView(view);

        if (Build.VERSION.SDK_INT == Build.VERSION_CODES.KITKAT) {
            view.setFitsSystemWindows(true);
            setTranslucentStatus(true);
            SystemBarTintManager tintManager = new SystemBarTintManager(this);
            tintManager.setStatusBarTintEnabled(true);
            tintManager.setStatusBarTintResource(R.color.colorPrimaryDark);

        }

        initDefaultView(layoutResID);
        initDefaultToolBar();

    }

/**
     * 初始化默认的布局组件
     *
     * @param layoutResID
     */
    private void initDefaultView(int layoutResID) {
        mProgressLayout = (ProgressLayout) findViewById(R.id.progress_layout);
        mProgressLayout.setAttachActivity(this);
        mProgressLayout.setUseSlideBack(false);
        mToolbarContainer = (FrameLayout) findViewById(R.id.toolbar_container);
        mDefaultToolBar = (Toolbar) findViewById(R.id.default_toolbar);
        mToolbarTitle = (TextView) findViewById(R.id.toolbar_title);
        mRvRight = (RippleView) findViewById(R.id.rv_right);
        mRightOptButton = (TextView) findViewById(R.id.right_opt_button);
        mContentContainer = (FrameLayout) findViewById(R.id.fl_content_container);

        View childView = layoutInflater.inflate(layoutResID, null);
        mContentContainer.addView(childView, 0);
    }

/**
     * 初始化默认的ToolBar
     */
    private void initDefaultToolBar() {
        if (mDefaultToolBar != null) {
            String label = getTitle().toString();
            setTitle(label);
            setSupportActionBar(mDefaultToolBar);
            mDefaultToolBar.setNavigationIcon(R.mipmap.toolbar_back_icn_transparent);
        }
    }
```
在BaseActivity中重写了setContentView方法，然后统一布局文件，即让我们调用setContentView传递的layoutId加载到我们自定义的ViewContain中，这样我们就统一了Activity的布局文件，当然了这也是共有的逻辑，我们可以卸载BaseActivity中。

- 执行共有的UI操作，比如显示Toast，显示SnakeBar，显示Progress，显示Dialog等

```
public void showToast(String text) {
        if (text != null && !text.trim().equals("")) {
            Toast.makeText(getApplicationContext(), text, Toast.LENGTH_SHORT).show();
        }
    }

public void showProgress(boolean canCancled, final Config.ProgressCancelListener listener) {
        Config.showProgressDialog(mContext, canCancled, listener);
    }

    public void dismissProgress() {
        Config.dismissProgress();
    }

public void showResponseCommonMsg(HeaderCommon.ResponseCommonMsg msg) {
        if (msg.getMsg() != null && msg.getMsg().length() > 0) {
            if (msg.hasShowType()) {
                if (msg.getShowType().equals(HeaderCommon.ResponseCommonMsgShowType.TOAST)) {
                    showSnackBarMsg(msg.getMsg());
                } else if (msg.getShowType().equals(HeaderCommon.ResponseCommonMsgShowType.WINDOW)) {
                    showResponseCommonMsg(msg, null);
                }
            } else {
                showSnackBarMsg(msg.getMsg());
            }
        }
    }
```
我们可以在BaseActivity中重写了一些显示Toast，Dialog，SnakeBar的操作，这样我们在Activity中显示这些UI的时候可以统一调用方法与UI风格。


从上面我们分析的基类内容可以知道一般在涉及Activity的时候都需要重写一个基类Activity，用于重写Activity中的共有逻辑，避免我们在每个Activity中都重写相关的重复逻辑。

另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
