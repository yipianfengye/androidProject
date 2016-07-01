上一篇文章中我们讲解了android app中的升级更新操作，app的升级更新操作算是App的标配了，升级操作就是获取App的升级信息，更新操作是下载，安装，更新app，其中我们既可以使用app store获取应用的升级信息，也可以在应用内通过请求本地服务器获取应用的升级信息，并通过与本地app的版本号对比判断应用是否需要升级。
升级信息是app更新的基础，只有我们的app的升级信息指明需要更新，我们才可以开始后续的更新操作--也就是下载安装更新app。这里强调一点的是应用的升级操作分为普通的升级和强制升级两种，普通的升级操作就是完成一次对app的更新操作，而强制更新是在线上的app出现bug 的时候一种强制用户升级的手段，用户体验不太好，所以一般不太建议使用这种方式升级用户的app。
更多关于app升级更新的信息，可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>

本文将讲解android中数据传输中需要了解的数据序列化方面的知识，我们知道android开发过程中不同Activity之间传输数据可以通过Intent对象的put**方法传递，对于java的八大基本数据类型(char int float double long short boolean byte)传递是没有问题的，但是如果传递比较复杂的对象类型（比如对象，比如集合等），那么就可能存在问题，而这时候也就引入了数据序列化的概念。

**序列化的定义：**

这里我们先看一下呢序列化在百科上的定义
> 序列化 (Serialization)将对象的状态信息转换为可以存储或传输的形式的过程。在序列化期间，对象将其当前状态写入到临时或持久性存储区。以后，可以通过从存储区中读取或反序列化对象的状态，重新创建该对象。

简单来说就是我们的数据在传输的时候需要将信息转化为可以传输的数据，然后在传输的目标方能够反序列化将数据还原回来，这里的将对象状态信息转换为可传输数据的过程就是序列化，将可传输的数据逆还原为对象的过程就是反序列化。


**为什么需要序列化：**

知道前面的序列化定义，内存对象什么需要实现序列化呢？
 
 - 永久性保存对象，保存对象的字节序列到本地文件。
 
 - 通过序列化对象在网络中传递对象。
 
 - 通过序列化对象在进程间传递对象。


**实现序列化的两种方式：**

那么我们如何实现序列化的操作呢？在android开发中我们实现序列化有两种方式：

- 实现Serializable接口

- 实现parcelable接口


**两种序列化方式的区别：**

都知道在android studio中序列化有两种方式：serializable与parcelable。那么这两种实现序列化的方式有什么区别呢？下面是这两种实现序列化方式的区别：

1. Serializeble是java的序列化方式，Parcelable是android特有的序列化方式；

2. 在使用内存的时候，Parcelable比Serializable性能高，所以推荐使用Parcelable。

3. Serializable在序列化的时候会产生大量的临时变量，从而引起频繁的GC。

4. Parcelable不能使用在要将数据存储在磁盘上的情况，因为Parcelable不能很好的保证数据的持续性在外界有变化的情况下。尽管Serializable效率低点， 也不提倡用，但在这种情况下，还是建议你用Serializable。

最后还有一点就是Serializeble序列化的方式比较简单，直接集成一个接口就好了，而parcelable方式比较复杂，不仅需要集成Parcelable接口还需要重写里面的方法。


**对象实现序列化的实例：**

**通过实现Serializable接口实现序列化：**

上面介绍了那么多概念上的知识，下面我们就具体看一下如何通过这两种方式实现序列化，我们首先看一下如何通过实现Serializable接口实现序列化，通过实现Serializable接口实现序列化，只需要简单的实现Serialiizable接口即可。通过实现Serializable接口就相当于标记类型为序列化了，不需要做其他的操作了。

```
/**
 * Created by aaron on 16/6/29.
 */
public class Person implements Serializable{

    private int age;
    private String name;
    private String address;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }
}
```
可以发现我们定义了一个普通的实体Person类，并设置了三个成员属性以及各自的set，get方法，然后我们就只是简单的实现了Serializable接口就相当于将该类序列化了，当我们在程序中传输该类型的对象的时候就没有问题了。


**通过实现Parcelable接口实现序列化：**

然后我们在看一下通过实现Parcelable接口来实现序列化的方式，通过实现Parcelable接口实现序列化相当于实现Serialiable接口稍微复杂一些，因为其需要实现一些特定的方法，下面我们还是以我们定义的Person类为例子，看一下如果是实现Parcelable接口具体是如何实现的：

```
/**
 * Created by aaron on 16/6/29.
 */
public class Person implements Parcelable{

    private int age;
    private String name;
    private String address;

    public int getAge() {
        return age;
    }

    public void setAge(int age) {
        this.age = age;
    }

    public String getName() {
        return name;
    }

    public void setName(String name) {
        this.name = name;
    }

    public String getAddress() {
        return address;
    }

    public void setAddress(String address) {
        this.address = address;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int flags) {
        dest.writeInt(this.age);
        dest.writeString(this.name);
        dest.writeString(this.address);
    }

    public Person() {
    }

    protected Person(Parcel in) {
        this.age = in.readInt();
        this.name = in.readString();
        this.address = in.readString();
    }

    public static final Creator<Person> CREATOR = new Creator<Person>() {
        @Override
        public Person createFromParcel(Parcel source) {
            return new Person(source);
        }

        @Override
        public Person[] newArray(int size) {
            return new Person[size];
        }
    };
}
```
可以发现当我们通过实现Parcelable接口实现序列化还需要重写里面的成员方法，并且这些成员方法的写法都比较固定。


**实现Parcelable序列化的android studio插件：**

顺便说一下最近发现一个比较不错的Parcelable序列化插件。下面就来看一下如何安装使用该插件。

- 打开android studio --> settings --> Plugins --> 搜索Parcelable --> Install --> Restart，这样安装好了Parcelable插件；

![这里写图片描述](http://img.blog.csdn.net/20160630214306195)

- 然后在源文件中右键 --> Generate... --> Parcelable

![这里写图片描述](http://img.blog.csdn.net/20160630214346337)


- 点击Parcelable之后可以看到，源文件中已经实现了Parcelable接口，并重写了相应的方法：

![这里写图片描述](http://img.blog.csdn.net/20160630214425822)

这样我们就安装好Parcelable插件了，然后当我们执行Parcelable操作的时候就重写了Parcelable接口的相应序列化方法了。

**总结：**

- 可以通过实现Serializable和Parcelable接口的方式实现序列化

- 实现Serializable接口是java中实现序列化的方式，而实现Parcelable是android中特有的实现序列化的方式，更适合android环境

- 实现Serializable接口只需要实现该接口即可无需其他操作，而实现Parcelable接口需要重写相应的方法

- android studio中有实现Parcelable接口的相应插件，可安装该插件很方便的实现Parcelable接口，实现序列化

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
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
