上一篇文章中我们讲解了android app实现长连接的几种方式，各自的优缺点以及具体的实现，一般而言使用第三方的推送服务已经可以满足了基本的业务需求，当然了若是对技术有追求的可以通过NIO或者是MINA实现自身的长连接服务，但是自己实现的长连接服务一来比较复杂耗时比较多，而且可能过程中有许多坑要填，一般而言推荐使用第三方的推送服务，稳定简单，具体管理长连接部分的模块可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/51671005">android产品研发（十二）-->App长连接实现</a>。

而本文将讲解app端的轮训请求服务，一般而言我们经常将轮训操作用于请求服务器。比如某一个页面我们有定时任务需要时时的从服务器获取更新信息并显示，比如当长连接断掉之后我们可能需要启动轮训请求作为长连接的补充等，所以这时候就用到了轮训服务。

**什么是轮训请求**

在说明我们轮训请求之前，这里先说明一下什么叫轮训请求，我的理解就是App端每隔一定的时间重复请求的操作就叫做轮训请求，比如：App端每隔一段时间上报一次定位信息，App端每隔一段时间拉去一次用户状态等，这些应该都是轮训请求，那么前一篇我们讲了App端的长连接，为什么我们有了长连接之后还需要轮训操作呢？

这是因为我们的长连接并不是稳定的可靠的，而我们执行轮训操作的时候一般都是要稳定的网络请求，而且轮训操作一般都是有生命周期的，即在一定的生命周期内执行轮训操作，而我们的长连接一般都是整个进程生命周期的，所以从这方面讲也不太适合。

**轮训请求实践**

**与长连接相关的轮训请求**

1. 上一篇我们在讲解长连接的时候说过长连接有可能会断，而这时候在长连接断的时候我们就需要启动一个轮训服务，它作为长连接的补充。

```
/**
     * 启动轮询服务
     */
    public void startLoopService() {
        // 启动轮询服务
        // 暂时不考虑加入网络情况的判断...
        if (!LoopService.isServiceRuning) {
            // 用户是登录态，启动轮询服务
            if (UserConfig.isPassLogined()) {
                // 判断当前长连接的状态，若长连接已连接，则不再开启轮询服务
                if (MinaLongConnectManager.session != null && MinaLongConnectManager.session.isConnected()) {
                    LoopService.quitLoopService(context);
                    return;
                }
                LoopService.startLoopService(context);
            } else {
                LoopService.quitLoopService(context);
            }
        }
    }
```
这里就是我们执行轮训服务的操作代码，其作用就是启动了一个轮训service（即轮训服务），然后在轮训服务中执行具体的轮训请求，既然这样我们就具体看一下这个service的代码逻辑。


