去年一整年android社区中刮过了一阵热修复的风，各大厂商，逼格大牛纷纷开源了热修复框架，恩，产品过程中怎么可能没有bug呢？重新打包上线？成本太高用户体验也不好，咋办？上热修复呗。

好吧，既然要开始上热修复的功能，那么就得调研一下热修复的原理。下面我将分别讲述一下热修复的原理，各大热修复框架的比较，以及自身产品中热修复功能的实践。

- 热修复的原理

1. 通过更改dex加载顺序实现热修复
最新github上开源了很多热补丁动态修复框架，大致有：
<a href="https://github.com/dodola/HotFix">HotFix</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://github.com/jasonross/Nuwa">Nuwa</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://github.com/bunnyblue/DroidFix">DroidFix</a>
上述三个框架呢，根据其描述，原理都来自：
<a href="https://mp.weixin.qq.com/s?__biz=MzI1MTA1MzM2Nw==&mid=400118620&idx=1&sn=b4fdd5055731290eef12ad0d17f39d4a&scene=1&srcid=1106Imu9ZgwybID13e7y2nEi#wechat_redirect">安卓App热补丁动态修复技术介绍</a>，以及<a href="http://my.oschina.net/853294317/blog/308583">Android dex分包方案</a>，其核心原理就是通过更改含有bug的dex文件的加载顺序。在dex的加载中，若以找到方法则不会继续查找，所以如果能让修复之后的方法在含有bug的方法之前加载就能达到修复bug的目的，而这个框架都是基于这个思路实现的，当然了具体实现过程中有很多坑，大家可以参考一下上面提到的两篇文章。

2. 通过Native替换方法指针的方式实现热修复
这里主要是阿里开源的两个热修复框架：<a href="https://github.com/alibaba/dexposed">Dexpost</a>&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;&nbsp;<a href="https://github.com/alibaba/AndFix">AndFix</a>都是通过Native层使用指针替换的方法替换bug，达到修复bug的目的的，具体可参考其github文章。


- 各大热修复框架的比较

这里暂时比较了Dexpost，AndFix，HotFix，Nuwa，DroidFix等框架。

Dexpost：（未测试）
1）原理：在底层虚拟机运行时hoop方法；
2）地址：https://github.com/alibaba/dexposed；
3）缺点：适配方面存在一些问题，目前不支持android6.0，5,1；art运行时；
4）优点：无需重启就可以达到修复bug的目的；

AndFix：（已测试）
1）原理：在Native层使用指针替换的方法替换bug方法，达到修复bug的目的；
2）地址：https://github.com/alibaba/AndFix；
3）缺点：底层替换，稳定性方面可能需要实际检测；
4）优点：无需中期就可以达到修复bug的目的；

HotFix：（个人维护，可能存在坑）
1）原理：通过替换类加载器中bugclass，达到修复bug的目的；
2）地址：https://github.com/dodola/HotFix；
3）缺点：需要重新启动才可以修复bug；
4）优点：java运行层修复，稳定性较好；

Nuwa：（个人维护，可能存在坑）
1）原理：通过替换类加载器中bugclass，达到修复bug的目的；
2）地址：https://github.com/jasonross/Nuwa；
3）缺点：需要重新启动才可以修复bug；
4）优点：java运行层修复，稳定性较好；

DroidFix：（个人维护，可能存在坑）
1）原理：通过替换类加载器中bugclass，达到修复bug的目的；
2）地址：https://github.com/bunnyblue/DroidFix；
3）缺点：需要重新启动才可以修复bug；
4）优点：java运行层修复，稳定性较好；

**其他的方面：**
Dexposed不支持Art模式（5.0+），且写补丁有点困难，需要反射写混淆后的代码，粒度太细，要替换的方法多的话，工作量会比较大。

AndFix支持2.3-6.0，但是不清楚是否有一些机型的坑在里面，毕竟jni层不像java曾一样标准，从实现来说，方法类似Dexposed，都是通过jni来替换方法，但是实现上更简洁直接，应用patch不需要重启。但由于从实现上直接跳过了类初始化，设置为初始化完毕，所以像是静态函数、静态成员、构造函数都会出现问题，复杂点的类Class.forname很可能直接就会挂掉。

ClassLoader方案支持2.3-6.0，会对启动速度略微有影响，只能在下一次应用启动时生效，在空间中已经有了较长时间的线上应用，如果可以接受在下次启动才应用补丁，是很好的选择。

总的来说，在兼容性稳定性上， ClassLoader方案很可靠 ，如果需要应用 不重启就能修复 ，而且方法足够简单，可以使用 AndFix ，而 Dexposed由于还不能支持art，所以只能暂时放弃，希望开发者们可以改进使它能支持art模式，毕竟xposed的种种能力还是很吸引人的。


- 热修复实践

最终我们App的热修复方案选择的是AndFix，原因有三：
（1）AndFix支持android2.3-6.0，所以在机型上的是适配上是没问题的；
（2）AndFix是由阿里开源的，并且持续维护中，目前不少公司已经使用其作为自身App的热修复方案；
（3）通过修改Dex加载顺序的方式实现热修复需要重新启动App，并且相应的开源框架多多少少存在着问题，没有持续的维护；

