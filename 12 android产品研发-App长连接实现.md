上一篇文章中我们讲解了android应用内页面跳转协议-scheme协议，通过该协议我们可以跳转至指定的Activity，并在该Activity中解析scheme用于跳转到指定的页面，我们可以利用scheme协议实现应用内页面跳转、H5页面与Native页面相互跳转、通知栏消息跳转相应页面等，具体可参考：<a href="http://blog.csdn.net/qq_23547831/article/details/51685310"> android产品研发（十一）-->使用scheme实现页面跳转</a>。

而本文中我们将讲解一下App的长连接实现。一般而言长连接已经是App的标配了，推送功能的实现基础就是长连接，当然了我们也可以通过轮训操作实现推送功能，但是轮训一般及时性比较差，而且网络消耗与电量销毁比较多，因此一般推送功能都是通过长连接实现的。

那么如何实现长连接呢？现在一般有这么几种实现方式：

- 使用第三方的长连接服务；

- 通过NIO等方案实现长连接服务；

- 通过MINA等第三方框架实现长连接；

###几种长连接服务的具体实现，以及各自的优缺点。


####1. 使用第三方的长连接服务
介绍：这是最简单的方式，我们可以通过接入极光推送，百度推送，友盟等第三方服务实现长连接，通过接入第三方的API我们可以很方便的接入第三方的长连接，推送服务，但是这种方式定制化程度不太好，如果对长连接服务不是要求特别高，对定制化要求不是很高的话基本可以考虑这种方式（目前主流的App都是使用第三方的长连接服务）
优势：简单，方便
劣势：定制化程度不高

####2. 使用NIO等方案实现长连接服务
介绍：通过NIO的方式实现长连接，这种方式对技术要求程度比较高，基本都是通过java API实现长连接，实现心跳包，实现异常情况的容错等操作，可以说通过NIO实现长连接对技术要求很高，一般如果没有成行的技术方案比建议这么做，就算实现了长连接，后期连接的维护，对电量，流量的损耗等都需要持续的优化。
优势：定制化比较高
劣势：技术要求高，需要持续的维护

####3. 使用MINA等第三方框架实现长连接
介绍：MINA是一个第三方的NIO框架，该框架实现了一整套的长连接机制，包括长连接的建立，心跳包的实现，异常机制的容错等。使用MINA实现长连接可以定制化的实现一些特有的功能，并且比NIO方案较为简单，因为其已经封装了一些长连接的特有机制，比如心跳包，容错等。
优势：可定制，较NIO方法简单
劣势：也需要一定的技术储备

### 长连接具体实现

在我们的Android客户端中长连接的实现机制采用--MINA方式。这里多说一句，一开始的长连接采用的是NIO方案，但是采用这种方案之后踩了很多坑，包括心跳，容错等机制都是自己写的，所以耗费了大量的时间，而且对手机电量的消耗很大，最后决定使用MINA NIO框架重新实现一遍长连接，后来经过实测，长连接的稳定性还有耗电量，流量的消耗等指标方面有了很大的提高。

下面我将简单的介绍一下通过NIO实现长连接的具体流程：

- 引入MINA jar包，在App启动页面，登录页面启动长连接；

- 创建后台服务，在服务中创建MINA长连接；

- 实现心跳包，重写一些容错机制；

- 实现长连接断了之后的重连机制，并且重连次数有限制不能一直重连；

- 长连接断了之后实现轮训操作，这里的轮训服务只有在长连接断了之后才启动，在长连接恢复之后关闭；


以下就是在长连接中实现的具体代码：

- 在Application的onCreate方法中检测App是否登录，若登录的话启动长连接

```
/**
 * 在Application的onCreate方法中执行启动长连接的操作
 **/
@Override
    public void onCreate() {
        ...
        // 登录后开启长连接
        if (UserConfig.isPassLogined()) {
            L.i("用户已登录，开启长连接...");
            startLongConn();
        }
		...
    }
```

- 通过闹钟服务实现具体的启动长连接service的操作，即每隔60秒钟判断长连接是否启动，若未启动则实现启动操作

```
    public void startLongConn() {
        quitLongConn();
        L.i("长连接服务已开启");
        AlarmManager manager = (AlarmManager) getSystemService(Context.ALARM_SERVICE);
        Intent intent = new Intent(this, LongConnService.class);
        intent.setAction(LongConnService.ACTION);
        PendingIntent pendingIntent = PendingIntent.getService(this, 0, intent, PendingIntent.FLAG_UPDATE_CURRENT);
        long triggerAtTime = SystemClock.elapsedRealtime();
        manager.setRepeating(AlarmManager.RTC_WAKEUP, triggerAtTime, 60 * 1000, pendingIntent);
    }
```