- 与长连接相关的轮训服务请求
```
/**
 * 长连接异常时启动服务，长连接恢复时关闭服务
 */
public class LoopService extends Service {

    public static final String ACTION = "com.youyou.uuelectric.renter.Service.LoopService";

    /**
     * 客户端执行轮询的时间间隔，该值由StartQueryInterface接口返回，默认设置为30s
     */
    public static int LOOP_INTERVAL_SECS = 30;
    /**
     * 轮询时间间隔(MLOOP_INTERVAL_SECS 这个时间间隔变量有服务器下发，此时轮询服务的场景与逻辑与定义时发生变化，涉及到IOS端，因此采用自己定义的常量在客户端写死时间间隔)
     */
    public static int MLOOP_INTERVAL_SECS = 30;
    /**
     * 当前服务是否正在执行
     */
    public static boolean isServiceRuning = false;
    /**
     * 定时任务工具类
     */
    public static Timer timer = new Timer();

    private static Context context;

    public LoopService() {
        isServiceRuning = false;
    }

    //-------------------------------使用闹钟执行轮询服务------------------------------------

    /**
     * 启动轮询服务
     */
    public static void startLoopService(Context context) {
        if (context == null)
            return;
        quitLoopService(context);
        L.i("开启轮询服务，轮询间隔：" + MLOOP_INTERVAL_SECS + "s");
        AlarmManager manager = (AlarmManager) context.getApplicationContext().getSystemService(Context.ALARM_SERVICE);
        Intent intent = new Intent(context.getApplicationContext(), LoopService.class);
        intent.setAction(LoopService.ACTION);
        PendingIntent pendingIntent = PendingIntent.getService(context.getApplicationContext(), 1, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        // long triggerAtTime = SystemClock.elapsedRealtime() + 1000;
        /**
         * 闹钟的第一次执行时间，以毫秒为单位，可以自定义时间，不过一般使用当前时间。需要注意的是，本属性与第一个属性（type）密切相关，
         * 如果第一个参数对应的闹钟使用的是相对时间（ELAPSED_REALTIME和ELAPSED_REALTIME_WAKEUP），那么本属性就得使用相对时间（相对于系统启动时间来说），
         *      比如当前时间就表示为：SystemClock.elapsedRealtime()；
         * 如果第一个参数对应的闹钟使用的是绝对时间（RTC、RTC_WAKEUP、POWER_OFF_WAKEUP），那么本属性就得使用绝对时间，
         *      比如当前时间就表示为：System.currentTimeMillis()。
         */
        manager.setRepeating(AlarmManager.RTC_WAKEUP, System.currentTimeMillis(), MLOOP_INTERVAL_SECS * 1000, pendingIntent);
    }

    /**
     * 停止轮询服务
     */
    public static void quitLoopService(Context context) {
        if (context == null)
            return;
        L.i("关闭轮询闹钟服务...");
        AlarmManager manager = (AlarmManager) context.getApplicationContext().getSystemService(Context.ALARM_SERVICE);
        Intent intent = new Intent(context.getApplicationContext(), LoopService.class);
        intent.setAction(LoopService.ACTION);
        PendingIntent pendingIntent = PendingIntent.getService(context.getApplicationContext(), 1, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        manager.cancel(pendingIntent);
        // 关闭轮询服务
        L.i("关闭轮询服务...");
        context.stopService(intent);
    }

    @Override
    public void onCreate() {
        super.onCreate();

        context = getApplicationContext();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        L.i("开始执行轮询服务... \n 判断当前用户是否已登录...");
        // 若当前网络不正常或者是用户未登陆，则不再跳转
        if (UserConfig.isPassLogined()) {
            // 判断当前长连接状态，若长连接正常，则关闭轮询服务
            L.i("当前用户已登录... \n 判断长连接是否已经连接...");
            if (MinaLongConnectManager.session != null && MinaLongConnectManager.session.isConnected()) {
                L.i("长连接已恢复连接，退出轮询服务...");
                quitLoopService(context);
            } else {
                if (isServiceRuning) {
                    return START_STICKY;
                }
                // 启动轮询拉取消息
                startLoop();
            }
        } else {
            L.i("用户已退出登录，关闭轮询服务...");
            quitLoopService(context);
        }
        return START_STICKY;
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
        L.i("轮询服务退出，执行onDestory()方法，inServiceRuning赋值false");
        isServiceRuning = false;
        timer.cancel();
        timer = new Timer();
    }

    @Override
    public IBinder onBind(Intent intent) {
        throw new UnsupportedOperationException("Not yet implemented");
    }

    /**
     * 启动轮询拉去消息
     */
    private void startLoop() {
        if (timer == null) {
            timer = new Timer();
        }
        timer.schedule(new TimerTask() {
            @Override
            public void run() {
                isServiceRuning = true;
                L.i("长连接未恢复连接，执行轮询操作... \n 轮询服务中请求getInstance接口...");
                LoopRequest.getInstance(context).sendLoopRequest();
            }
        }, 0, MLOOP_INTERVAL_SECS * 1000);
    }
}
```
可以发现这里的service轮训服务的代码量还是比较多的，但是轮训服务请求代码注释已经很详细了，所以就不做过多的说明，需要说明的是其核心就是通过Timer对象每个一段时间执行一次网络请求。具体的网络请求代码：

```
L.i("长连接未恢复连接，执行轮询操作... \n 轮询服务中请求getInstance接口...");           

LoopRequest.getInstance(context).sendLoopRequest();
```
这里的轮训服务请求核心逻辑：当长连接出现异常时，启动轮训服务，并通过Timer对象每隔一定时间拉取服务器状态，当长连接恢复时，关闭轮训服务。这就是我们与长连接有关的轮训服务的代码执行逻辑，看完这部分之后我们再看一下与页面相关的轮训请求的执行逻辑。


**与页面相关的轮训请求**

- 与页面相关的轮训请求
我们的App中当用户停留在某一个页面的时候我们可能需要定时的拉取用户状态，这时候也需要使用轮训请求拉取服务器状态，当用户离开该页面的时候关闭轮训服务请求。

这里我们看一下我们产品当前行程页面的轮训操作，用于轮训请求当前用户的车辆里程，费用，用时等信息，具体可参考下图：
<div align=center><img width="350" height="550" align="center" src="http://img.blog.csdn.net/20160621084242823"></div>

其实在当前Fragment页面有一个定时的拉去订单信息的轮训请求，下面我们具体看一下这个定时请求的执行逻辑：

