前面一篇文章中我们讲解了android里面的多渠道打包，对于大型的app来说，几百个上千个渠道包都是很正常的事，所以效率定制化是一件很重要的事。主要讲解了三种多渠道打包方式，并分析了其各自的利弊，在各自产品多渠道打包的时候，可以根据自身的产品需求选择相应的打包方式。

而本文主要讲解Apk的混淆，这里的混淆分为两种代码混淆和资源文件混淆。实际的产品研发中为了防止自己的劳动成果被别人窃取，混淆代码能有效防止apk文件被反编译，进而查看源代码。说来惭愧，作为互联网创业公司的我们也确实对竞品Apk反编译研究过，如果Apk混淆之后确实对理解源码的业务流程造成了困扰，这也从侧面说明了Apk混淆的重要性。

所以对于android apk安装文件来说如何混淆代码实现对apk文件的保护是一个很重要的问题，而android提供了Progurd方式来混淆apk中的代码，其核心的逻辑是在代码层将一些易懂的源代码类名，方法名称替换成毫无意义的a、b、c、d...，这样当别人反编译出你的Apk文件时，看到的源代码也无法还原其本身的逻辑。

**下面我们将分别介绍代码混淆与资源文件混淆具体实践。**

- 代码混淆-Progurd

下面来总结以下混淆代码的步骤： 

1. 在android studio的android项目中找到module的gradle配置文件，添加proguard配置

```
buildTypes {
        debug {
            // 显示Log
            buildConfigField "boolean", "LOG_DEBUG", "true"
            //混淆
            minifyEnabled false
            //Zipalign优化
            zipAlignEnabled true
            // 移除无用的resource文件
            shrinkResources true
            //加载默认混淆配置文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //签名
            signingConfig signingConfigs.debug
        }
        release {
            // 不显示Log
            buildConfigField "boolean", "LOG_DEBUG", "false"
            //混淆
            minifyEnabled true
            //Zipalign优化
            zipAlignEnabled true
            // 移除无用的resource文件
            shrinkResources true
            //加载默认混淆配置文件
            proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            //签名
            signingConfig signingConfigs.relealse
        }
    }
```

2. 找到项目中的proguard-rules.pro文件，该文件就是我们的混淆配置文件

