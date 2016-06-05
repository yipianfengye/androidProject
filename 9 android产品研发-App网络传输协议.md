上一篇文章中我们讲解了App的数据统计，其主要分为两种：使用第三方服务统计和自身实现数据统计。一般而言我们使用第三方统计服务已经可以很好的满足我们的也无需求了，只是部分数据敏感性App，可能自身实现数据统计服务是一个更好的选择。

而本文中将要介绍的是App端的网络传输协议。那么这里首先需要明确一点的是什么是网络传输协议呢？好吧，这里首先套用一段百度百科的定义：

> 网络传输协议或简称为传送协议（Communications Protocol[1]  ），是指计算机通信的共同语言。现在最普及的计算机通信为网络通信，所以“传送协议”一般都指计算机通信的传送协议，如TCP/IP、NetBEUI等。然而，传送协议也存在于计算机的其他形式通信，例如，面向对象编程里面对象之间的通信；操作系统内不同程序之间的消息，都需要有一个传送协议，以确保传信双方能够沟通无间。

简单而言就是App端与服务器端交互的时候约定好的内容格式。比如我们常见的Json格式，xml格式等都是网络传输协议，而现在在App开发中比较常见的网络传输协议有三种：<a href="http://baike.baidu.com/link?url=HBlJ8mkEgnw9DfDiOl9oq9XhILLR9D6JFGO-xTec-4-QjOBtItP5lBGV1Eb4pHy2CyHeBf8onxir1tmqHxMoGeQMovJfGq0Dz32iV8Obn5lEbDwxMwkU5FLBOAsZdzjP">xml</a>，<a href="http://baike.baidu.com/link?url=A5HS6n-Hqj6yvSqHP1yW_WqTWzniuDtEVFjdtFSQ6F2h1cbSsIiK21iKf7WVW1rSJ0EzGfwkVAT-CWB-yBYkt_">json</a>，<a href="http://baike.baidu.com/link?url=wuynuiEmT9miQNtr3aD8atJUBywMOe2YFLzxwXd2rbRL4hDCmzuTkRU37TssBCdHR4gg8L98gIbpd_JAsDnda_">protobuf</a>而我们下面将分别分析其各自的使用利弊以及解析方案。

**- 网络传输协议-XML**

xml是一种最早的网络传输协议，常见于java web开发中，不单单作为网络层的参数协议，还常见于各种配置文件中，在移动开发中也常见但是已不是主流的网络传输协议。

优点：可读性强，解析方便；
缺点：效率不高，资源消耗过大；
解析方式：DOM解析，SAX解析，PULL解析；

（1）DOM解析：
解析器读入整个文档，然后构建一个驻留内存的树结构，然后代码就可以使用 DOM 接口来操作这个树结构。优点：整个文档树在内存中，便于操作；支持删除、修改、重新排列等多种功能；缺点：将整个文档调入内存（包括无用的节点），浪费时间和空间；使用场合：一旦解析了文档还需多次访问这些数据；硬件资源充足（内存、CPU）
（2）SAX解析：
SAX ，事件驱动型解析方式。当解析器发现元素开始、元素结束、文本、文档的开始或结束等时，发送事件，程序员编写响应这些事件的代码，保存数据。优点：不用事先调入整个文档，占用资源少；SAX解析器代码比DOM解析器代码小，适于Applet，下载。缺点：不是持久的；事件过后，若没保存数据，那么数据就丢了；无状态性；从事件中只能得到文本，但不知该文本属于哪个元素；使用场合：Applet;只需XML文档的少量内容，很少回头访问；机器内存少；
（3）PULL解析：
PULL解析方式是android专门为移动设备上解析XML文件而设计的一种解析方式，显而易见的其更加适用于移动设备解析xml文件。Pull解析和Sax解析很相似,Pull解析和Sax解析不一样的地方是pull读取xml文件后触发相应的事件调用方法返回的是数字还有pull可以在程序中控制想解析到哪里就可以停止解析。


**- 网络传输协议-JSON**

JSON是在移动端比较常见的网络传输协议，它较xml格式更叫的简单和“小”，因此比xml更适合移动端对流量和内存的控制。

优点：较XML格式更加小巧；
缺点：传输效率也不是太别高，但相较于xml提高了很多；
解析方式：Gson解析，JSONObject方式解析，FastJson解析

（1）Gson解析：
Gson解析方式是Google开源的一套解析方式，通过提供的Gson jar包，通过静态方法直接由字符串解析成java对象，简单方便。
具体使用方法，可参考：<a href="http://www.cnblogs.com/haippy/archive/2012/05/20/2509329.html">Google Gson 使用简介</a>

（2）JSONObject解析：
JSONObject在org.json下面的包中，其也是一个解析Json字符串的工具类，具体使用方式可参考：<a href="http://www.cnblogs.com/xwdreamer/archive/2011/12/16/2296904.html">JSONObject与JSONArray的使用</a>

（3）FastJson解析：
FastJson是阿里巴巴开源的一个解析Json数据的类库，能够将json字符串解析成java对象。这里有其介绍，英文水平不好就不献丑了...

- Provide best performance in server side and android client.

