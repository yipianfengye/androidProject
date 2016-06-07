上一篇文章中我们介绍了android社区中比较火的热修复功能，并介绍了目前的几个比较流行的热修复框架，以及各自的优缺点，同时也介绍了一下自身项目中对热修复功能的实践。目前主流的热修复原理上其实分为两种，一种是通过利用dex的加载顺序实现热修复功能，一种是通过native层实现指针替换实现热修复功能，两种各有利弊可以根据自身产品的需要选择不同的方案。

而文本将要介绍一下android产品中另一项基础功能-数据统计。App数据统计的意义在于通过统计用户的行为方式有针对性的更新展示算法，根据用户的行为习惯更新产品功能等等，具体而言当我们开发好App后就会把它发到应用市场上，但是目前有很多的应用市场(如，豌豆荚，应用宝，安卓市场等)，那么问题来了，假如我们想统计我们开发的应用的下载次数，就必须使用统计吧，而且它不仅可以统计我们的应用的下载量，启动次数，还可以统计页面访问量、查看程序的bug等等，所以相对于项目而言产品由于存在着持续的迭代与用户体验，所以做好数据统计工作是一项必不可少的工作。

本文中数据统计主要介绍两种：第三方统计服务和自己实现数据统计功能。

相对而言这两种方式各有利弊，第三方统计服务简单、方便、统计范围广，但是数据保存在第三方，对于一些数据比较敏感的App可能不太喜欢这种方式，而自己实现数据统计功能主要优点就是安全可定制化，当然了缺点也是显而易见的，就是繁琐，复杂。

下面我分别就这两种数据统计方式做一下简单的介绍。

- 第三方统计服务

一般情况下App中都是使用第三方数据统计服务的，友盟、百度统计等等，这里简单介绍一下友盟统计，其余的都是大同小异。

<a href="http://mobile.umeng.com/">友盟统计官网</a>
友盟统计中不但有统计相关功能，还提供了错误分析，社会化分享，推送，即时通讯等等，当然这里我们重点分析的是其数据统计功能。

