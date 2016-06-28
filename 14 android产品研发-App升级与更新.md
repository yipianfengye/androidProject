上一篇文章中我们讲解了android app中的轮训操作，讲解的内容主要包括：我们在App中使用轮训操作的情景，作用以及实现方式等。一般而言我们使用轮训操作都是通过定时任务的形式请求服务器并更新用户界面，轮训操作都有一定的使用生命周期，即在一定的页面中启动轮操作，然后在特定的情况下关闭轮训操作，这点需要我们尤为注意，我们还介绍了使用Timer和Handler实现轮训操作的实例，更多关于App中轮训操作的信息，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>

本文将讲解app的升级与更新。一般而言用户使用App的时候升级提醒有两种方式获得：

- 一种是通过App Store获取

- 一种是打开应用之后提醒用户更新升级

而更新操作一般是在用户点击了升级按钮之后开始执行的，这里的升级操作也分为两种形式：

- 一般升级

- 强制升级

**app升级操作：**

- App Store升级

在App Store中升级需要为App Store上传新版App，我们在新版本完成之后都会上传到App Store中，不同的应用市场审核的时间不同，一般除了第一次上传时间较长之外，其余的审核都是挺快的，一般不会超过半天（不排除例外情况奥），在审核完成之后就相当于完成了这个应用市场的发布了，也就是发布上线了。这时候如果用户安装了这个应用市场，那么就能看到我们的App有新版本的升级提醒了。

- 应用内升级

除了可以在应用市场升级，我们还可以在应用内升级，在应用内升级主要是通过调用服务器端接口获取应用的升级信息，然后通过获取的服务器升级应用信息与本地的App版本比对，若服务器下发的最新的App版本高于本地的版本号，则说明有新版本发布，那么我们就可以执行更新操作了，否则忽略掉即可。

应用内升级其实已经有好多第三方的SDK了，常见的友盟，百度App开发工具包都已经集成了升级的功能，部分SDK厂商还提供增量更新的功能。增量更新的内容不是我们这里的讨论重点，想了解更多增量更新的内容可参考：<a href="http://blog.csdn.net/hmg25/article/details/8100896">浅谈Android增量升级</a>

这里我们先简单介绍一下友盟的App升级功能，友盟其实已经有了App升级的API，我们只需要简单的调用即可。

- 友盟更新接口API

```
/**
 * 请求友盟更新API，判断是否弹出更新弹窗
 */
public static void updateVersion(final Activity mContext, final MainActivity.UpdateCallback updateCallback, final boolean isShow) {
        UmengUpdateAgent.setUpdateListener(new UmengUpdateListener() {
            @Override
            public void onUpdateReturned(int updateStatus, UpdateResponse updateInfo) {
                switch (updateStatus) {
                    //判断是否有新版本需要更新
                    case UpdateStatus.Yes: // has update
                        try {
                            //在线读取更新参数
                            String value = MobclickAgent.getConfigParams(mContext, "FORCE_UPDATE_MIXVERSION");
                            if (value != null && !value.trim().equals("")) {
                                int versionCode = Config.changeVersionNameToCode(value);
                                if (versionCode != 0) {
                                    String localVersionName = getVersionName(mContext);
                                    int localVersionCode = Config.changeVersionNameToCode(localVersionName);
                                    //判断当前版本号于友盟中的最低版本号，若当前版本号小于最低版本号，则强制更新，否则非强制更新
                                    if (localVersionCode <= versionCode) {
	                                    // 弹窗更新弹窗
                                        updateCallback.onUpdateSuccess(updateInfo);
                                    } else {
                                        UmengUpdateAgent.setUpdateAutoPopup(true);
                                        UmengUpdateAgent.showUpdateDialog(mContext, updateInfo);
                                    }
                                } else {
                                    UmengUpdateAgent.setUpdateAutoPopup(true);
                                    UmengUpdateAgent.showUpdateDialog(mContext, updateInfo);
                                }
                            } else {
                                UmengUpdateAgent.setUpdateAutoPopup(true);
                                UmengUpdateAgent.showUpdateDialog(mContext, updateInfo);
                            }
                        } catch (Exception e) {
                            e.printStackTrace();
                        }

                        break;
                    case UpdateStatus.No: // has no update
                        if (isShow) {
                            Config.showToast(mContext, "您当前使用的友友用车已是最新版本");
                        }
                        break;
                }
            }
        });
        UmengUpdateAgent.setUpdateAutoPopup(false);
        UmengUpdateAgent.forceUpdate(mContext);
        UmengUpdateAgent.setChannel(ChannelUtil.getChannel(mContext));
    }
```
以上是友盟的升级API，在调用之前需要先继承友盟的SDK，这样经过调用之后我们就可以通过友盟实现更新接口的提示功能了，默认的友盟提供了静默安装，更新提示弹窗，强制更新等几种，可以根据自身App的需求来确定更新的方式。