- Provide simple toJSONString() and parseObject() methods to convert Java objects to JSON and vice-versa

- Allow pre-existing unmodifiable objects to be converted to and from JSON

- Extensive support of Java Generics

- Allow custom representations for objects

- Support arbitrarily complex objects (with deep inheritance hierarchies and extensive use of generic types)


**- 网络传输协议-ProtoBuf**

ProtoBuf是Google开源的一套二进制流网络传输协议，它独立于语言，独立于平台。google 提供了多种语言的实现：java、c#、c++、go 和 python，每一种实现都包含了相应语言的编译器以及库文件。由于它是一种二进制的格式，比使用 xml 进行数据交换快许多。可以把它用于分布式应用之间的数据通信或者异构环境下的数据交换。作为一种效率和兼容性都很优秀的二进制数据传输格式，可以用于诸如网络传输、配置文件、数据存储等诸多领域。

优点：传输效率快（比xml和json快10-20倍），文档型协议；
缺点：使用不太方便；

这里简单解释一下什么是文档型协议，向我们的xml和json一般在使用的时候都需要保存一份说明文档和一个实际的java类，而protobuf在使用的时候其定义的格式就是说明文档，简单明了而且可以将其编译成各个平台的类库，以java平台为例，其编程成jar之后，若定义文件发生了变化，则在使用jar包的话就会报错，必须重新编译，这也就保证了App端与服务器端的协议统一性。


**网络传输协议实践**

由于ProtoBuf的传输效率和文档型协议的特性，公司产品选择了Protobuf作为网络传输协议。下面我就以一个简单的登录操作，介绍一下对ProtoBuf的实际应用。

（1）定义ProtoBuf文件
![这里写图片描述](http://img.blog.csdn.net/20160605124258648)

（2）定义登录网络操作请求接口

```
// 提交短信验证码，如果该手机号码没有注册过，则直接给用户注册
message SmsLogin {
	message Request {
		required string phone = 1; // 手机号码
		required string verifyCode = 2; // 验证码
	}

	message Response {
		/*
		 * 0：成功；-1:登录失败；-2：验证码错误；-3：验证码已经失效；-4：手机号码格式不正确；-5：不是内测用户不能登录
		 */
		required int32 ret = 1;
		optional com.uu.facade.passport.pb.bean.UserAppSessionTicket sessionTicket = 2;
		optional string phone = 3; // 手机号码
		optional string imgUrl = 4; // 用户图像url
		optional com.uu.facade.base.common.UserStatus userStatus = 5; // 用户状态
	}

}
```
可以看到在Protobuf中定义网络请求，分为两个部分，请求部分和应答部分，其message request定义的是请求信息，而message response定义的是应答信息。请求或者应答信息中的字段就是我们请求中需要传递的字段。

每个字段都有修饰符，这里的required表示的是这个字段必须要传递而optional表示的是这个字段可传可不传。并且message中可以嵌套另外的message信息，比如应答信息中的UserStatus。

（3）将proto文件编译成jar包
这里就不在具体介绍怎么讲proto文件编译成jar了，google已经提供了相应的编译工具。

（4）在android代码中使用
由于我们将proto文件编译成了jar包，首先我们需要将jar引入到我们的工程，然后就可以使用了。这里简单看一下具体的使用代码。

```
/**
     * 请求服务器短信登陆
     *
     * @param phone
     * @param code
     */
    private void requestSmsLogin(String phone, String code) {
        showProgress(false);// 显示进度条
        LoginInterface.SmsLogin.Request.Builder request = LoginInterface.SmsLogin.Request.newBuilder();
        request.setPhone(phone); // 设置手机号
        request.setVerifyCode(code); // 设置验证码
        NetworkTask task = new NetworkTask(Cmd.CmdCode.SmsLogin_SSL_VALUE);
        task.setBusiData(request.build().toByteArray());
        NetworkUtils.executeNetwork(task, new HttpResponse.NetWorkResponse<UUResponseData>() {
            @Override
            public void onSuccessResponse(UUResponseData responseData) {
                if (responseData.getRet() == 0) {
                    try {
	                    // 显示通用下发消息
                        showResponseCommonMsg(responseData.getResponseCommonMsg());
                        // 解析请求应答信息
                        LoginInterface.SmsLogin.Response response = LoginInterface.SmsLogin.Response.parseFrom(responseData.getBusiData());
                        // 判断是否请求成功
                        if (response.getRet() == 0) {
                            ...doSomeThing()...
                        }
                    } catch (Exception e) {
                        e.printStackTrace();
                        showDefaultNetworkSnackBar();
                    }
                }
            }

            @Override
            public void onError(VolleyError errorResponse) {
                showDefaultNetworkSnackBar();
            }

            @Override
            public void networkFinish() {
		        // 取消进度条显示
                dismissProgress();
            }
        });
    }
```
可以发现我们在代码中直接有对应的登录请求message类，这样我们就可以直接通过java类调用了，O(∩_∩)O哈哈~。

**总结：**

xml，json，protobuf三种不同的网络传输协议各有利弊，大家在选择的时候可以根据具体的产品需求来确定到底选择哪一个，当然了这里我还是比较推荐protobuf的。