在友盟官网中我们可以看到其对统计服务的介绍，这里我们大概的看一下：
![这里写图片描述](http://img.blog.csdn.net/20160607161417963)
可以发现我们提供了用户统计，渠道统计，页面访问路径统计，点击事件统计等等。
那么我们如何才能继承友盟的统计功能的，其在SDK文档介绍中已经做了详细介绍，大概的集成流程是下载友盟SDK jar包，然后引用，并在相应的Activity/Fragment的生命周期方法中调用相关的API。

- 自己实现数据统计功能

对于一些数据敏感的App来说可能自己实现部分数据统计功能是一个比较不错的选择，当初笔者也做过类似的功能，其原理就是参考友盟的数据统计在App打开的时候执行数据上报功能。

这里简单介绍一下自己实现的数据统计功能的流程：
（1）数据统计上报主要分为两中方式，网络请求上报和文件信息上报；
（2）网络请求上报就是直接调用网络请求若请求失败，则将上报信息保存至本地文件中；
（3）本地上报文件设置阙值和间隔时间，若距离上次上报时间大于阙值或者是文件大小大于阙值则执行上报操作，若上报成功则删除本地数据文件；
（4）本地数据文件在上报之前有加密和压缩操作；
（5）在App打开的时候执行一次数据上报操作，在特定的操作下可以执行上报操作；
（6）上报操作是在Service服务中执行，避免上报操作被由于进程被杀死而中断；

最后贴上为了实现数据上报而自定义的数据统计服务：

```
/**
 * Created by aaron on 2016/4/8.
 * desc:UU打点service
 */
public class UUPointService extends Service {
    @Override
    public void onCreate() {
        super.onCreate();
    }

    @Override
    public int onStartCommand(Intent intent, int flags, int startId) {
        /**
         * desc:该service为APP打点service
         * 主要实现两个后台服务：
         * （1）内存数据上报；
         * （2）文件数据上报；
         */
        if (intent != null && intent.getStringExtra(UUPoint.LOADTYPE) != null && intent.getStringExtra(UUPoint.LOADTYPE).trim().equals(UUPoint.LOADDATA)) {
            if (UUPoint.writeMap.size() > 0) {
                //(1)转换数据
                final List<String> strList = new ArrayList<String>();
                Iterator<Map.Entry<String, Integer>> iterator = UUPoint.writeMap.entrySet().iterator();
                while (iterator.hasNext()) {
                    StringBuffer sb = new StringBuffer();
                    Map.Entry<String, Integer> entry = iterator.next();
                    String key = entry.getKey();
                    Integer count = entry.getValue();
                    sb.append(key).append(",").append(count);
                    strList.add(sb.toString());
                }
                if (strList.size() > 0) {
                    // 上传服务器数据上传服务器数据
                    SystemCommon.ReportLogBatch.Builder builder = SystemCommon.ReportLogBatch.newBuilder();
                    builder.addAllLogLine(strList);
                    SystemInterface.ReportLogInfoRequest.Builder request = SystemInterface.ReportLogInfoRequest.newBuilder();
                    request.setLogData(ByteString.copyFrom(GZipUtils.compress(builder.build().toByteArray())));// 压缩数据包
                    NetworkTask task = new NetworkTask(CmdCodeDef.CmdCode.ReportLogInfo_VALUE);
                    task.setBusiData(request.build().toByteArray());
                    NetworkUtils.executeNetwork(task, new HttpResponse.NetWorkResponse<UUResponseData>() {
                        @Override
                        public void onSuccessResponse(UUResponseData responseData) {
                            if (responseData.getRet() == 0) {
                                try {
                                    SystemInterface.ReportLogInfoResponse response = SystemInterface.ReportLogInfoResponse.parseFrom(responseData.getBusiData());
                                    if (response.getRet() == 0) {
                                        /*MLog.i("tab", "########################");
                                        for (String str : strList) {
                                            MLog.i("tab", str);
                                        }*/
                                        UUPoint.writeMap.clear();
                                        UUPointUtils.setLastPointTime(UUPointService.this);
                                        /*MLog.i("tab", "上报数据完成################");*/
                                    } else {
                                        UUPoint.saveWriteToIO(UUPointService.this);// 将写map中的数据持久化到IO
                                    }
                                } catch (Exception e) {
                                    UUPoint.saveWriteToIO(UUPointService.this);// 将写map中的数据持久化到IO
                                }
                            } else {
                                UUPoint.saveWriteToIO(UUPointService.this);// 将写map中的数据持久化到IO
                            }
                        }

                        @Override
                        public void onError(VolleyError errorResponse) {
                            MLog.i("tab", "保存到本地文件");
                            UUPoint.saveWriteToIO(UUPointService.this);
                        }

                        @Override
                        public void networkFinish() {
                        }
                    });
                }
            }
        }
        // 上传文件
        else if (intent != null && intent.getStringExtra(UUPoint.LOADTYPE) != null && intent.getStringExtra(UUPoint.LOADTYPE).trim().equals(UUPoint.LOADFILE)) {
            File file = new File(Config.CountFile);
            List<File> fileList = Arrays.asList(file.listFiles());
            if (fileList != null && fileList.size() > 0) {
                //按文件名称排序
                Collections.sort(fileList, new Comparator<File>() {
                    @Override
                    public int compare(File lhs, File rhs) {
                        return rhs.getName().compareTo(lhs.getName());
                    }
                });
                // 上传文件
                for (final File files : fileList) {
                    // 先解密，在使用票据加密
                    FileEncrypter.decrypt(files, UUPoint.secretKey);// 解密
                    if (Config.isNetworkConnected(UUPointService.this)) {
                        NetworkTask networkTask = new NetworkTask(CmdCodeDef.CmdCode.ReportLogFile_VALUE);
                        networkTask.setUploadDataFile(true);// 上传大点数据文件
                        networkTask.setBusiData(UUPointUtils.getBytesFromFile(files));
                        NetworkUtils.executeNetwork(networkTask, new HttpResponse.NetWorkResponse<UUResponseData>() {
                            @Override
                            public void onSuccessResponse(UUResponseData responseData) {
                                if (responseData.getRet() == 0) {
                                    try {
                                        SystemInterface.ReportLogInfoResponse response = SystemInterface.ReportLogInfoResponse.parseFrom(responseData.getBusiData());
                                        if (response.getRet() == 0) {
                                            // 上传成功, 删除加密文件
                                            files.delete();
                                            MLog.i("tab", "上传文件完成" + files.getName() + "######################");
                                        } else {
                                            FileEncrypter.encrypt(files, UUPoint.secretKey);// 上传失败, 重新加密文件
                                        }
                                    } catch (Exception e) {
                                        e.printStackTrace();
                                        FileEncrypter.encrypt(files, UUPoint.secretKey);// 上传失败, 重新加密文件
                                    }
                                } else {
                                    FileEncrypter.encrypt(files, UUPoint.secretKey);// 上传失败, 重新加密文件
                                }
                            }

                            @Override
                            public void onError(VolleyError errorResponse) {
                                MLog.i("tab", errorResponse.toString());
                                FileEncrypter.encrypt(files, UUPoint.secretKey);// 上传失败, 重新加密文件
                            }

                            @Override
                            public void networkFinish() {
                            }
                        });
                    }
                }
            }
        }

        return super.onStartCommand(intent, flags, startId);
    }

    @Override
    public void onDestroy() {
        super.onDestroy();
    }

    @Override
    public IBinder onBind(Intent intent) {
        return null;
    }
}
```

**总结：**
一般而言直接使用第三方统计服务就已经可以满足绝大部分的业务需求了，当然了如果数据比较敏感也可以自己实现数据上报统计的功能。

另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261">android产品研发（五）-->多渠道打包</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51581491">android产品研发（六）-->Apk混淆</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51587927"> android产品研发（七）-->Apk热修复</a>
