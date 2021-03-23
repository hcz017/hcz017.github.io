---
date: 2017-04-19 17:44
status: public
title: 'CollapsingToolbarLayout 收缩时状态栏颜色设置不生效'
tags: SupportDesign
---

Android Design Support Library 中推出了一系列方便开发者实现Material Design concept app的widget，CollapsingToolbarLayout就是其中一个。
网上已有很多博客介绍CollapsingToolbarLayout的使用，所以今天我们不说这个widget怎么使用了，这里主要记录试一下在使用过程中遇到的问题。

如果还不知道怎么使用建议阅读一下文章： 

- [看，这个工具栏能伸缩折叠——Android CollapsingToolbarLayout使用介绍](http://www.jianshu.com/p/06c0ae8d9a96)
- [Material Design之CollapsingToolbarLayout使用](http://blog.csdn.net/u010687392/article/details/46906657])

#正文
CollapsingToolbarLayout有一个属性可以设置在折叠状态时状态栏的颜色：
app:contentScrim="?attr/colorPrimary"//一般会写成透明或者半透明
但是我发现，在写了这一行代码后没有生效，即颜色还是默认的颜色。
后调查发现：

不管是
1. 在布局文件里设置
    ```xml
    <android.support.design.widget.CollapsingToolbarLayout
            android:id="@+id/toolbar_layout"
            android:layout_width="match_parent"
            android:layout_height="match_parent"
            android:fitsSystemWindows="true"
            app:contentScrim="?attr/colorPrimary"
            app:statusBarScrim="@android:color/holo_purple"
            app:layout_scrollFlags="scroll|exitUntilCollapsed">
    ```
1. 还是在java代码里设置
    ```java
       collapsingToolbarLayout.setStatusBarScrimColor(Color.GREEN);
    ```

都需要先对状态栏先进行透明度的设置，三种方法：
1. 在style.xml中增加下面的代码把状态栏设置成全透明
    ```xml
        <item name="android:statusBarColor">@android:color/transparent</item>
    ```
1. 在style.xml中增加下面的代码把状态栏设置成半透明
    ```xml
       <item name="android:windowTranslucentStatus">true</item>
    ```
1. 在java代码中onCreate()方法里把状态栏设置成半透明
    ```java
       getWindow().addFlags(WindowManager.LayoutParams.FLAG_TRANSLUCENT_STATUS);
    ```
如果你已经在**style**里面把状态栏设置成了透明，那么CollapsingToolbarLayout也想要透明效果的话就无需设置了。

设置半透明和透明的效果如下（两边都是紫色）：
![](./_image/ADW_collapsingToolbarLayout.gif-csdn)