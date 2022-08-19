---
title: Android自定义TitleBar
date: 2014-12-06 12:12
url: CoutomTitleBar
---

系统自带的控件不足以满足我们的需求时，我们可以自定义控件。

一般我们的程序中可能有很多个活动都需要自定义的标题栏，如果在每个活动中的布局中都编写一遍同样的标题栏代码，明显会导致代码的大量重复。这时候我们就可以引入布局的方式来解决这个问题。
首先创建标题栏的布局文件title.xml，代码如下
	
	<?xml version="1.0" encoding="utf-8"?>
	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:orientation="horizontal"
	    android:layout_width="match_parent"
	    android:layout_height="wrap_content"
	    android:background="#ff5bc842">
	    <Button
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/btn_back"
	        android:text="back"
	        android:background="#ff3c8a2f"/>
	    <TextView
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/title"
	        android:text="title"
	        android:textAppearance="?android:attr/textAppearanceMedium"
	        android:layout_weight="1"
	        android:gravity="center" />
	
	    <Button
	        android:layout_width="wrap_content"
	        android:layout_height="wrap_content"
	        android:id="@+id/btn_edit"
	        android:text="edit"
	        android:background="#ff3c8a2f"/>
	
	</LinearLayout>
效果如图:  
![](http://ww4.sinaimg.cn/mw690/69443115jw1enboimxsyuj20780c2glk.jpg)

标题栏编写完成，如果我们要在程序中使用这个标题栏，需要修改activity_main.xml中的代码

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical">
	
	  	<include layout="@layout/title"></include>

	    <ListView
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:id="@+id/lv">
    	</ListView>
	
	</LinearLayout>

然后去掉原来的标题栏：

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
		requestWindowFeature(Window.FEATURE_NO_TITLE);
        setContentView(R.layout.activity_main);
    }

效果：  
![](http://ww4.sinaimg.cn/mw690/69443115jw1enbotil5xhj20780c23yr.jpg)

在任意布局里面想要添加自定义的titlebar，只需用	`<include layout="@layout/title"></include>`即可。

只做到这一步的话，虽然不用重复编写标题栏代码了，但是在每个使用了此标题栏的活动中，要对标题栏上的控件响应还是要在每个活动中编写相应的代码。这种情况最好是使用自定义控件的方式解决。
新建TitleLayout继承自LinearLayout，让它策划能够为我们自定义的标题栏控件，代码如下：

	public class TitleLayout extends LinearLayout {
	    public TitleLayout(Context context, AttributeSet attrs) {
	        super(context, attrs);
	        LayoutInflater.from(context).inflate(R.layout.title,this);
			//为按钮注册点击事件
	        Button titleBack = (Button) findViewById(R.id.btn_back);
	        Button titleEdit = (Button) findViewById(R.id.btn_edit);
	        titleBack.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                ((Activity)getContext()).finish();
	            }
	        });
	        titleEdit.setOnClickListener(new OnClickListener() {
	            @Override
	            public void onClick(View v) {
	                Toast.makeText(getContext(),"You clicked EDIT!",
					Toast.LENGTH_SHORT).show();
	            }
	        });
	    }
	}
同时修改activity_main.xml中的代码：

	<LinearLayout xmlns:android="http://schemas.android.com/apk/res/android"
	    android:layout_width="match_parent"
	    android:layout_height="match_parent"
	    android:orientation="vertical">
	
	    <com.example.hcz.customtitlebar.TitleLayout
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content">
	    </com.example.hcz.customtitlebar.TitleLayout>
	
	    <ListView
	        android:layout_width="match_parent"
	        android:layout_height="wrap_content"
	        android:id="@+id/lv">
	    </ListView>
	
	</LinearLayout>
添加自定义控件的时候要指明控件的完整类名，包名不可省。

效果：

<img src="http://ww2.sinaimg.cn/mw690/69443115jw1enbpcyc2bdj20u01e0wfv.jpg"width="350"/>
这样之后，每当我们再要个布局中引入TitleLayout，返回按钮和编辑按钮的点击事件都已经自动实现好了，也是省去了很多重复编写代码的工作。