因此我们最终选择了AndFix作为我们的开源方案。具体的AndFix集成方式可参考<a href="https://github.com/alibaba/AndFix">github中AndFix的介绍</a>

这里简单介绍一下具体的继承流程
（1）在App的Application的onCreate方法中执行AndFix的初始化操作；
（2）判断服务器端是否有可更新的热修复差异包
（3）若无则直接退出，若有则下载并执行修复动作
（4）修复完成之后删除下载的补丁差异包
（5）在判断服务器端是否有可更新的补丁包的时候可添加灰度，如版本，渠道，用户等，实现对补丁包定制化的修复

另外需要说明的是：若一个版本中存在着多个bug，则一般的都是让后一个补丁包覆盖前一个补丁包，并删除前一个补丁包，简单来说就是对于每一个版本至多有一个补丁包。

最后贴上App端AndFix的实现源码：

```
/**
 * Created by aaron on 2016/3/7.
 * 主要用于实现热修复逻辑
 *   采用阿里巴巴开源框架-andfix
 *
 */
public class AndfixManager {
    public static final String TAG = AndfixManager.class.getSimpleName();

    // AndfixManager单例对象
    private static AndfixManager instance = null;
    // 补丁文件名称
    public static final String PATCH_FILENAME = "/patchname.apatch";

    public static PatchManager patchManager = null;
    private AndfixManager() {}

    /**
     * 线程安全之懒汉模式实现单例模型
     * @return
     */
    public static synchronized AndfixManager getInstance() {
        return instance == null ? new AndfixManager() : instance;
    }

    /**
     * 执行andfix初始化操作
     */
    public static void init(Context mContext) {
        if (mContext == null) {
            L.i("初始化热修复框架，参数错误！！！");
            return;
        }
        patchManager = new PatchManager(mContext);
        // 初始化patch版本，这里初始化的是当前的App版本；
        patchManager.init(VersionUtils.getVersionName(mContext));
        // 加载已经添加到PatchManager中的patch
        patchManager.loadPatch();


        downLoadAndAndPath(mContext);

    }

    /**
     * 请求服务器获取补丁文件并加载
     */
    public static void downLoadAndAndPath(final Context mContext) {
        // 请求服务器获取差异包
        ExtInterface.GetShContent.Request.Builder request = ExtInterface.GetShContent.Request.newBuilder();

        // 获取本地保存的补丁包版本号
        final String patchVersion = AndfixSp.getPatchVersion(mContext);
        L.i(TAG, "patchVersion:" + patchVersion);
        if (!TextUtils.isEmpty(patchVersion)) {
            request.setShVersion(patchVersion);
        } else {
            request.setShVersion("0");
        }
        NetworkTask task = new NetworkTask(Cmd.CmdCode.GetShContent_SSL_VALUE);
        task.setBusiData(request.build().toByteArray());
        NetworkUtils.executeNetwork(task, new HttpResponse.NetWorkResponse<UUResponseData>() {
            @Override
            public void onSuccessResponse(UUResponseData responseData) {
                if (responseData.getRet() == 0) {
                    try {
                        ExtInterface.GetShContent.Response response = ExtInterface.GetShContent.Response.parseFrom(responseData.getBusiData());
                        // 若返回成功，则更新脚本下载补丁包
                        if (response.getRet() == 0) {
                            ByteString zipDatas = response.getContent();
                            // 数据解压缩
                            byte[] oriDatas = GZipUtils.decompress(zipDatas.toByteArray());
                            String patchFileName = mContext.getCacheDir() + PATCH_FILENAME;
                            L.i(TAG, "patchFileName:" + response.getShVersion());
                            // 将byte数组数据写入文件
                            boolean boolResult = getFileFromBytes(patchFileName, oriDatas);
                            // 写入文件成功则加载
                            if (boolResult) {
                                patchManager.removeAllPatch();
                                patchManager.addPatch(patchFileName);

                                // 保存补丁版本号
                                AndfixSp.putPatchVersion(mContext, response.getShVersion());
                                // 删除补丁文件
                                File files = new File(patchFileName);
                                if (files.exists()) {
                                    files.delete();
                                }
                            }

                        } else {
                            // -1 请求失败
                            // 1 请求成功，但是没有更新版本的脚本
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                    }
                }
            }

            @Override
            public void onError(VolleyError errorResponse) {
            }

            @Override
            public void networkFinish() {
            }
        });

    }


    /**
     * 根据数组获取文件
     * @param path
     * @param oriDatas
     */
    public static boolean getFileFromBytes(String path, byte[] oriDatas) {
        boolean result = false;
        if (TextUtils.isEmpty(path)) {
            return result;
        }
        if (oriDatas == null || oriDatas.length == 0) {
            return result;
        }

        try {
            FileOutputStream fos = new FileOutputStream(path);
            fos.write(oriDatas);
            fos.close();
            return true;
        } catch (Exception e) {
            e.printStackTrace();
            return false;
        }

    }
}
```

**总结：**
android的热修复原理大体上分为两种，其一是通过dex的执行顺序实现Apk热修复的功能，但是其需要将App重启才能生效，其二是通过Native修改函数指针的方式实现热修复，有兴趣的同学可以深入研究。
