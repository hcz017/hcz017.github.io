---
date: 2014-09-11 02:06
status: public
tags: Adapter
title: ArrayAdapter和SimpleAdapter简单使用
---

数据到视图的一般步骤：

1.新建一个数据适配器

2.适配器加载数据源

3.视图ListView加载适配器

- - -
- **1.ArrayAdapter**

```
public ArrayAdapter<String>(Context context, int resource, String[] objects)

public ArrayAdapter (Context context, int resource, T[] objects) 

context： The current context. 
resource： The resource ID for a layout file containing a TextView to use when instantiating  views. 
objects： The objects to represent in the ListView.
 ```
 应用：
 ```
String[] arr_data={"慕课网1","慕课网2","慕课网3","慕课网4"};
arr_adapter=new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1,arr_data);
 ```
第一个参数是上下文，就是当前的Activity, 
第二个参数是android sdk中自己内置的一个布局，它里面只有一个TextView，这个参数是表明我们数组中每一条数据的布局是这个view，就是将每一条数据都显示在这个view上面；
第三个参数就是我们要显示的数据。

- - -
- **2.SimpleAdapter**

```
public SimpleAdapter (Context context, List<? extends Map<String, ?>> data, int resource, String[] from, int[] to)

data:A List of Maps. Each entry in the List corresponds to one row in the list. The Maps contain the data for each row, and should include all the entries specified in "from"
resource:Resource identifier of a view layout that defines the views for this list item. The layout file should include at least those named views defined in "to"
from:A list of column names that will be added to the Map associated with each item.
to:The views that should display column in the "from" parameter. These should all be TextViews. The first N views in this list are given the values of the first N columns in the from parameter.
```
英文多了是不是太装逼。。。

context： 上下文
data：数据源（List<? extends Map<String, ?>> data）一个map所组成的list的集合。
	每一个Map都会对应ListView列表中的一行
	每一个Map（键值对）中的键必须包含在from中所指定的键
resource：列表项的布局文件id（layout）
from：Map中的键名
to：绑定数据是图中的id，与from成对应关系


代码示例（参考[慕课网](http://www.imooc.com)）

```
public class MainActivity extends ActionBarActivity {

	private ListView listView;
	private ArrayAdapter<String> arr_adapter;
	private SimpleAdapter simp_adapter;
	private List<Map<String,Object>>dataList;
	
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.main);
        listView=(ListView) findViewById(R.id.listView);
        //1.新建一个数据适配器
        //ArrayAdapter（上下文，当前LIstView加载每一项列表所对应的布局文件，数据源）
        //2.适配器加载数据源
        //SimpleAdapter()
        /**
         * context： 上下文
         * data：数据源（List<? extends Map<String, ?>> data）一个map所组成的list的集合。
         * 	每一个Map都会对应ListView列表中的一行
         * 	每一个Map（键值对）中的键必须包含在from中所指定的键
         * resource：列表项的布局文件id（可以是自定义layout）
         * from：Map中的键名
         * to：绑定数据是图中的id，与from成对应关系
        */
        String[] arr_data={"慕课网1","慕课网2","慕课网3","慕课网4"};
        dataList=new ArrayList<Map<String,Object>>();
		arr_adapter=new ArrayAdapter<String>(this, android.R.layout.simple_list_item_1,arr_data);
		simp_adapter=new SimpleAdapter(this, getData(), R.layout.item, new String[]{"image","text"}, new int[]{R.id.image,R.id.textView});
		//3.视图ListView加载适配器
		//listView.setAdapter(arr_adapter);
		listView.setAdapter(simp_adapter);
    }
    private List<Map<String, Object>>  getData(){
    	for(int i=0;i<20;i++){
    		Map<String,Object> map=new HashMap<String, Object>();
    		map.put("image", R.drawable.ic_launcher);
    		map.put("text","慕课网"+i);
    		dataList.add(map);
    	}
    	return dataList;
    	
    }
}


```

R.layout.item 是一LinearLayout 包含一个ImageView和一个TextView。