上一篇文章中我们讲解了android产品研发过程中的代码Review。通过代码Review能够提高产品质量，增强团队成员之间的沟通，提高开发效率，所以良好的产品开发迭代过程中，代码Review是一个必不可少的步骤。那么如何进行代码Review呢？我们主要讲解了团队成员之间的代码Review，代码lint检查，开发规范等方面的知识点，更多关于代码Review相关的知识可参考我的：<a href="http://blog.csdn.net/qq_23547831/article/details/51833080">android产品研发（二十）-->代码Review</a>

本文我们将讲解一下android UI优化方面的知识。android系统的优化分为好多方面：比如性能优化，UI优化，资源文件优化等等，这里我们先暂时讲解android UI优化方面的知识点。

**三种布局方式**
android对布局优化提供了三种布局：

```
<include/>
<merge/>
<ViewStub/>
```
这三种布局都可以简化我们的布局文件，优化绘制流程，下面我们简单看一下这三种组件的使用方式。

1、重用布局`<include/>`

```
<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"  
    android:orientation="vertical"   
    android:layout_width=”match_parent”  
    android:layout_height=”match_parent”  
    android:background="@color/app_bg"  
    android:gravity="center_horizontal">  
  
    <include layout="@layout/titlebar"/>  
  
    <TextView android:layout_width=”match_parent”  
              android:layout_height="wrap_content"  
              android:text="@string/hello"  
              android:padding="10dp" />  
  
    ...  
  
</LinearLayout>  
```

1)<include />标签可以使用单独的layout属性，这个也是必须使用的。
2)可以使用其他属性。<include />标签若指定了ID属性，而你的layout也定义了ID，则你的layout的ID会被覆盖。
3)在include标签中所有的android:layout_*都是有效的，前提是必须要写layout_width和layout_height两个属性。


2、减少视图层级`<merge/>`
    这个标签在UI的结构优化中起着非常重要的作用，它可以删减多余的层级，优化UI。`<merge/>`多用于替换FrameLayout或者当一个布局包含另一个时，标签消除视图层次结构中多余的视图组。例如你的主布局文件是垂直布局，引入了一个垂直布局的include，这是如果include布局使用的LinearLayout就没意义了，使用的话反而减慢你的UI表现。这时可以使用<merge/>标签优化。

```
<merge xmlns:android="http://schemas.android.com/apk/res/android">  
  
    <Button  
        android:layout_width="fill_parent"   
        android:layout_height="wrap_content"  
        android:text="@string/add"/>  
  
    <Button  
        android:layout_width="fill_parent"   
        android:layout_height="wrap_content"  
        android:text="@string/delete"/>  
  
</merge>  
```

现在，当你添加该布局文件时(使用<include />标签)，系统忽略<merge />节点并且直接添加两个Button。

3、需要时使用`<ViewStub/>`
    这个标签最大的优点是当你需要时才会加载，使用他并不会影响UI初始化时的性能。各种不常用的布局想进度条、显示错误消息等可以使用这个标签，以减少内存使用量，加快渲染速度。

```
<ViewStub  
    android:id="@+id/stub_import"  
    android:inflatedId="@+id/panel_import"  
    android:layout="@layout/progress_overlay"  
    android:layout_width="fill_parent"  
    android:layout_height="wrap_content"  
    android:layout_gravity="bottom" />  
```

当你想加载布局时，可以使用下面其中一种方法：

```
((ViewStub) findViewById(R.id.stub_import)).setVisibility(View.VISIBLE);  
// or  
View importPanel = ((ViewStub) findViewById(R.id.stub_import)).inflate();  

```


**android中的过度绘制**

android开发者选项中有一项是：“调试GPU过度绘制”，过度绘制描述的是屏幕上一个像素在单个帧中被重绘了多少次。比如一个有背景的TextView，那么显示文本的那些像素至少绘制了两次，一次是背景，一次是文本。过度绘制是Android平台上一个很棘手的性能问题，它非常容易出现。

**过度绘制产生的原因**

- 太多重叠的背景
重叠着的背景有时候是有必要的，有时候是没必要的。这要视你的项目具体情况而定.

- 太多叠加的View
或者本来这个UI布局就很复杂或者你是为了追求一个炫丽的视觉效果，这都有可能使得很多view叠加在一起。这个情况非常普遍，下面的建议中会谈谈怎么减少这种情况带来的影响。

