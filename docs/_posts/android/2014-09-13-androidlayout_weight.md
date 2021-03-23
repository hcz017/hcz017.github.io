---
date: 2014-09-13 16:48
status: public
tags: layout
title: android:layout_weight的真实含义(转)
---

首先声明只有在Linearlayout中，该属性才有效。之所以android:layout_weight会引起争议，是因为在设置该属性的同时，设置android:layout_width为wrap_content和match_parent会造成两种截然相反的效果。如下所示:

```
<LinearLayout  
       android:layout_width="match_parent"  
       android:layout_height="wrap_content"  
       android:orientation="horizontal" >  
  
       <TextView  
           android:layout_width="match_parent"  
           android:layout_height="wrap_content"  
           android:layout_weight="1"  
           android:background="@android:color/black"  
           android:text="111"  
           android:textSize="20sp" />  
  
       <TextView  
           android:layout_width="match_parent"  
           android:layout_height="wrap_content"  
           android:layout_weight="2"  
           android:background="@android:color/holo_green_light"  
           android:text="222"  
           android:textSize="20sp" />  
```

上面的布局将两个TextView的宽度均设为match_parent，一个权重为1，一个权重为2.得到效果如下:
![](http://img.blog.csdn.net/20140428205102328?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuemkxMjI1NjI3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

可以看到权重为1的反而占了三分之二！

再看如下布局:

```
<LinearLayout  
    android:layout_width="match_parent"  
    android:layout_height="wrap_content"  
    android:orientation="horizontal" >  
  
    <TextView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:layout_weight="1"  
        android:background="@android:color/black"  
        android:text="111"  
        android:textSize="20sp" />  
  
    <TextView  
        android:layout_width="wrap_content"  
        android:layout_height="wrap_content"  
        android:layout_weight="2"  
        android:background="@android:color/holo_green_light"  
        android:text="222"  
        android:textSize="20sp" />  
</LinearLayout>  
```

即宽度为wrap_content，得到视图如下：
![](http://img.blog.csdn.net/20140428205331750?watermark/2/text/aHR0cDovL2Jsb2cuY3Nkbi5uZXQveWFuemkxMjI1NjI3/font/5a6L5L2T/fontsize/400/fill/I0JBQkFCMA==/dissolve/70/gravity/Center)

左边 TextView占比三分之一，又正常了。

***android:layout_weight的真实含义是:一旦View设置了该属性(假设有效的情况下)，那么该 View的宽度等于原有宽度(android:layout_width)加上剩余空间的占比！***

设屏幕宽度为L，在两个view的宽度都为match_parent的情况下，原有宽度为L，两个的View的宽度都为L，那么剩余宽度为L-（L+L） = -L, 左边的View占比三分之一，所以总宽度是L+(-L)*1/3 = (2/3)L.事实上默认的View的weight这个值为0，一旦设置了这个值，那么所在view在绘制的时候执行onMeasure两次的原因就在这。

Google官方推荐，当使用weight属性时，将width设为0dip即可，效果跟设成wrap_content是一样的。这样weight就可以理解为占比了！

文章转自[yanzi1225627的专栏](http://blog.csdn.net/yanzi1225627/article/details/24667299)，感谢原作者。