```
/**
 * TimerTask对象，主要用于定时拉去服务器信息
 */
public class Task extends TimerTask {
        @Override
        public void run() {
            L.i("开始执行执行timer定时任务...");
            handler.post(new Runnable() {
                @Override
                public void run() {
                    isFirstGetData = false;
                    getData(true);
                }
            });
        }
    }
```
而这里的getData方法就是拉去服务器状态的方法，这里不做过多的解释，当用户退出这个页面的时候需要清除这里的轮训操作。所以在Fragment的onDesctoryView方法中执行了清除timerTask的操作。

```
@Override
    public void onDestroyView() {
        super.onDestroyView();
        ...
        if (timer != null) {
            timer.cancel();
            timer = null;
        }
        if (timerTask != null) {
            timerTask.cancel();
            timerTask = null;
        }
        ...
    }
```
这样当用户打开这个页面的时候初始化TimerTask对象，每个一分钟请求一次服务器拉取订单信息并更新UI，当用户离开页面的时候清除TimerTask对象，即取消轮训请求操作。可以发现上面我们看到的与长连接和页面相关的轮训请求服务都是通过timer对象的定时任务实现的轮训请求服务，下面我们看一下如何通过Handler对象实现轮训请求服务。


**通过Handler对象实现轮训请求**

- 下面我们来看一个通过Handler异步消息实现的轮训请求服务。

```
/**
     * 默认的时间间隔：1分钟
     */
    private static int DEFAULT_INTERVAL = 60 * 1000;
    /**
     * 异常情况下的轮询时间间隔:5秒
     */
    private static int ERROR_INTERVAL = 5 * 1000;
    /**
     * 当前轮询执行的时间间隔
     */
    private static int interval = DEFAULT_INTERVAL;
    /**
     * 轮询Handler的消息类型
     */
    private static int LOOP_WHAT = 10;
    /**
     * 是否是第一次拉取数据
     */
    private boolean isFirstRequest = false;
    /**
     * 第一次请求数据是否成功
     */
    private boolean isFirstRequestSuccess = false;

    /**
     * 开始执行轮询，正常情况下，每隔1分钟轮询拉取一次最新数据
     * 在onStart时开启轮询
     */
    private void startLoop() {
        L.i("页面onStart，需要开启轮询");
        loopRequestHandler.sendEmptyMessageDelayed(LOOP_WHAT, interval);
    }

    /**
     * 关闭轮询，在界面onStop时，停止轮询操作
     */
    private void stopLoop() {
        L.i("页面已onStop，需要停止轮询");
        loopRequestHandler.removeMessages(LOOP_WHAT);
    }

    /**
     * 处理轮询的Handler
     */
    private Handler loopRequestHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {

            // 如果首次请求失败，
            if (!isFirstRequestSuccess) {
                L.i("首次请求失败，需要将轮询时间设置为:" + ERROR_INTERVAL);
                interval = ERROR_INTERVAL;
            } else {
                interval = DEFAULT_INTERVAL;
            }

            L.i("轮询中-----当前轮询间隔：" + interval);

            loopRequestHandler.removeMessages(LOOP_WHAT);

            // 首次请求为成功、或者定位未成功时执行重定位，并加载网点数据
            if (!isFirstRequestSuccess || !Config.locationIsSuccess()) {
                isClickLocationButton = false;
                doLocationOption();
            } else {
                loadData();
            }

            System.gc();

            loopRequestHandler.sendEmptyMessageDelayed(LOOP_WHAT, interval);

        }
    };
```
这里是通过Handler实现的轮训操作，其核心原理就是在handler的handlerMessage方法中，接收到消息之后再次发送延迟消息，这里的延迟时间就是我们定义的轮训间隔时间，这样当我们下次接收到消息的时候又一次发送延迟消息，从而造成我们时时发送轮训消息的情景。

以上就是我们实现轮训操作的两种方式：

- Timer对象实现轮训操作

- Handler对象实现轮训操作

上面我们分析了轮训请求的不同使用场景，作用以及实现方式，当我们在具体的开发过程中需要定时的向服务器拉取消息的时候就可以考虑使用轮训请求了。

**总结：**

- 轮训操作一般都是通过定时请求服务器拉取信息并更新UI；

- 轮训操作一般都有一定的生命周期，比如在某个页面打开时启动轮训操作，在某个页面关闭时取消轮训操作；

- 轮训操作的请求间隔需要根据具体的需求确定，间隔时间不宜果断，否则可能造成并发性问题；

- 产品开发过程中，某些需要试试更新服务器拉取信息并更新UI时，可以考虑使用轮训操作实现；

- 可以通过Timer对象和Handler对象两种方式实现轮训请求操作；

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