- 复杂的Layout层级
复杂的层级关系，这个在布局中也很常见，下面也会说这种情况怎么做可以尽可能的减少过度绘制。

**建议**

- 太多重叠的背景
这个问题其实最容易解决，建议就是检查你在布局和代码中设置的背景，有些背景是被隐藏在底下的，它永远不可能显示出来，这种没必要的背景一定要移除，因为它很可能会严重影响到app的性能。如果采用的是selector的背景，将normal状态的color设置为”@android:color/transparent”,也同样可以解决问题。

- 太多重叠的view
第一个建议是：使用ViewStub来加载一些不常用的布局，它是一个轻量级且默认不可见的视图，可以动态的加载一个布局，只有你用到这个重叠着的view的时候才加载，推迟加载的时间。第二个建议是：如果使用了类似viewpager+Fragment这样的组合或者有多个Fragment在一个界面上，需要控制Fragment的显示和隐藏，尽量使用动态地Inflation view，它的性能要比SetVisiblity好。

- 复杂的Layout层级
这里的建议比较多一些，首先推荐用Android提供的布局工具Hierarchy Viewer来检查和优化布局。第一个建议是：如果嵌套的线性布局加深了布局层次，可以使用相对布局来取代。第二个建议是：用标签来合并布局，这可以减少布局层次。第三个建议是：用标签来重用布局，抽取通用的布局可以让布局的逻辑更清晰明了。记住，这些建议的最终目的都是使得你的Layout在Hierarchy Viewer里变得宽而浅，而不是窄而深。

**实例分析**