如果不喜欢使用第三方的更新方式，我们也可以通过调用服务器接口的方式实现自己的更新弹窗提示，主要的逻辑也是通过判断服务器下发的最新App版本号与本地版本号对比，若服务器端的App版本号大于本地的App版本号，则说明当前App不是最新的版本，需要升级，这里我们简单看一下友友用车中自定义的更新接口实现：

```
/**
     * 检测App是否需要更新
     *
     * @param mContext
     * @param isShow   若不需要更新是否需要弹出文案
     */
    public static void queryAppBaseVersionInfo(final Activity mContext, final boolean isOneUpdate, final boolean isShow) {
        try {
            // 若当前网络异常，则直接return
            if (!Config.isNetworkConnected(mContext)) {
                // 关闭进度条
                dismissProgress(isShow);
                return;
            }
            // 控制变量，App更新接口进程生命周期中只会调用一次
            if (isQueryAppUpdated && isOneUpdate) {
                return;
            }
            L.i("开始调用请求是否需要版本更新的接口....");
            ExtInterface.QueryAppBaseVersionInfoNL.Request.Builder request = ExtInterface.QueryAppBaseVersionInfoNL.Request.newBuilder();
            request.setClientChannel(CHANNEL_ANDROID);
            // 查询最新的版本信息，不需要传入版本号
            // request.setVersionCode(VersionUtils.getVersionName(mContext));
            NetworkTask task = new NetworkTask(Cmd.CmdCode.QueryAppBaseVersionInfo_VALUE);
            task.setBusiData(request.build().toByteArray());
            NetworkUtils.executeNetwork(task, new HttpResponse.NetWorkResponse<UUResponseData>() {
                @Override
                public void onSuccessResponse(UUResponseData responseData) {
                    if (responseData.getRet() == 0) {
                        try {
                            isQueryAppUpdated = true;
                            ExtInterface.QueryAppBaseVersionInfoNL.Response response = ExtInterface.QueryAppBaseVersionInfoNL.Response.parseFrom(responseData.getBusiData());
                            if (response.getRet() == 0) {
                                L.i("请求检测App是否更新接口成功，开始解析返回结果");
                                // 解析检测结果
                                parserUpdateResule(mContext, response, isShow);
                            } else {
                                if (isShow) {
                                    showDefaultNetworkSnackBar(mContext);
                                }
                            }
                        } catch (InvalidProtocolBufferException e) {
                            e.printStackTrace();
                            if (isShow) {
                                showDefaultNetworkSnackBar(mContext);
                            }
                        }
                    }
                }

                @Override
                public void onError(VolleyError errorResponse) {
                    L.e("请求检测更新接口失败....");
                    if (isShow) {
                        showDefaultNetworkSnackBar(mContext);
                    }
                }

                @Override
                public void networkFinish() {
                    L.i("请求检测更新接口完成....");
                    // 关闭进度条
                    dismissProgress(isShow);
                }
            });
        } catch (Exception e) {
            e.printStackTrace();
        }
    }
```
该接口只会在App打开时调用一次，判断App是否需要更新，然后在请求服务器成功之后，会解析请求结果，我们继续看一下我们的解析逻辑：

