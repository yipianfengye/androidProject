上一篇文章中我们讲解了android中的实用调试技巧。讲解了android中的原生Log API以及其使用方式，讲解了自定义日志API、使用方式和实现原理，讲解了通过gradle配置日志框架在正式环境中屏蔽日志信息等。最后我们还重点讲解了android studio中的断点调试技巧，主要包括：断点调试功能、日志断点、求值调试、异常断点、方法断点等。更多关于android中实用调试技巧的知识，可以参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51868496">android产品研发（二十二）-->android实用调试技巧</a>

本文我们将讲解一个android产品研发中可能会碰到的一个问题：如何在App中保存静态秘钥以及保证其安全性。许多的移动app需要在app端保存一些静态字符串常量，其可能是静态秘钥、第三方appId等。在保存这些字符串常量的时候就涉及到了如何保证秘钥的安全性问题。如何保证在App中静态秘钥唯一且正确安全，这是一个很重要的问题，公司的产品中就存在着静态字符串常量类型的秘钥，所以一个明显的问题就是如何生成秘钥，保证秘钥的安全性？

**现今保存静态秘钥的几种主流通用做法：**（参考：<a href="http://www.cnblogs.com/alisecurity/archive/2016/05/16/5498539.html">Android安全开发之浅谈密钥硬编码</a>）

- 通过SharedPreferences保存静态秘钥；

- 通过java硬编码的方式保存

- 通过NDK的方式，将静态秘钥保存在so文件中；

**几种保存静态秘钥方式的优劣势：**

> - 密钥直接明文存在sharedprefs文件中，这是最不安全的。

> - 密钥直接硬编码在Java代码中，这很不安全，dex文件很容易被逆向成java代码。

> - 将密钥分成不同的几段，有的存储在文件中、有的存储在代码中，最后将他们拼接起来，可以将整个操作写的很复杂，这因为还是在java层，逆向者只要花点时间，也很容易被逆向。

> - 用ndk开发，将密钥放在so文件，加密解密操作都在so文件里，这从一定程度上提高了的安全性，挡住了一些逆向者，但是有经验的逆向者还是会使用IDA破解的。

> - 在so文件中不存储密钥，so文件中对密钥进行加解密操作，将密钥加密后的密钥命名为其他普通文件，存放在assets目录下或者其他目录下，接着在so文件里面添加无关代码（花指令），虽然可以增加静态分析难度，但是可以使用动态调式的方法，追踪加密解密函数，也可以查找到密钥内容。

可以说在设备上安全存储密钥这个基本无解，只能选择增大逆向成本。而要是普通开发者的话，这需要耗费很大的心血，要评估你的app应用的重要程度来选择相应的技术方案。

**产品App中需要保存的秘钥：**

由于app需要与服务器交互所以这时候若用户为登录则客户端需要一个默认的秘钥来确认App的游客身份，因此需要在app中保存一个默认的请求秘钥。

- 当用户未登录或者是退出登录时，请求服务器使用默认的秘钥字符串；

- 当用户登录时使用从服务器或其的秘钥字符串；

这样我们需要在App端保存一个默认的秘钥字符串用于标识用户的游客身份，而一个问题就是如何保证秘钥字符串的安全性？

**考虑到的其他几种保存秘钥方式：**

- 通过保存文件的方式保存秘钥信息；

- 通过数据库的方式保存秘钥信息；

- 通过配置gradle的方式保存秘钥信息；

- 通过配置string.xml的方式保存秘钥信息；

**文件方式：**显而易见的通过保存文件的方式保存秘钥信息，有一个问题就是，如果用户恶意删除App保存的秘钥信息，那么App就无法使用默认的秘钥信息了，这样秘钥的唯一性也就无法保证。

**数据库方式：**和通过文件的方式保存秘钥信息一样，通过数据库保存也是无法保证恶意用户删除数据库信息，这样我们的App就丢失了默认的秘钥信息。

**gradle配置：**我们通过下面的gradle配置变量的方式，来讲解如何在gradle配置变量信息。

**string.xml：**在下面我们做详细的介绍。

**通过gradle配置变量：**

在android gradle中我们不单可以引用依赖包，执行脚本还可以配置静态变量：

```
buildTypes {
        debug {
            // 显示Log
            buildConfigField "boolean", "LOG_DEBUG", "true"
            buildConfigField "String", "appKeyPre", "\"xxx\""
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
            buildConfigField "String", "appKeyPre", "\"xxx\""
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
如上面代码所示我们在gradle中配置了一个名称为appKey的字符串变量，编译gradle则会在gradle的编译类：BuildConfig中生成该静态变量：

```
/**
 * gradle编译后生成的编译类
 */
