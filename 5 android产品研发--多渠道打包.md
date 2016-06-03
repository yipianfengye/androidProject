国内的Android开发者还是很苦逼的，由于众所周知的原因，google play无法再国内打开，所以android系的应用市场，群雄争霸，而后果就是国内存在着有众多的应用市场，产品在不同的渠道可能有这不同的统计需求，为此android开发人员需要为每个应用市场发布一个安装包，这里就涉及到了android的多渠道打包。

本文主要讲解的就是几种主流的多渠道打包方式，以及其优劣势。

- 通过配置gradle脚本实现多渠道打包

这种打包方式是使用android Studio的编译工具gradle配合使用的，其核心原理就是通过脚本修改androidManifest.xml中的mate-date内容，执行N次打包签名操作实现多渠道打包的需求，具体实现如下。

（一）在androidmanifest.xml中定义mate-data标签
```
<manifest xmlns:android="http://schemas.android.com/apk/res/android"    
    package="your.package.name">    
    <application>    
    
          <meta-data android:name="UMENG_CHANNEL" android:value="{UMENG}"/>    
    
    </application>    
</manifest>   
```

 
这里需要注意的是：上面的value的值要和渠道名所对应，比如wandoujia里面要对应为你豌豆荚的渠道名称

（二）在build.gradle下的productFlavors定义渠道号：

```
productFlavors {  
  
        internal {}  
  
        /*InHouse {}  
        pcguanwang {}  
        h5guanwang {}  
        hiapk {}  
        m91 {}  
        appchina {}  
        baidu {}  
        qq {}  
        jifeng {}  
        anzhi {}  
        mumayi {}  
        m360 {}  
        youyi {}  
        wandoujia {}  
        xiaomi {}  
        sougou {}  
        leshangdian {}  
        huawei {}  
        uc {}  
        oppo {}  
        flyme {}  
        jinli {}  
        letv {}*/  
  
        productFlavors.all { flavor ->  
             flavor.manifestPlaceholders = [UMENG_CHANNEL_VALUE: name]  
        }  
    }  
```

同时需要注意的是，这里需要在defaultConfig中配置一个默认的渠道名称

```
manifestPlaceholders = [UMENG_CHANNEL_VALUE: "channel_name"]  
```

实现多渠道打包更换mate-data标签中的内容

优势：方便灵活，可以根据自身的需求配置不同的渠道执行不同的逻辑；
劣势：打包速度过慢；

- 使用第三方打包工具

这种方式就是使用第三方的服务，比如360，百度，友盟等，其原理也是通过修改androidManifest.xml中的mate-data标签内容，然后执行N次打包签名的操作实现多渠道打包的。这里就不在做具体解释说明，免得又做广告的嫌疑，O(∩_∩)O哈哈~。

优势：简单方便，几乎不用自身做什么工作；
劣势：打包速度过慢；

- 使用美团多渠道打包方式

而这里主要是根据美团客户端打包经验（详见：<a href="http://tech.meituan.com/mt-apk-packaging.html">美团Android自动化之旅—生成渠道包</a>）
主要是介绍利用在META-INF目录内添加空文件的方式，实现批量快速打包Android应用。

**实现原理**

Android应用安装包apk文件其实是一个压缩文件，可以将后缀修改为zip直接解压。解压安装文件后会发现在根目录有一个META-INF目录。如果在META-INF目录内添加空文件，可以不用重新签名应用。因此，通过为不同渠道的应用添加不同的空文件，可以唯一标识一个渠道。
“采用这种方式，每打一个渠道包只需复制一个apk，在META－INF中添加一个使用渠道号命名的空文件即可。这种打包方式速度非常快，900多个渠道不到一分钟就能打完。”

**实现步骤**

（一）编写渠道号文件

（二）编写python脚本，实现解压缩apk文件，为META-INF目录添加文件，重新压缩apk文件等逻辑：

```
# coding=utf-8
import zipfile
import shutil
import os

def delete_file_folder(src):  
    '''delete files and folders''' 
    if os.path.isfile(src):  
        try:  
            os.remove(src)  
        except:  
            pass 
    elif os.path.isdir(src):  
        for item in os.listdir(src):  
            itemsrc=os.path.join(src,item)  
            delete_file_folder(itemsrc)  
        try:  
            os.rmdir(src)  
        except:  
            pass

# 创建一个空文件，此文件作为apk包中的空文件
src_empty_file = 'info/empty.txt'
f = open(src_empty_file,'w')
f.close()

# 在渠道号配置文件中，获取指定的渠道号
channelFile = open('./info/channel.txt','r')
channels = channelFile.readlines()
channelFile.close()
print('-'*20,'all channels','-'*20)
print(channels)
print('-'*50)

# 获取当前目录下所有的apk文件
src_apks = [];
for file in os.listdir('.'):
    if os.path.isfile(file):
        extension = os.path.splitext(file)[1][1:]
        if extension in 'apk':
            src_apks.append(file)

# 遍历所以的apk文件，向其压缩文件中添加渠道号文件
for src_apk in src_apks:
	src_apk_file_name = os.path.basename(src_apk)
	print('current apk name:',src_apk_file_name)
	temp_list = os.path.splitext(src_apk_file_name)
	src_apk_name = temp_list[0]
	src_apk_extension = temp_list[1]

	apk_names = src_apk_name.split('-');

	output_dir = 'outputDir'+'/'
	if os.path.exists(output_dir):
		delete_file_folder(output_dir)
	if not os.path.exists(output_dir):
		os.mkdir(output_dir)

	# 遍历从文件中获得的所以渠道号，将其写入APK包中
	for line in channels:
		target_channel = line.strip()
		target_apk = output_dir + apk_names[0] + "-" + target_channel+"-"+apk_names[2] + src_apk_extension
		shutil.copy(src_apk,  target_apk)
		zipped = zipfile.ZipFile(target_apk, 'a', zipfile.ZIP_DEFLATED)
		empty_channel_file = "META-INF/uuchannel_{channel}".format(channel = target_channel)
		zipped.write(src_empty_file, empty_channel_file)
		zipped.close()

print('-'*50)
print('repackaging is over ,total package: ',len(channels))
input('\npackage over...')
```