![这里写图片描述](http://img.blog.csdn.net/20160604095341131)

3.编写proguard-rules.pro文件，添加混淆配置

（1）proguard混淆语法

```
-libraryjars class_path 应用的依赖包，如android-support-v4  
-keep [,modifier,...] class_specification 这里的keep就是保持的意思，意味着不混淆某些类 
-keepclassmembers [,modifier,...] class_specification 同样的保持，不混淆类的成员  
-keepclasseswithmembers [,modifier,...] class_specification 不混淆类及其成员  
-keepnames class_specification 不混淆类及其成员名  
-keepclassmembernames class_specification 不混淆类的成员名  
-keepclasseswithmembernames class_specification 不混淆类及其成员名  
-assumenosideeffects class_specification 假设调用不产生任何影响，在proguard代码优化时会将该调用remove掉。如system.out.println和Log.v等等  
-dontwarn [class_filter] 不提示warnning  
```

（2）混淆原则

```
jni方法不可混淆
反射用到的类不混淆(否则反射可能出现问题)
AndroidMainfest中的类不混淆，四大组件和Application的子类和Framework层下所有的类默认不会进行混淆
Parcelable的子类和Creator静态成员变量不混淆，否则会产生android.os.BadParcelableException异常
使用GSON、fastjson等框架时，所写的JSON对象类不混淆，否则无法将JSON解析成对应的对象
使用第三方开源库或者引用其他第三方的SDK包时，需要在混淆文件中加入对应的混淆规则
有用到WEBView的JS调用也需要保证写的接口方法不混淆
```

（3）第三方库的混淆原则

```
一般的第三方库都有自身的混淆方案，可直接引用其自身的混淆配置即可
若无混淆配置，一般的可配置不混淆第三方库
```

（4）最后帖上我们项目中实际的混淆方案

```
# Glide图片库的混淆处理
-keep public class * implements com.bumptech.glide.module.GlideModule
-keep public enum com.bumptech.glide.load.resource.bitmap.ImageHeaderParser$** {
    **[] $VALUES;
    public *;
}

-optimizationpasses 5
-dontusemixedcaseclassnames
-dontskipnonpubliclibraryclasses
-dontpreverify
-verbose
-optimizations !code/simplification/arithmetic,!field/*,!class/merging/*

# 高德地图混淆脚本
-keep class com.android.support.**{ *; }
-keep interface android.support.v4.app.**{ *; }
-keep public class * extends android.support.v4.**
-keep public class * extends android.app.Fragment

-dontwarn com.amap.api.**
-dontwarn com.a.a.**
-dontwarn com.autonavi.**
-keep class com.amap.api.** {*;}
-keep class com.autonavi.** {*;}
-keep class com.a.a.** {*;}

# Gson混淆脚本
-keep class com.google.gson.stream.** {*;}
-keep class com.youyou.uuelectric.renter.Network.user.** {*;}

# butterknife混淆脚本
-dontwarn butterknife.internal.**
-keep class **$$ViewInjector { *; }
-keepnames class * { @butterknife.InjectView *;}

# -------------系统类不需要混淆 --------------------------
-keep public class * extends android.app.Fragment
-keep public class * extends android.app.Activity
-keep public class * extends android.app.Application
-keep public class * extends android.app.Service
-keep public class * extends android.content.BroadcastReceiver
-keep public class * extends android.content.ContentProvider
-keep public class * extends android.app.backup.BackupAgentHelper
-keep public class * extends android.preference.Preference
-keep public class * extends android.support.**
-keep public class com.android.vending.licensing.ILicensingService

-keepclasseswithmembernames class * { # 保持native方法不被混淆
    native <methods>;
}
-keepclasseswithmembernames class * { # 保持自定义控件不被混淆
    public <init>(android.content.Context, android.util.AttributeSet);
}
-keepclasseswithmembernames class * { # 保持自定义控件不被混淆
    public <init>(android.content.Context, android.util.AttributeSet, int);
}
-keepclassmembers enum * { # 保持枚举enum类不被混淆
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
-keep class * implements android.os.Parcelable { # 保持Parcelable不被混淆
  public static final android.os.Parcelable$Creator *;
}

# --------- 忽略异常提示 --------------------
-dontwarn butterknife.internal.**
-dontwarn com.alipay.**
-dontwarn com.mikepenz.**
-dontwarn org.apache.**
-dontwarn com.amap.**
-dontwarn com.android.volley.**
-dontwarn com.rey.**
-dontwarn com.testin.**
-dontwarn jp.wasabeef.**

# ---------- 保持代码 --------------
-keep class com.youyou.uuelectric.renter.Utils.** {*;}
-keep class it.neokree.** {*;}
-keep class org.apache.** {*;}
-keep class com.iflytek.** {*;}
-keep class com.google.protobuf.** { *; }
-keep class com.youyou.uuelectric.renter.pay.** {*;}

# ---------------- eventbus避免混淆 ------------
-keepclassmembers class ** {
    public void onEvent*(**);
    void onEvent*(**);
}


# --------------- 友盟统计避免混淆 -------------------------
-dontwarn android.support.v4.**
-dontwarn org.apache.commons.net.**
-dontwarn com.tencent.**
-keepclasseswithmembernames class * {
    native <methods>;
}
-keepclasseswithmembernames class * {
    public <init>(android.content.Context, android.util.AttributeSet);
}
-keepclasseswithmembernames class * {
    public <init>(android.content.Context, android.util.AttributeSet, int);
}
-keepclassmembers enum * {
    public static **[] values();
    public static ** valueOf(java.lang.String);
}
-keep class * implements android.os.Parcelable {
  public static final android.os.Parcelable$Creator *;
}
-keepclasseswithmembers class * {
    public <init>(android.content.Context);
}
-dontshrink
-dontoptimize
-dontwarn com.google.android.maps.**
-dontwarn android.webkit.WebView
-dontwarn com.umeng.**
-dontwarn com.tencent.weibo.sdk.**
-dontwarn com.facebook.**
-keep enum com.facebook.**
-keepattributes Exceptions,InnerClasses,Signature
-keepattributes *Annotation*
-keepattributes SourceFile,LineNumberTable
-keep public interface com.facebook.**
-keep public interface com.tencent.**
-keep public interface com.umeng.socialize.**
-keep public interface com.umeng.socialize.sensor.**
-keep public interface com.umeng.scrshot.**
-keep public class com.umeng.socialize.* {*;}
-keep public class javax.**
-keep public class android.webkit.**
-keep class com.facebook.**
-keep class com.umeng.scrshot.**
-keep public class com.tencent.** {*;}
-keep class com.umeng.socialize.sensor.**
-keep class com.tencent.mm.sdk.openapi.WXMediaMessage {*;}
-keep class com.tencent.mm.sdk.openapi.** implements com.tencent.mm.sdk.openapi.WXMediaMessage$IMediaObject {*;}
-keep class im.yixin.sdk.api.YXMessage {*;}
-keep class im.yixin.sdk.api.** implements im.yixin.sdk.api.YXMessage$YXMessageData{*;}
-keep public class [your_pkg].R$*{
    public static final int *;
}

# 热修复混淆
-keep class * extends java.lang.annotation.Annotation
-keep class com.alipay.euler.andfix.** { *; }
-keepclasseswithmembernames class * {
    native <methods>;
}
```

（5）混淆配置完成之后编译混淆包，测试

有的时候混淆之后可能会出现一些奇形怪状的bug，有条件的话，可以让QA回滚一次混淆包的测试。

- 资源文件混淆-微信方案
前面我们说过本文主要讲的是Apk的混淆，除了源代码的混淆，还有资源文件的混淆。

去年微信推出了一个apk资源混淆方案，该方案的具体原理课参见：<a href="http://mp.weixin.qq.com/s?__biz=MzAwNDY1ODY2OQ==&mid=208135658&idx=1&sn=ac9bd6b4927e9e82f9fa14e396183a8f#rd">安装包立减1M--微信Android资源混淆打包工具</a>

在其文章中分析了资源文件混淆的几种方案：

- 方案一：最简单的方法，我们按照Proguard的做法，直接在源码级别修改，将代码以及xml的R.string.name中替换到R.string.a，icon.png重命名为a.png 然后再交给Android编译。

- 方案二：根据Android的编译流程，所有资源ID已经被编译成32位int值。这说明我们并不需要去修改xml与java，因为在编译过程已经被R.java所替换，我们直接修改resources.arsc的二进制数据，不改变打包流程，只要在生成resources.arsc之后修改它，同时重命名资源文件。

- 方案三：直接处理安装包. 不依赖源码，不依赖编译过程，仅仅输入一个安装包，得到一个混淆包。

几种方案的对比如下：
![这里写图片描述](http://img.blog.csdn.net/20160219105712587)

微信版apk资源混淆方案采用的就是方案三，其具体的实现原理可参见其微信公众号中的介绍；

利用该方案我们可以实现对资源文件的混淆，将apk中所有的资源文件名称都替换为a、b、c、d...，这样也从侧面增加了不良人员反编译apk的难度，同时也减少了apk的大小，有兴趣的同学可以自己尝试一下。


**总结：**
上面我们分析了两种混淆方式代码混淆和资源文件混淆，其都是通过对源代码或者资源文件名称混淆（将其名称替换成无意义的名称）增加反编译的难度，减小Apk的大小，因此对产品而言这项工作还是很有意义的，一般而言都是做到了源代码混淆，而对资源文件混淆方面意识不足，希望大家通过阅读本文对Apk的混淆能有一个整体的认识。

另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51569261">android产品研发（五）-->多渠道打包</a>
