---
title: Android中的ContextMenu
date: 2014-12-16 14:48
tag: ContextMenu
url: androidContextMenu
---

Android 的ContextMenu，即上下文菜单。（类似电脑上的鼠标右键功能，选中某个元素，然后点右键，在弹出菜单上执行操作。）在手机上，通过长时间按住界面上的元素，就会出现事先设计好的上下文菜单。

实现ContextMenu，一般要用到以下三个方法：

(1)registerForContextMenu(getExpandableListView());//注册上下文菜单 

(2)onCreateContextMenu(ContextMenu menu, View v,ContextMenuInfo menuInfo);//创建上下文菜单

(3)onContextItemSelected(MenuItem item);//上下文菜单的选中事件  

Tips:Options Menu的拥有者是Activity，而上下文菜单的拥有者是Activity中的View。每个Activity有且只有一个Options Menu，它为整个Activity服务。而一个Activity往往有多个View，并不是每个View都有上下文菜单，这就需要我们调用registerForContextMenu(View view)来指定。

上代码：
	
	import android.app.Activity;
	import android.os.Bundle;
	import android.support.v7.app.ActionBarActivity;
	import android.view.ContextMenu;
	import android.view.Menu;
	import android.view.MenuItem;
	import android.view.View;
	import android.view.Window;
	import android.widget.ArrayAdapter;
	import android.widget.ListView;
	import android.widget.Toast;
	import java.util.ArrayList;
	
	
	public class MainActivity extends Activity {
	
	    @Override
	    protected void onCreate(Bundle savedInstanceState) {
	        super.onCreate(savedInstanceState);
	        requestWindowFeature(Window.FEATURE_NO_TITLE);
	        setContentView(R.layout.activity_main);
	        showListView();
	    }
	
	    private void showListView(){
	        ListView listView = (ListView) findViewById(R.id.lv);
	        ArrayAdapter<String> adapter=new ArrayAdapter<String>(this,android.R.layout.simple_list_item_1,getData());
	        listView.setAdapter(adapter);
	        this.registerForContextMenu(listView);//注册上下文菜单
	    }
		
		//创建菜单		
	    @Override
	    public void onCreateContextMenu(ContextMenu menu, View v, ContextMenu.ContextMenuInfo menuInfo) {
	        super.onCreateContextMenu(menu, v, menuInfo);
	        //设置mune显示的内容
	        menu.setHeaderTitle("文件操作");
	        menu.setHeaderIcon(R.drawable.ic_launcher);
	//         public MenuItem add(int groupId, int itemId, int order, CharSequence title);
	        menu.add(1,1,1,"copy");
	        menu.add(1,2,1,"cut");
	        menu.add(1,3,1,"past");
	        menu.add(1,4,1,"cancel");
	    }
		//响应菜单
	    @Override
	    public boolean onContextItemSelected(MenuItem item) {
	        switch (item.getItemId()){
	            case 1:
	                Toast.makeText(this, "clicked copy",Toast.LENGTH_SHORT).show();
	                break;
	            case 2:
	                Toast.makeText(this, "clicked cut",Toast.LENGTH_SHORT).show();
	                break;
	            case 3:
	                Toast.makeText(this, "clicked past",Toast.LENGTH_SHORT).show();
	                break;
	            case 4:
	                Toast.makeText(this, "clicked cancel",Toast.LENGTH_SHORT).show();
	                break;
	        }
	        return super.onContextItemSelected(item);
	    }
	
	    private ArrayList<String> getData(){
	        ArrayList<String> list=new ArrayList<>();
	        for(int i=0;i<5;i++){
	            list.add("file"+(i+1));
	        }
	        return list;
	    }
	｝

效果：
![](http://ww3.sinaimg.cn/mw690/69443115jw1enbiwrpqwgj20u01e00wp.jpg)

**参考：**

1. [Android 菜单(ContextMenu)](http://stephen830.iteye.com/blog/1130637)  
2. [Android之ContextMenu的使用方法以及与OptionMenu的区别](http://blog.csdn.net/pfgmylove/article/details/7560290)

感谢原作者。