```
/**
     * 解析更新检查结果
     *
     * @param response
     */
    private static void parserUpdateResule(Activity mContext, ExtInterface.QueryAppBaseVersionInfoNL.Response response, boolean isShow) {
        if (mContext == null) {
            return;
        }
        // 判断是否需要更新
        ExtInterface.AppBaseVersionInfo appBaseVersionInfo = response.getAppBaseVersionInfo();
        // 若当前更新是否有效
        if (appBaseVersionInfo.getIsDel() == ENEFFECT) {
            return;
        }
        String updateVersionCode = appBaseVersionInfo.getVersionCode();
        int updateCode = changeVersionNameToCode(updateVersionCode);
        int localCode = changeVersionNameToCode(VersionUtils.getVersionName(mContext));
        // 本地应用版本号小于更新的应用版本号，则需要更新
        L.i("本地版本号：" + localCode + "  " + VersionUtils.getVersionName(mContext) + "  远程版本号：" + updateCode
                            + "  " + updateVersionCode);
        if (localCode < updateCode) {
            // 显示更新文案
            L.i("开始显示更新弹窗...");
            showUpdateDialog(mContext, appBaseVersionInfo);
        }
        // 不需要更新
        else {
            if (isShow) {
                Config.showToast(mContext, mContext.getResources().getString(R.string.about_new));
            }
        }
    }
```
解析更新接口信息的时候，会判断App的更新操作是普通更新还是强制更新，若是强制更新的话，则没有取消按钮，并且更新弹窗不可关闭。若是普通的更新的话则有暂不更新按钮，点击暂不更新更新弹窗会取消，但是当下次打开App的时候，弹窗提醒还是会弹窗。