- 下面的代码就是长连接服务的具体实现

```
/**
 * 后台长连接服务
 **/
public class LongConnService extends Service {
    public static String ACTION = "com.youyou.uuelectric.renter.Service.LongConnService";
    private static MinaLongConnectManager minaLongConnectManager;
    public String tag = "LongConnService";
    private Context context;

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        context = getApplicationContext();
        // 执行启动长连接的操作
        startLongConnect();
        ObserverManager.addObserver("LongConnService", stopListener);
        return START_STICKY;
    }

    public ObserverListener stopListener = new ObserverListener() {
        @Override
        public void observer(String from, Object obj) {
            closeConnect();
        }
    };

    @Override
    public void onDestroy() {
        super.onDestroy();
        closeConnect();
    }

    private void startLongConnect() {
        if (Config.isNetworkConnected(context)) {
            if (minaLongConnectManager != null && minaLongConnectManager.checkConnectStatus()) {
                L.i("长连接状态正常...");
                return;
            }
            if (minaLongConnectManager == null) {
                startThreadCreateConnect();
            } else {
                if (minaLongConnectManager.connectIsNull() && minaLongConnectManager.isNeedRestart()) {
                    L.i("session已关闭，需要重新创建一个session");
                    minaLongConnectManager.startConnect();
                } else {
                    L.i("长连接已关闭，需要重开一个线程来重新创建长连接");
                    startThreadCreateConnect();
                }
            }
        }

    }

    private final AtomicInteger mCount = new AtomicInteger(1);

    private void startThreadCreateConnect() {
        if (UserConfig.getUserInfo().getB3Key() != null && UserConfig.getUserInfo().getSessionKey() != null) {
            System.gc();

            new Thread(new Runnable() {
                @Override
                public void run() {
	                // 执行具体启动长连接操作
                    minaLongConnectManager = MinaLongConnectManager.getInstance(context);
                    minaLongConnectManager.crateLongConnect();
                }
            }, "longConnectThread" + mCount.getAndIncrement()).start();
        }
    }


    private void closeConnect() {

        if (minaLongConnectManager != null) {
            minaLongConnectManager.closeConnect();
        }
        minaLongConnectManager = null;

        // 停止长连接服务LongConnService
        stopSelf();
    }

    @Override
    public IBinder onBind(Intent intent) {
        throw new UnsupportedOperationException("Not yet implemented");
    }
}
```

- 而下面的代码就是长连接的具体实现操作，具体的代码有相关注释说明