public final class BuildConfig {
  public static final boolean DEBUG = Boolean.parseBoolean("true");
  public static final String APPLICATION_ID = "com.sample.renter";
  public static final String BUILD_TYPE = "debug";
  public static final String FLAVOR = "internal";
  public static final int VERSION_CODE = 1;
  public static final String VERSION_NAME = "1.0.0";
  // Fields from build type: debug
  public static final boolean LOG_DEBUG = true;
  public static final String appKey = "xxx";
}
```
可以发现通过配置gradle的方式配置静态秘钥反编译的时候也是找到秘钥的，但是增加了逆向的难度，而且避免了被用户的恶意删除，所以通过gradle配置字符串的方式保存app中的静态秘钥是一个不错的选择。


**通过string.xml配置秘钥信息**

通过string.xml配置秘钥信息也是一个不错的选择。在string.xml中定义字符串值之后，通过代码获取反编译apk效果如下：

![image](http://img.blog.csdn.net/20160722115648713)

可以发现这里无法显式的展示出string字符串值，但是这里出现了string字符串的ID值，但是需要说明的是恶意攻击是可以通过string的ID之在R.java文件中查找到相应的string名称，进而在string.xml中找到字符串值。但是这样也是增加了反编译的难度，相对来说也是一个比较不错的选择。


**产品中保存静态秘钥实践：**

最终经过比对各种静态秘钥存储方案，我们决定使用gradle配置 + 静态代码 + 字符串运算 + string.xml值的方式实现静态秘钥的存储。

首先将静态秘钥分为四部分：

- 第一部分通过gradle配置的方式存储；

- 第二部分通过java硬编码的方式存储；

- 第三部分通过java字符串拼接运算的方式存储；

- 第四部分通过string.xml保存；


**获取appKey第一部分字符串：**
通过gradle配置的方式我们上面已经做了介绍，其就是在mudle中的gradle文件中再起buildType节点下定义字符串变量，这里需要注意的是若是有正式环境和测试环境之分，需要分别定义字符串变量，这样我们就可以通过BuildConfig获取appKay第一部分的字符串了。

```
/**
 * 获取AppKey part1
 */
public static String getBK1() {
        return BuildConfig.appKey;
    }
```

**获取appKey第二部分字符串：**

第二部分的appKay字符串是通过运算的出来的，这里的运算可以是任意运算方式，越复杂越好，越让人看不懂越好，当然结果需要时唯一的。比如：
```
public static StringBuffer getBk2() {
        StringBuffer sb = new StringBuffer();
        sb.append(Config.getGBS(2, 5));
        return sb;
    }
```
而这里的getGBS方法的实现：

```
public static int getGBS(int x, int y){
        for(int i = 1; i<= x * y; i++){
            if(i % x == 0 && i % y == 0)
                return i;
        }

        return x * y;
    }
```
最终的结果返回是：10，当然了不同的字符串需要不同的算法；

**获取appKey第三部分字符串：**

这里就只是使用了简单的字符串硬编码

```
public static String getBK3() {
	return "xhxh";
}
```

**获取appKey第四部分字符串**

- 在string.xml中定义appKey的第四部分

```
<string name="bk4">chs</string>
```

- 在代码中获取string中定义的字符串值

```
public static String getBk4() {
	mContext().getResources().getString(R.string.bk4);
}
```


获取最终的appKey字符串：

```
/**
 * 获取最终的appKey字符串
 */
public static byte[] getDefaultKey() {
		StringBuffer sb = new StringBuffer();
	    sb.append(getBK1()).append(getBK2()).append(getBk3()).append(getBk4());
            
        return sb.toString();
    }
```
这样经过一系列的操作之后我们就获取到了最终的静态秘钥。当然了产品中最好实现了代码混淆的功能，这样也能增大逆向的难度。

**总结：**

- 在App端保存静态秘钥可以通过SharedPreferences、java硬编码，ndk中的so文件，文件，数据库，gradle配置的方式实现；

- 为了保证秘钥的安全性可以采用多种方式混合，这样可以增加恶意反编译的难度；

- 在App端保存秘钥不能真正的保证秘钥的安全性，只能增加反编译的难易程度；

- 可以使用gradle配置的方式配置静态秘钥，使用string.xml配置秘钥增加逆向反编译的难度；

- 不推荐使用文件，数据库的方式保存静态秘钥，容易被用户恶意删除，而出现不可预知的错误；

<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">android产品研发（十九）-->android studio中的单元测试</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080">android产品研发（二十）-->代码Review</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868453">android产品研发（二十一）-->android中的UI优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868496">android产品研发（二十二）-->android实用调试技巧</a>
