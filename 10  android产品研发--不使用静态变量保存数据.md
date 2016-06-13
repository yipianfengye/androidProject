上一篇文章中我们讲解了Android中的几种常见网络协议：xml，json，protobuf等，以及各自的优缺点，一般而言主要我们的App涉及到了网络传输都会有这方面的内容，具体可根据项目的需求确定各自的网络传输协议。这里可参考<a href="http://blog.csdn.net/qq_23547831/article/details/51612429"> android产品研发（九）-->App网络传输协议</a>

而本文讲解的其实并不是一个技术方面，而是一个android产品研发过程中的技巧：尽量不使用静态变量保存核心数据。这是为什么呢？这是因为android的进程并不是安全的，包括application对象以及静态变量在内的进程级别变量并不会一直呆着内存里面，它会被kill掉，它真的有可能会被kill掉，真的真的，重要的事情说三遍。与大家普遍的看法不同之处在于，实际上app不会重新开始启动。Android系统会创建一个新的Application 对象，然后启动上次用户离开时的activity以造成这个app从来没有被kill掉得假象。而这时候静态变量等数据由于进程已经被杀死而被初始化，所以就有了我们的不推荐在静态变量（包括Application中保存全局数据静态数据）的观点。

最近我们的产品<a href="http://www.uucars.com/">友友用车</a>就遇到了这样的问题，下面我们详细的说明一下这个问题以及解决方案。在友友用车下单的过程中有一个当前行程的页面，该页面就是用户开车过程中需要展示的页面，主要用于展示当前用户的用车费用，行驶里程，用车时长，还车网点等信息。具体如下图：
![这里写图片描述](http://img.blog.csdn.net/20160613183250533)

而在当前行程页面中App端有一个轮训请求，大概每隔一分钟会请求一次服务器，用于更新用户的用车费用，行驶里程，用车时长还车网点等信息。用户可以在当前行程页面停留很长的时间。需要注意的是这里还有一个更换还车网点的按钮，点击这个按钮会跳转到更换还车网点的页面，从中会选择用户的常用网点，而这时候需要传递一个参数：用户的订单ID，此时用户的订单ID保存在了系统的静态变量中。

在前几天服务器端报了一个bug，说的是在当前行程页面点击更改换车网点请求常用网点时没有上传订单ID，而App端的代码是判断此时的订单ID是否为空，若不为空的话则上传.

```
if (!TextUtils.isEmpty(orderId)) {
                builder.setOrderId(orderId);
            }
```

可以看到，如果这时候我们的orderId为空，那么我们就不会上传orderId，而服务器端说我们没有上传orderId，可能的原因就是我们的orderId这时候为空了，那么这里的orderId是一个静态变量，而且我们没有手动的置空，为什么orderId会变为空了呢？好吧，看一下服务器端的用户请求日志。

下面的这一段日志是用户这段时间所有的网络请求日志，当然也包括更改还车地点的请求服务器的日志：
```
[log@iZ28tw9apzsZ UUAccess-App]$ cat CmdAccess4Stat.log.2016-06-08 | grep '2119489256'
2016-06-08 11:22:52,778 1100004 229BAEC4B95E4C6ABB83F26FB7A48C88        2119489256      117.136.0.134   1465356172778   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:58,409 1070002 BABEDA5E46494FB4B00A2E50FD2F8F4D        2119489256      117.136.0.134   1465356178409   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:58,409 1130001 5F9D626DB30D442880A67D1C3A2A1C67        2119489256      117.136.0.134   1465356178409   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:58,862 1070002 68767819033342E5A31D03956C3FA84C        2119489256      117.136.0.134   1465356178862   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:58,862 1110001 6EB7CFC572E7404782512BAA26FE9D10        2119489256      117.136.0.134   1465356178862   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:59,029 1110002 C1039D50827D40098E3654FAD1260404        2119489256      117.136.0.134   1465356179029   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:22:59,498 1050007 DD0198099023454C889522A148F82955        2119489256      117.136.0.134   1465356179498   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:23:00,168 1130002 5E2A776EBF8743869202FB86A419580A        2119489256      117.136.0.134   1465356180168   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:23:10,038 1010003 0311997A14544D85BAA0F942650EFA84        2119489256      117.136.0.134   1465356190038   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 11:23:39,098 1070002 4B495330AC804EC3932BEBB6B6586595        2119489256      117.136.0.134   1465356219098   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:32,322 1070003 5F3F2146CDF54662A4CD5E4849D07422        2119489256      123.118.169.6   1465365692322   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:32,953 1050007 29A5FF713B11481CAABBD9084145AAA1        2119489256      123.118.169.6   1465365692953   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:33,177 1070002 AFB545D58AF648B69328D1417E73DAE6        2119489256      123.118.169.6   1465365693177   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:37,157 1110001 5CD82483782E4AF98A5FD34F2C8FB96A        2119489256      123.118.169.6   1465365697157   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:02:52,246 1050007 AF177A8CDCEC4B97B7C0EAA92B0DD7F5        2119489256      123.118.169.6   1465365772246   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:03:52,277 1050007 778CA2EC92094513A16E6EDCBB0664F0        2119489256      123.118.169.6   1465365832277   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:13,395 1010003 2DFA5673D9104412B6C5621B9E9FE19A        2119489256      123.118.169.6   1465372093395   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:15,019 1010003 D95851D7A08F4227A91A445E12105FEE        2119489256      123.118.169.6   1465372095019   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:16,984 1100004 CB208E17087C422DB56DB681E774B7C9        2119489256      123.118.169.6   1465372096984   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:17,382 1110001 CBABF14385374911BDCAEA427C1C0780        2119489256      123.118.169.6   1465372097382   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:18,011 1110002 F1D79FF8481D43F0B8861B986B458EFD        2119489256      123.118.169.6   1465372098011   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:18,716 1050007 58BDBF796411497CBD1A7CCD2F3E21AA        2119489256      123.118.169.6   1465372098716   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:18,716 1110002 8CF8D68D0A4A420C8D83A78818DBD4B7        2119489256      123.118.169.6   1465372098716   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:51:01,855 1050007 284E391D94EA48C6877F81B94E8E51DD        2119489256      123.118.169.6   1465372261855   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:51:04,784 1010002 6085D946E4AA46C884E775DE9B40987E        2119489256      123.118.169.6   1465372264784   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:51:09,249 1050007 4147044DC8014FC4B64488FFC1A0D83C        2119489256      123.118.169.6   1465372269249   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
```

```
[log@iZ28foila6mZ UUAccess-App]$  cat CmdAccess4Stat.log.2016-06-08 | grep '2119489256'
2016-06-08 14:01:32,273 1100004 C6CC832BE038415CB6590D9174025FE9        2119489256      123.118.169.6   1465365692273   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:32,577 1070002 A23766F7F63C405080C3E7FEA4C6D1E1        2119489256      123.118.169.6   1465365692577   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:37,248 1110002 4C9A9F7521954E439C25B41CB4EB1E1A        2119489256      123.118.169.6   1465365697248   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:51,142 1010003 8EBB280697D349669A9C01925D2965E8        2119489256      123.118.169.6   1465365711142   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:01:51,955 1010002 0F35494F31ED475D93AC79F08D50CAF5        2119489256      123.118.169.6   1465365711955   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:06:44,272 1110001 2AF0F04090714B6FAAC15AD4E5F7924A        2119489256      123.118.169.6   1465366004272   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 14:06:44,365 1110002 728B5A82C759443A9DBBC91D3812C14F        2119489256      123.118.169.6   1465366004365   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:13,447 1100004 40AC6AFA4A5D4E75867A6459CAFA50AD        2119489256      123.118.169.6   1465372093447   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:17,025 1070003 3A2DAC87041E4B4BAD4A73596D5A1785        2119489256      123.118.169.6   1465372097025   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:17,758 1070002 BA4F5C07FB504882826DB2B99FA8CB5D        2119489256      123.118.169.6   1465372097758   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:48:18,015 1110001 CC2F250F823044298B85C23344D36B2A        2119489256      123.118.169.6   1465372098015   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:51:02,089 1070002 BF580DD1CA2F4F29800EE631C7940747        2119489256      123.118.169.6   1465372262089   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:51:03,518 1010003 9047E26C47C94F4C90A4FA9D460C2F54        2119489256      123.118.169.6   1465372263518   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:57:26,699 1050007 A283E7F6F70E4902946716474231F01A        2119489256      123.118.169.6   1465372646699   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 15:59:04,659 1050007 D6AF179627014BBF885C41292B59F26E        2119489256      123.118.169.6   1465372744659   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:00:45,601 1050007 D8029E9094B44737ADB47C94803D2753        2119489256      123.118.169.6   1465372845601   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:00:45,890 1070002 0475136678324CAABD0D18637ED97D3C        2119489256      123.118.169.6   1465372845890   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:01:12,120 1110001 6E784DF19F3C479183A8D06FD84DBF1B        2119489256      123.118.169.6   1465372872120   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:01:48,705 1050007 AA87FE1B960B40889A9F0511187D2F3D        2119489256      123.118.169.6   1465372908705   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:02:03,148 1010003 6801B16D60364BC8826C7AB9E7D29051        2119489256      123.118.169.6   1465372923148   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:02:03,501 1010002 CCA10A3EE4D9415384A761E623FE9959        2119489256      123.118.169.6   1465372923501   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:02:05,096 1050007 837721D2E671453888E170C95849E926        2119489256      123.118.169.6   1465372925096   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:02:09,446 1070002 D161AA4123354BA4A653582B65C8077E        2119489256      123.118.169.6   1465372929446   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:04:04,784 1050007 3B8B45A4066747AD8E43DD197B47C223        2119489256      123.118.169.6   1465373044784   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:05:04,778 1050007 18AA8CE0E8C446928B230D54F068D2E6        2119489256      123.118.169.6   1465373104778   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:20,860 1070003 BC4E84F58F654CB49FD76D8BA4854D85        2119489256      117.136.0.154   1465376180860   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:20,883 1100004 184A5E0784F94A64AE5D5D7DD6E6FB86        2119489256      117.136.0.154   1465376180883   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:21,291 1070002 25738C17D2314B4382089F600EF3BEB9        2119489256      117.136.0.154   1465376181291   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:22,233 1070002 FDACF66E84DC4FF696E2FC61C368DAB0        2119489256      117.136.0.154   1465376182233   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:25,967 1110002 B5B3CC24758B4AF7AB6F3FD6185D6213        2119489256      117.136.0.154   1465376185967   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:29,674 1050003 3EB8C2993802494998603A6FBCC4243A        2119489256      117.136.0.154   1465376189674   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:35,833 1070002 655CDEEDF98E4718A17431AB9B90060C        2119489256      117.136.0.154   1465376195833   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:57,419 1050004 FD6EEDA543744A3998FB7B1E4CCFABD2        2119489256      117.136.0.154   1465376217419   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
2016-06-08 16:56:57,837 1070002 A4BE993338094CFDBB28C1C7C72BF0C6        2119489256      117.136.0.154   1465376217837   MI 4LTE ANDROID xiaomi  2.2.0   c671b0b2b6de7aacdc2eabfa93ed4601        865372024757509
```

这就是出现问题的用户的网络请求日志，需要说明的是后台的日志分为两个节点，所以用户的请求日志信息被随机的分到了两个节点中，可以按照时间将这两段日志信息看成是一段日志信息。

可以发现在2016-06-08 14:00:00的时候用户每个一分钟执行一次请求1050007请求，而这里的1050007就是当前行程页面的轮训请求，大概每隔一分钟执行一次，然后在2016-06-08 14:03的的时候就不开始执行了，这应该是用户锁屏之后小米手机限制后台的网络请求，而这时候直接到了15：48分钟，这时候用户开始请求1010003接口，该接口就是用户请求更换网点时请求的接口，但是这时候我们发现用户同时调用了：1100004接口，而这个接口只有在Application的onCreate方法中调用了，所以这说明这时候应用进程已经重新启动了，但是这时候用户还是在当前行程页面，并且用户点击了更换还车网点按钮，而这时候由于点击更换还车网点需要静态变量orderId，而这时候上传的orderId为空，所以这时候的进程静态变量已经被初始化了。

所以综上可以还原一下bug的出现流程：

- 用户在14：00的时候停留在当前形成页面；

- 用户锁屏，系统限制后台网络请求，轮训操作被限制；

- 系统内存吃紧，用户应用进程被杀死，进程的静态变量被初始化，activity界面被保留；

- 用户打开屏幕，点击更换还车网点，这时候由于进程已经被杀死，静态变量被初始化，所以上报给服务器的orderId为空

- 在用户打开屏幕的同时，系统重启应用进程，造成应用进程没有被杀死的假象；


好吧，既然已经知道了bug出现的原因，那么就好解决了，既然在内存中保存数据可能被系统杀死，那么我们可以有针对性的：

- 直接将数据通过intent传递给 Activity 。

- 使用官方推荐的几种方式将数据持久化到磁盘上。

- 在使用数据的时候总是要对变量的值进行非空检查。

最后我们决定将数据持久化，具体而言使用SharedPreferences来保存数据，相对来说这种方式比较简单。下面是使用sharedpreferences保存数据的静态工具方法：

```
/**
     * 获取SP中保存的订单ID
     *
     * @param context
     * @return
     */
    public static String getOrderId(Context context) {
        if (context == null) {
            L.i("从sp中获取orderId失败,context对象为空...");
            return "";
        }
        SharedPreferences sp = context.getSharedPreferences("morder", Context.MODE_PRIVATE);
        return sp.getString("morderid", "");
    }

    /**
     * 向Sp中保存orderId信息
     * @param context
     * @param orderId
     */
    public static void setOrderId(Context context, String orderId) {
        if (context == null) {
            L.i("向sp中设置orderId失败,context对象为空...");
            return;
        }
        // 若当前orderId为null,则设置orderId为""
        if (orderId == null) {
            orderId = "";
        }

        SharedPreferences sp = context.getSharedPreferences("morder", Context.MODE_PRIVATE);
        boolean isCommitSuccess = sp.edit().putString("morderid", orderId).commit();
        if (!isCommitSuccess) {
            sp.edit().putString("morderid", orderId).commit();
        }
    }

    /**
     * 清空sp中的orderId
     * @param context
     */
    public static void clearOrderId(Context context) {
        if (context == null) {
            L.i("向sp中设置orderId失败,context对象为空...");
            return;
        }

        SharedPreferences sp = context.getSharedPreferences("morder", Context.MODE_PRIVATE);
        sp.edit().putString("morderid", "").commit();
    }
```
这样我们在使用order数据的时候可以通过sharedpreferences方法来获取。这样静态变量数据被销毁的情况就不会出现了，但是需要注意的是，当数据持久化的时候一定要在app启动的时候清空。

**总结：**
在实际的android开发过程中我们不建议使用静态变量传递数据，这样会因为进程被杀死而使静态变量初始化，我们可以使用其他数据传输方式：

- 直接将数据通过intent传递给 Activity 。

- 使用官方推荐的几种方式将数据持久化到磁盘上。

- 在使用数据的时候总是要对变量的值进行非空检查。


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