普通更新包含暂不更新和立即更新两个按钮操作：
![这里写图片描述](http://img.blog.csdn.net/20160627223151601)

强制更新只有立即更新按钮，弹窗不可取消：
![这里写图片描述](http://img.blog.csdn.net/20160627223207332)


**app更新操作：**

app的更新操作就是下载App并安装了，下面我们还是分两部分看，应用市场的更新与应用内更新

- App store更新App

在应用市场中更新App很简单就是执行简单的下载操作，然后顺着App的提醒，一步步安装即可，这里没有什么需要注意的地方。

- 应用内更新

应用内更新操作主要是当用户点击了更新按钮之后执行的，下载，安装等逻辑，下面我们看一下友友用车应用内更新的实践。

应用内更新主要包含了：普通更新和强制更新两种，其中普通更新弹窗可以选择更新也可以选择忽略，而强制更新只能选择更新，并且更新弹窗不可取消。

下面的代码是执行下载操作的核心逻辑：

```
/**
     * 开始执行下载动作
     */
    private static void doDownLoad(final Activity mContext, String downloadUrl, final String actionButtonMsg, final boolean isFocusUpdate) {
        // 强制更新
        if (isFocusUpdate) {
            DownLoadDialog.updateRela.setVisibility(View.VISIBLE);
            DownLoadDialog.progressBar.setProgress(0);
            DownLoadDialog.progressBar.start();
            DownLoadDialog.updatePercent.setText("0%");
            DownLoadDialog.materialDialog.getPositiveButton().setEnabled(false);
            DownLoadDialog.materialDialog.getPositiveButton().setText("下载中");
        }
        Config.showToast(mContext, "开始下载安装包.......");
        // 删除下载的apk文件
        doDeleteDownApk(mContext);
        L.i("安装包下载地址：" + downloadUrl);
        DownloadManager.getInstance().cancelAll();
        DownloadManager.downloadId = DownloadManager.getInstance().add(DownloadManager.getDownLoadRequest(mContext, downloadUrl, new DownloadStatusListenerV1() {
            @Override
            public void onDownloadComplete(DownloadRequest downloadRequest) {
                L.i("onDownloadComplete_____...");
                // 设置按钮是否可点击
                showPositiveText(false, actionButtonMsg);
                if (isFocusUpdate) {
                    // 更新进度条显示
                    DownLoadDialog.updatePercent.setText("100%");
                    DownLoadDialog.progressBar.stop();
                } else {
                    String title = "正在下载友友用车...";
                    String content = "下载成功";
                    DownloadNotification.showNotification(mContext, title, content, DownloadNotification.notofyId);
                    // 关闭通知栏消息
                    UUApp.notificationManager.cancel(DownloadNotification.notofyId);
                }
                // 下载完成，执行安装逻辑
                doInstallApk(mContext);
                // 退出App
                UUApp.getInstance().exit();
            }

            @Override
            public void onDownloadFailed(DownloadRequest downloadRequest, int errorCode, String errorMessage) {
                L.i("onDownloadFiled______...");
                L.i("errorMessage:" + errorMessage);
                // 设置按钮是否可点击
                showPositiveText(false, actionButtonMsg);
                if (isFocusUpdate) {
                    // DownLoadDialog.progressBar.stop();
                    DownLoadDialog.updatePercent.setText("更新失败");
                } else {
                    String title = "正在下载友友用车...";
                    String content = "下载失败";
                    DownloadNotification.showNotification(mContext, title, content, DownloadNotification.notofyId);
                }
            }

            @Override
            public void onProgress(DownloadRequest downloadRequest, long totalBytes, long downloadedBytes, int progress) {
                if (lastProgress != progress) {
                    lastProgress = progress;
                    L.i("onProgress_____progress:" + progress + "  totalBytes:" + totalBytes + "  downloadedBytes:" + downloadedBytes);
                    // 设置按钮是否可点击
                    showPositiveText(true, actionButtonMsg);
                    // 强制更新则更新进度条
                    if (isFocusUpdate) {
                        String content = downloadedBytes * 100 / totalBytes + "%";
                        float result = progress / (float)100.00;
                        DownLoadDialog.progressBar.setProgress(result);
                        DownLoadDialog.updatePercent.setText(content);
                    } else {
                        String title = "正在下载友友用车...";
                        String content = downloadedBytes * 100 / totalBytes + "%";
                        DownloadNotification.showNotification(mContext, title, content, DownloadNotification.notofyId);
                    }
                }
            }
        }));
    }
```
这里的下载操作包含了三个回调方法：

- onDownloadComplete()

- onDownloadFailed()

- onProgress()

其中onDownlaodComplete方法在下载完成时回调，onDownloadFailed方法在下载失败是回调，而onProgress方法则用于刷新下载进程，我们在onProcess方法中更新通知栏下载进度，具体我们可以看一下更新通知栏消息的方法：

```
/**
     * 更新通知栏显示
     * @param title
     * @param content
     * @param notifyId
     */
    public static void showNotification(Activity mContext, String title, String content, int notifyId) {
        NotificationCompat.Builder mNotifyBuilder = new NotificationCompat.Builder(mContext)
                .setSmallIcon(R.mipmap.icon)
                .setContentTitle(title)
                .setContentText(content)
                .setSmallIcon(android.R.drawable.stat_sys_download);

        Notification notification = mNotifyBuilder.build();
        // notification.flags = Notification.FLAG_NO_CLEAR;
        UUApp.notificationManager.notify(notifyId, notification);
    }
```

而在onDownloadFailed方法中，执行的代码逻辑是提示用户下载失败，
而在onDownloadComplete方法中，执行安装下载apk文件的操作，我们可以继续看一下我们是如何执行安装逻辑的。

```
/**
     * 执行安装apk文件
     */
    private static void doInstallApk(Activity mContext) {
        Intent intent = new Intent(Intent.ACTION_VIEW);
        intent.addFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setDataAndType(Uri.fromFile(new File(DownloadManager.getApkPath(mContext))),
                "application/vnd.android.package-archive");
        mContext.startActivity(intent);
    }
```
这段代码会调用android的安装apk程序，这样我们就执行了下载文件的安装操作，不同的手机安装程序及展示界面略有不同。


**总结：**

- App升级操作分为两种，在应用市场提示升级和在应用内提示升级，而在应用内提示升级可以继承第三方升级API（如：友盟），也可以自己实现；

- 应用升级的提示主要逻辑是根据服务器端的APK版本号与本地的应用版本号对比，若服务器端的应用版本号高于本地版本号，则说明应用需要升级；

- 应用升级可以分为普通升级和强制升级两种，一般不太建议使用强制升级（用户体验很差），除非是一些严重的线上bug；

- App的更新操作包含下载与安装两部分，下载操作时可以选择继承第三方服务，也可以自己实现。



<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261">android产品研发（五）-->多渠道打包</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51581491">android产品研发（六）-->Apk混淆</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51587927">android产品研发（七）-->Apk热修复</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51598041">android产品研发（八）-->App数据统计</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51612429">android产品研发（九）-->App网络传输协议</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51655330">android产品研发（十）-->不使用静态变量保存数据</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51685310">android产品研发（十一）-->应用内跳转scheme协议</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
</br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