（三）打包一个正常的apk包
（四）执行python脚本，多渠道打包
（五）android代码中获取渠道号

```
/**
 * 渠道号工具类：解析压缩包，从中获取渠道号
 */
public class ChannelUtil {
    private static final String CHANNEL_KEY = "uuchannel";
    private static final String DEFAULT_CHANNEL = "internal";
    private static String mChannel;

    public static String getChannel(Context context) {
        return getChannel(context, DEFAULT_CHANNEL);
    }

    public static String getChannel(Context context, String defaultChannel) {
        if (!TextUtils.isEmpty(mChannel)) {
            return mChannel;
        }
        //从apk中获取
        mChannel = getChannelFromApk(context, CHANNEL_KEY);
        if (!TextUtils.isEmpty(mChannel)) {
            return mChannel;
        }
        //全部获取失败
        return defaultChannel;
    }

```

```
    /**
     * 从apk中获取版本信息
     *
     * @param context
     * @param channelKey
     * @return
     */
    private static String getChannelFromApk(Context context, String channelKey) {
        long startTime = System.currentTimeMillis();
        //从apk包中获取
        ApplicationInfo appinfo = context.getApplicationInfo();
        String sourceDir = appinfo.sourceDir;
        //默认放在meta-inf/里， 所以需要再拼接一下
        String key = "META-INF/" + channelKey;
        String ret = "";
        ZipFile zipfile = null;
        try {
            zipfile = new ZipFile(sourceDir);
            Enumeration<?> entries = zipfile.entries();
            while (entries.hasMoreElements()) {
                ZipEntry entry = ((ZipEntry) entries.nextElement());
                String entryName = entry.getName();
                if (entryName.startsWith(key)) {
                    ret = entryName;
                    break;
                }
            }
        } catch (IOException e) {
            e.printStackTrace();
        } finally {
            if (zipfile != null) {
                try {
                    zipfile.close();
                } catch (IOException e) {
                    e.printStackTrace();
                }
            }
        }
        String channel = "";
        if (!TextUtils.isEmpty(ret)) {
            String[] split = ret.split("_");
            if (split != null && split.length >= 2) {
                channel = ret.substring(split[0].length() + 1);
            }
            System.out.println("-----------------------------");
            System.out.println("渠道号：" + channel + "，解压获取渠道号耗时:" + (System.currentTimeMillis() - startTime) + "ms");
            System.out.println("-----------------------------");
        } else {
            System.out.println("未解析到相应的渠道号，使用默认内部渠道号");
            channel = DEFAULT_CHANNEL;
        }
        return channel;
    }
}
```

整个打包的流程就是这样了，打工工具可参考github上的项目：<a href="https://github.com/yipianfengye/buildapk">多渠道打包实现</a>

优势：打包速度很快，很方便；
劣势：不够灵活，不能灵活的配置不同的渠道不同的业务逻辑；

问题：
项目中由于使用了友盟统计，以前是在meta-data中保存渠道信息，现在更改了方式之后需要手动执行渠道号的设置代码：

```
String channel = ChannelUtil.getChannel(mContext);
        System.out.println("启动页获取到的渠道号为：" + channel);
        // 设置友盟统计的渠道号，原来是在Manifest文件中设置的meta-data，现在启动页中设置
        AnalyticsConfig.setChannel(channel);
```

通过这种打包方式以前需要一个小时的打包工作现在只需要一分钟即可，极大的提高了效率，目前在实际的应用中尚未发现有什么问题，有这种需求的童鞋可以尝试一下。

**总结**

虽说我们总结了三种打包方式，但是其实通过gradle打包和使用第三方服务打包都是执行了N次的打包签名操作，时间上耗费太多，因此不太推荐，而美团的方式在效率上提高了很多，但是对于那种不同的渠道包执行不同的业务逻辑的需求就无能为例了，只能通过gradle配置，因此大家在选择多渠道打包方式的时候可以根据自身的需求来选择。



另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51534013">android产品研发（一）-->实用开发规范</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51541277">android产品研发（二）-->启动页优化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51546974">android产品研发（三）-->基类Activity</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51559066">android产品研发（四）-->减小Apk大小</a>