```
/**
 * 具体实现长连接的管理对象
 **/
public class MinaLongConnectManager {

    private static final String TAG = MinaLongConnectManager.class.getSimpleName();
    /**
     * 服务器端口号
     */
    public static final int DEFAULT_PORT = 18156;
    /**
     * 连接超时时间，30 seconds
     */
    public static final long SOCKET_CONNECT_TIMEOUT = 30 * 1000L;

    /**
     * 长连接心跳包发送频率，60s
     */
    public static final int KEEP_ALIVE_TIME_INTERVAL = 60;
    private static Context context;
    private static MinaLongConnectManager minaLongConnectManager;

    private static NioSocketConnector connector;
    private static ConnectFuture connectFuture;
    public static IoSession session;
    private static ExecutorService executorService = Executors.newSingleThreadExecutor();

    /**
     * 长连接是否正在连接中...
     */
    private static boolean isConnecting = false;

    private MinaLongConnectManager() {
        EventBus.getDefault().register(this);
    }

    public static synchronized MinaLongConnectManager getInstance(Context ctx) {

        if (minaLongConnectManager == null) {
            context = ctx;
            minaLongConnectManager = new MinaLongConnectManager();
        }
        return minaLongConnectManager;
    }

    /**
     * 检查长连接的各种对象状态是否正常，正常情况下无需再创建
     *
     * @return
     */
    public boolean checkConnectStatus() {
        if (connector != null && connector.isActive() && connectFuture != null && connectFuture.isConnected() && session != null && session.isConnected()) {
            return true;
        } else {
            return false;
        }
    }

    public boolean connectIsNull() {
        return connector != null;
    }

    /**
     * 创建长连接，配置过滤器链和心跳工厂
     */
    public synchronized void crateLongConnect() {
        // 如果是长连接正在创建中
        if (isConnecting) {
            L.i("长连接正在创建中...");
            return;
        }
        if (!Config.isNetworkConnected(context)) {
            L.i("检测到网络未打开，无法正常启动长连接，直接return...");
            return;
        }
        // 检查长连接的各种对象状态是否正常，正常情况下无需再创建
        if (checkConnectStatus()) {
            return;
        }
        isConnecting = true;
        try {
            connector = new NioSocketConnector();
            connector.setConnectTimeoutMillis(SOCKET_CONNECT_TIMEOUT);

            if (L.isDebug) {
                if (!connector.getFilterChain().contains("logger")) {
                    // 设置日志输出工厂
                    connector.getFilterChain().addLast("logger", new LoggingFilter());
                }
            }
            if (!connector.getFilterChain().contains("codec")) {
                // 设置请求和响应对象的编解码操作
                connector.getFilterChain().addLast("codec", new ProtocolCodecFilter(new LongConnectProtocolFactory()));
            }
            // 创建心跳工厂
            ClientKeepAliveMessageFactory heartBeatFactory = new ClientKeepAliveMessageFactory();
            // 当读操作空闲时发送心跳
            KeepAliveFilter heartBeat = new KeepAliveFilter(heartBeatFactory, IdleStatus.READER_IDLE);
            // 设置是否将事件继续往下传递
            heartBeat.setForwardEvent(true);
            // 设置心跳包请求后超时无反馈情况下的处理机制，默认为关闭连接,在此处设置为输出日志提醒
            heartBeat.setRequestTimeoutHandler(KeepAliveRequestTimeoutHandler.LOG);
            //设置心跳频率
            heartBeat.setRequestInterval(KEEP_ALIVE_TIME_INTERVAL);
            if (!connector.getFilterChain().contains("keepAlive")) {
                connector.getFilterChain().addLast("keepAlive", heartBeat);
            }
            if (!connector.getFilterChain().contains("reconnect")) {
                // 设置长连接重连过滤器，当检测到Session(会话)断开后，重连长连接
                connector.getFilterChain().addLast("reconnect", new LongConnectReconnectionFilter());
            }
            // 设置接收和发送缓冲区大小
            connector.getSessionConfig().setReceiveBufferSize(1024);
            connector.getSessionConfig().setSendBufferSize(1024);
            // 设置读取空闲时间：单位为s
            connector.getSessionConfig().setReaderIdleTime(60);

            // 设置长连接业务逻辑处理类Handler
            LongConnectHandler longConnectHandler = new LongConnectHandler(this, context);
            connector.setHandler(longConnectHandler);

        } catch (Exception e) {
            e.printStackTrace();
            closeConnect();
        }

        startConnect();
    }

    /**
     * 开始或重连长连接
     */
    public synchronized void startConnect() {
        if (connector != null) {
            L.i("开始创建长连接...");
            boolean isSuccess = beginConnect();
            // 创建成功后，修改创建中状态
            if (isSuccess) {
                isNeedRestart = false;
                if (context != null) {
                    // 长连接启动成功后，主动拉取一次消息
                    LoopRequest.getInstance(context).sendLoopRequest();
                }
            } else {
                // 启动轮询服务
                startLoopService();
            }
            isConnecting = false;
//            printProcessorExecutor();
        } else {
            L.i("connector已为null，不能执行创建连接动作...");
        }
    }

    /**
     * 检测MINA中线程池的活动状态
     */
    private void printProcessorExecutor() {
        Class connectorClass = connector.getClass().getSuperclass();
        try {
            L.i("connectorClass:" + connectorClass.getCanonicalName());
            Field field = connectorClass.getDeclaredField("processor");
            field.setAccessible(true);
            Object connectorObject = field.get(connector);
            if (connectorObject != null) {
                SimpleIoProcessorPool processorPool = (SimpleIoProcessorPool) connectorObject;
                Class processPoolClass = processorPool.getClass();
                Field executorField = processPoolClass.getDeclaredField("executor");
                executorField.setAccessible(true);
                Object executorObject = executorField.get(processorPool);
                if (executorObject != null) {
                    ThreadPoolExecutor threadPoolExecutor = (ThreadPoolExecutor) executorObject;
                    L.i("线程池中当前线程数：" + threadPoolExecutor.getPoolSize() + "\t 核心线程数:" + threadPoolExecutor.getCorePoolSize() + "\t 最大线程数:" + threadPoolExecutor.getMaximumPoolSize());
                }

            } else {
                L.i("connectorObject = null");
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }


    /**
     * 开始创建Session
     *
     * @return
     */
    public boolean beginConnect() {

        if (session != null) {
            session.close(false);
            session = null;
        }
        if (connectFuture != null && connectFuture.isConnected()) {
            connectFuture.cancel();
            connectFuture = null;
        }
        FutureTask<Boolean> futureTask = new FutureTask<>(new Callable<Boolean>() {
            @Override
            public Boolean call() {
                try {
                    InetSocketAddress address = new InetSocketAddress(NetworkTask.getBASEURL(), DEFAULT_PORT);
                    connectFuture = connector.connect(address);
                    connectFuture.awaitUninterruptibly(3000L);
                    session = connectFuture.getSession();
                    if (session == null) {
                        L.i(TAG + "连接创建失败...当前环境:" + NetworkTask.getBASEURL());
                        return false;
                    } else {
                        L.i(TAG + "长连接已启动，连接已成功...当前环境:" + NetworkTask.getBASEURL());
                        return true;
                    }
                } catch (Exception e) {
                    return false;
                }
            }
        });

        executorService.submit(futureTask);
        try {
            return futureTask.get();
        } catch (Exception e) {
            return false;
        }

    }

    /**
     * 关闭连接，根据传入的参数设置session是否需要重新连接
     */
    public synchronized void closeConnect() {
        if (session != null) {
            session.close(false);
            session = null;
        }
        if (connectFuture != null && connectFuture.isConnected()) {
            connectFuture.cancel();
            connectFuture = null;
        }
        if (connector != null && !connector.isDisposed()) {
            // 清空里面注册的所以过滤器
            connector.getFilterChain().clear();
            connector.dispose();
            connector = null;
        }
        isConnecting = false;
        L.i("长连接已关闭...");
    }

    private volatile boolean isNeedRestart = false;

    public boolean isNeedRestart() {
        return isNeedRestart;
    }

    public void onEventMainThread(BaseEvent event) {
        if (event == null || TextUtils.isEmpty(event.getType()))
            return;
        if (EventBusConstant.EVENT_TYPE_NETWORK_STATUS.equals(event.getType())) {
            String status = (String) event.getExtraData();
            // 当网络状态变化的时候请求startQuery接口
            if (status != null && status.equals("open")) {
                if (isNeedRestart && UserConfig.getUserInfo().getB3Key() != null && UserConfig.getUserInfo().getSessionKey() != null) {
                    L.i("检测到网络已打开且长连接处于关闭状态，需要启动长连接...");
                    Intent intent = new Intent(context, LongConnService.class);
                    intent.setAction(LongConnService.ACTION);
                    context.startService(intent);
                }
            }
        }
    }

    /**
     * 出现异常、session关闭后，接收事件进行长连接重连操作
     */
    public void onEventMainThread(LongConnectMessageEvent event) {

        if (event.getType() == LongConnectMessageEvent.TYPE_RESTART) {

            long currentTime = System.currentTimeMillis();

            // 票据有效的情况下进行重连长连接操作
            if (UserConfig.getUserInfo().getB3Key() != null && UserConfig.getUserInfo().getSessionKey() != null
                    && ((currentTime / 1000) < UserConfig.getUserInfo().getUnvalidSecs())) {
                // 等待2s后重新创建长连接
                SystemClock.sleep(1000);
                if (Config.isNetworkConnected(context)) {
                    L.i("出现异常情况，需要自动重连长连接...");
                    startConnect();
                } else {
                    isNeedRestart = true;
                    L.i("长连接出现异常，需要重新创建session会话...");
                }
            }
        } else if (event.getType() == LongConnectMessageEvent.TYPE_CLOSE) {
            L.i("收到session多次close的消息，此时需要关闭长连接，等待下次闹钟服务来启动...");
            closeConnect();
        }
    }


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

}
```

长连接创建成功之后需要重新拉取一次服务器端的长连接消息，并且这里的长连接做了容错处理，当长连接断了之后需要有重连机制，一直启动轮训服务，当长连接修复之后轮训服务退出。以上只是通过MINA框架实现的长连接操作的核心流程，还有一些长连接实现的操作细节这里就不做过多的说明。


###总结：###
基本上对于App来说长连接已经是标配了，产品开发人员可以根据具体的产品需求选择不同的实现方式，一般而言使用第三方的推送服务已经可以满足大部分的需求了，当然了若是相对技术有所追求的话也可以选择自己实现一套长连接服务，不过其中可能存在一些坑需要填，希望这里的长连接实现能够对大家对长连接实现上有所帮助。

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