（一）启用调试GPU过度绘制功能；
设置-》其他高级设置-》开发者选项-》调试GPU过度绘制；
![这里写图片描述](http://img.blog.csdn.net/20160115101113411)

我们可以看到我们的手机界面中显现除了一下背景颜色，这里介绍一下这些背景颜色表示的就是布局过度绘制的情况，其中：
蓝色：一次绘制
绿色：两次绘制
淡红：三次绘制
深红：四次绘制
所以当我们App的界面中深红或者是淡红的部分较多时我们可以分析一下我们的界面布局是否存在过度绘制的情况。

（二）打开我们的测试页面，这里以一个意见反馈为例：

![image](http://img.blog.csdn.net/20160115123159747)

这里可以看出整个页面你的布局为蓝色，即渲染了一次；
输入框中的北京为绿色，渲染了两次；
文字部分为淡红色，渲染了三次；

整体来看整个布局渲染基本正常，不存在过度绘制的问题；
下面这是布局文件的xml定义：

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/c10">

    <RelativeLayout
        android:layout_height="match_parent"
        android:layout_width="match_parent">

        <EditText
            android:id="@+id/feedback_content_edit"
            android:layout_width="match_parent"
            android:layout_height="126dp"
            android:layout_marginTop="18dp"
            android:paddingLeft="@dimen/s3"
            android:paddingRight="@dimen/s3"
            android:paddingTop="@dimen/s2"
            android:paddingBottom="@dimen/s2"
            android:background="#FFFFFF"
            android:textColorHint="@color/c4"
            android:textSize="@dimen/f4"
            android:textColor="@color/c3"
            android:hint="尽量详细描述您遇到的问题，感谢您给我们提出建议"
            android:gravity="top|left" />

        <EditText
            android:id="@+id/feedback_contact_edit"
            android:layout_below="@id/feedback_content_edit"
            android:layout_width="match_parent"
            android:layout_height="45dp"
            android:layout_marginTop="@dimen/s3"
            android:paddingLeft="@dimen/s3"
            android:paddingRight="@dimen/s3"
            android:background="#FFFFFF"
            android:textColorHint="@color/c4"
            android:textSize="@dimen/f4"
            android:textColor="@color/c3"
            android:hint="请输入您的手机号码、QQ或者邮箱"
            android:singleLine="true" />

        <TextView
            android:id="@+id/feedback_info_text"
            android:layout_below="@id/feedback_contact_edit"
            android:layout_width="match_parent"
            android:layout_height="wrap_content"
            android:layout_marginTop="@dimen/s3"
            android:paddingLeft="@dimen/s3"
            android:paddingRight="@dimen/s3"
            android:background="#00000000"
            android:textSize="@dimen/f5"
            android:textColor="@color/c5"
            android:text="留下联系方式，以便能更好更快的给您答复，还可能有意外奖励哦" />

        <include
            layout="@layout/b3_button"
            android:layout_width="match_parent"
            android:layout_height="48dp"
            android:layout_below="@id/feedback_info_text"
            android:layout_marginLeft="@dimen/s4"
            android:layout_marginRight="@dimen/s4"
            android:layout_marginTop="12dp"
            android:gravity="center"
            android:text="提交"
            android:textSize="@dimen/f2"
            android:textColor="@color/c11" />

    </RelativeLayout>

</RelativeLayout>

```

下面我们为整个布局文件多添加一层布局控件

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/c10">
    
    <!-- 这里是新添加的布局控件-->
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent">

        <RelativeLayout
            android:layout_height="match_parent"
            android:layout_width="match_parent">
    
            <EditText
                android:id="@+id/feedback_content_edit"
                android:layout_width="match_parent"
                android:layout_height="126dp"
                android:layout_marginTop="18dp"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:paddingTop="@dimen/s2"
                android:paddingBottom="@dimen/s2"
                android:background="#FFFFFF"
                android:textColorHint="@color/c4"
                android:textSize="@dimen/f4"
                android:textColor="@color/c3"
                android:hint="尽量详细描述您遇到的问题，感谢您给我们提出建议"
                android:gravity="top|left" />
    
            <EditText
                android:id="@+id/feedback_contact_edit"
                android:layout_below="@id/feedback_content_edit"
                android:layout_width="match_parent"
                android:layout_height="45dp"
                android:layout_marginTop="@dimen/s3"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:background="#FFFFFF"
                android:textColorHint="@color/c4"
                android:textSize="@dimen/f4"
                android:textColor="@color/c3"
                android:hint="请输入您的手机号码、QQ或者邮箱"
                android:singleLine="true" />
    
            <TextView
                android:id="@+id/feedback_info_text"
                android:layout_below="@id/feedback_contact_edit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/s3"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:background="#00000000"
                android:textSize="@dimen/f5"
                android:textColor="@color/c5"
                android:text="留下联系方式，以便能更好更快的给您答复，还可能有意外奖励哦" />
    
            <!--<TextView
                android:id="@+id/feedback_ok_text"
                android:layout_width="match_parent"
                android:layout_height="45dp"
                android:layout_below="@id/feedback_info_text"
                android:layout_marginLeft="@dimen/s1"
                android:layout_marginRight="@dimen/s1"
                android:layout_marginTop="18dp"
                android:gravity="center"
                android:text="提交"
                android:textSize="@dimen/f2"
                android:textColor="@color/c11"
                android:background="@drawable/red_button_bg" />-->
    
            <include
                layout="@layout/b3_button"
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:layout_below="@id/feedback_info_text"
                android:layout_marginLeft="@dimen/s4"
                android:layout_marginRight="@dimen/s4"
                android:layout_marginTop="12dp"
                android:gravity="center"
                android:text="提交"
                android:textSize="@dimen/f2"
                android:textColor="@color/c11" />
    
        </RelativeLayout>
    </RelativeLayout>

</RelativeLayout>

```
再次执行查看绘制情况：
![image](http://img.blog.csdn.net/20160115124248519)

咦？没什么变化么？这是怎么回事？其实对于android系统来说，在布局文件中新添加一个布局控件只是会让它存在布局的逻辑关系而不会为这个布局控件重新绘制一次，只有这个布局控件设置background的时候才会重新绘制一次，下面我们为这个RelativeLayout设置以下background

```
<RelativeLayout xmlns:android="http://schemas.android.com/apk/res/android"
    xmlns:tools="http://schemas.android.com/tools"
    android:layout_width="match_parent"
    android:layout_height="match_parent"
    android:background="@color/c10">

    <!-- 这里是新添加的布局控件-->
    <RelativeLayout
        android:layout_width="match_parent"
        android:layout_height="match_parent"
        android:background="@color/c10"
        >

        <RelativeLayout
            android:layout_height="match_parent"
            android:layout_width="match_parent">

            <EditText
                android:id="@+id/feedback_content_edit"
                android:layout_width="match_parent"
                android:layout_height="126dp"
                android:layout_marginTop="18dp"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:paddingTop="@dimen/s2"
                android:paddingBottom="@dimen/s2"
                android:background="#FFFFFF"
                android:textColorHint="@color/c4"
                android:textSize="@dimen/f4"
                android:textColor="@color/c3"
                android:hint="尽量详细描述您遇到的问题，感谢您给我们提出建议"
                android:gravity="top|left" />

            <EditText
                android:id="@+id/feedback_contact_edit"
                android:layout_below="@id/feedback_content_edit"
                android:layout_width="match_parent"
                android:layout_height="45dp"
                android:layout_marginTop="@dimen/s3"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:background="#FFFFFF"
                android:textColorHint="@color/c4"
                android:textSize="@dimen/f4"
                android:textColor="@color/c3"
                android:hint="请输入您的手机号码、QQ或者邮箱"
                android:singleLine="true" />

            <TextView
                android:id="@+id/feedback_info_text"
                android:layout_below="@id/feedback_contact_edit"
                android:layout_width="match_parent"
                android:layout_height="wrap_content"
                android:layout_marginTop="@dimen/s3"
                android:paddingLeft="@dimen/s3"
                android:paddingRight="@dimen/s3"
                android:background="#00000000"
                android:textSize="@dimen/f5"
                android:textColor="@color/c5"
                android:text="留下联系方式，以便能更好更快的给您答复，还可能有意外奖励哦" />

            <!--<TextView
                android:id="@+id/feedback_ok_text"
                android:layout_width="match_parent"
                android:layout_height="45dp"
                android:layout_below="@id/feedback_info_text"
                android:layout_marginLeft="@dimen/s1"
                android:layout_marginRight="@dimen/s1"
                android:layout_marginTop="18dp"
                android:gravity="center"
                android:text="提交"
                android:textSize="@dimen/f2"
                android:textColor="@color/c11"
                android:background="@drawable/red_button_bg" />-->

            <include
                layout="@layout/b3_button"
                android:layout_width="match_parent"
                android:layout_height="48dp"
                android:layout_below="@id/feedback_info_text"
                android:layout_marginLeft="@dimen/s4"
                android:layout_marginRight="@dimen/s4"
                android:layout_marginTop="12dp"
                android:gravity="center"
                android:text="提交"
                android:textSize="@dimen/f2"
                android:textColor="@color/c11" />

        </RelativeLayout>
    </RelativeLayout>

</RelativeLayout>

```

查看界面的绘制情况，我们发现：
![image](http://img.blog.csdn.net/20160115124736670)

怎么样？蓝色的区域变成绿色了，绿色的区域变成淡红色了，淡红色的区域变成红色了，说明界面中存在过度绘制的情况，这时候我们就该考虑一下是否存在优化的可能了。


**其他UI优化的Tips**

- 使用LieanrLayout替代RelativeLayout布局；

- 布局文件中减少布局层级，减少组件的嵌套层数；

- 当有多个组件有相似的属性时，可以使用styles，复用样式定义；

- 通过定义drawable来替代图片资源的使用，降低内存消耗；


总结：

- 过度绘制描述的是屏幕上一个像素在单个帧中被重绘了多少次。
蓝色：绘制一次
绿色：绘制两次
淡红色：绘制三次
红色：绘制四次

- 为布局文件添加布局控件不会导致界面绘制次数的增加，只有为布局空间设置background时才会导致绘制次数的增加

- 当我们的界面红色区域较多时就需要考虑我们是否存在过度绘制的问题了

- 可以使用merge、include、ViewStub简化布局文件优化布局绘制流程

- 在使用布局文件的时候尽量减少布局层级


<br>另外对产品研发技术，技巧，实践方面感兴趣的同学可以参考我的：
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51685310">android产品研发（十一）-->应用内跳转scheme协议</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51690047">android产品研发（十二）-->App长连接实现</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51719389">android产品研发（十三）-->App轮训操作</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51764773">android产品研发（十四）-->App升级与更新</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51779528">android产品研发（十五）-->内存对象序列化</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51809497">android产品研发（十六）-->开发者选项</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51812985">android产品研发（十七）-->Hybrid开发</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51820139">android产品研发（十八）-->webview问题集锦</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51868451">android产品研发（十九）-->android studio中的单元测试</a>
<br><a href="http://blog.csdn.net/qq_23547831/article/details/51833080">android产品研发（二十）-->代码Review</a>

