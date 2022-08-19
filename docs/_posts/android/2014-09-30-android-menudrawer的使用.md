---
date: 2014-09-30 23:33
status: public
tags: Android
title: android-menudrawer的使用
---


GitHub上发现一个挺不错的menudrawer，记录一下使用过程。

###MenuDrawer侧边菜单
- - -

地址为：[https://github.com/gokhanakkurt/android-menudrawe](https://github.com/gokhanakkurt/android-menudrawer) 包含独立的library和sampleDemo。
其实这个已经是两年前的版本了，此版本为Eclipse工程。新版的地址为：[https://github.com/SimonVT/android-menudrawer](https://github.com/SimonVT/android-menudrawer)
同时新版是AS项目。

在此感谢作者[SimonVT](https://github.com/SimonVT),项目使用说明可以到GitHub上查看。为了方便快捷我直接在Demo上修改的。

修改的部分代码：

    List<Object> items = new ArrayList<Object>();
    items.add(new Category("分类"));
    items.add(new Item("Home", R.drawable.ic_action_refresh_dark));
    items.add(new Item("Category", R.drawable.ic_action_select_all_dark));
    items.add(new Item("Archive", R.drawable.ic_action_refresh_dark));
    items.add(new Item("Album", R.drawable.ic_action_select_all_dark));
    items.add(new Item("Tags", R.drawable.ic_action_refresh_dark));
    items.add(new Item("Item 6", R.drawable.ic_action_select_all_dark));
            
初步修改后的效果如图：

![Image Title](http://ww4.sinaimg.cn/mw690/69443115jw1el77u5cu1mj20u01e0gpk.jpg)


###不同item下对应内容

作者提供的Demo点击不同的Item并不会像是不同的内容（只是更改了一个TextView显示的文字）。

原Demo中点击Item后执行的操作：

    private AdapterView.OnItemClickListener mItemClickListener = new AdapterView.OnItemClickListener() {
            @Override
            public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
                mActivePosition = position;
                mMenuDrawer.setActiveView(view, position);
                mContentTextView.setText(((TextView) view).getText());
                mMenuDrawer.closeMenu();
            }
    };
    
 用Fragment填充内容，先将```extends Activity```修改为```extends FragmentActivity```。之后重写  AdapterView.OnItemClickListener

    private AdapterView.OnItemClickListener mItemClickListener = new AdapterView.OnItemClickListener() {
        @Override
        public void onItemClick(AdapterView<?> parent, View view, int position, long id) {
            mMenuDrawer.closeMenu();
            if(position==1){
            	changeFragment(new HomeFragment());
            }else if (position == 2){
                changeFragment(new CategoryFragment());

            }else if (position == 3){
            	changeFragment(new ArchiveFragment());
            }
            System.out.println("position:"+position+"，id++++"+id);
        }
    };

获得点击Item的position，然后切换相应的Fragment的changeFragment()方法：

	public void changeFragment(Fragment targetFragment){
        getSupportFragmentManager().beginTransaction()
        		.replace(R.id.main_fragment, targetFragment, "fragment")
                .setTransitionStyle(FragmentTransaction.TRANSIT_FRAGMENT_FADE)
                .commit();
    }

项目已托管在GitHub上，地址为：[https://github.com/hcz017/FarboxBlog](https://github.com/hcz017/FarboxBlog)
项目已转为AS项目,地址：[https://github.com/hcz017/CodeSimple](https://github.com/hcz017/CodeSimple)，欢迎批评指正。
**参考：**
1. [GitHub MenuDrawer](https://github.com/gokhanakkurt/android-menudrawe)
1. [主Activity以frame填充Activity的方式交互管理Fragment](http://blog.sina.com.cn/s/blog_537d61430101bakx.html)    
2. [Android Fragment 基本介绍](http://www.cnblogs.com/mengdd/archive/2013/01/08/2851368.html)
3. [Android Fragment的使用(1)](http://www.cnblogs.com/xinye/archive/2012/08/28/2659712.html)