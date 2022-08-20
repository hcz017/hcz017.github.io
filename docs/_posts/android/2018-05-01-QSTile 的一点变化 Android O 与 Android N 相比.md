---
date: 2018-05-11 13:03
status: public
title: 'QSTile 的一点变化 Android O 与 Android N 相比'
---

# 起因
Android O 点击QSPanel 中的数据图标没法展开详情查看数据使用量，而在Android N 上可以的，所以想调查看看O 上有什么不同。
注：本文不涉及任何流程分析，只是单纯的想找回个功能，并借此描述一下O 和N 在tile 上的一点点不同。

# 现象差异
我们都知道在Android N 上QSTile 有两种展示形式:
1. QSPanel 半展开是点击icon 即开关，执行的方法为handleSecondaryClick();
2. QSPanel 全展开时，点击icon 有些tile 有detai展示，比如下图中的wifi tile，此时执行的方法是handleClick()，之后调用showDetail() 方法显示详细信息;

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_N_turn_on_wifi_2.gif)

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_N_turn_on_wifi_expanded.gif)

但是到了Android O 上变的有点不一样了：
1. 半展开时点击icon 和Android N 上效果一样，但执行的方法是handleClick()；
2. 全展开时有两种情况，点击**icon**还是开关，此时执行的方法也是handleClick()；
3. 全展开时点击**label** 会展示detail（如果有的话），有detail 的tile 在label 旁边会有一个**indicator**，而且注意观察的话，会发现点击icon 和点击label 水波纹的中心会不一样（如果没有detail 的话，点击label 也是上一种效果）。此时执行的方法是handleSecondaryClick()

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_O_turn_on_wifi.gif)

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_O_turn_on_wifi_expanded.gif)

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_O_turn_on_wifi_expanded_detail.gif)

那么Android O 上是怎么会有icon 和label 两种点击事件的呢？

# Tile 布局
我们要想label 可以点击，那我们先看看这个indicator 是怎么回事，从布局看起。
找到几个和tile，layout 相关的类，并没有发现线索，直到在qs_tile_label.xml 中看到了下面的代码：
```xml
        <!-- tile label -->
        <TextView
            android:id="@+id/tile_label"
            ...
            android:textAppearance="@style/TextAppearance.QS.TileLabel"
            android:textColor="?android:attr/textColorPrimary"/>
        <!--  下面这个暂时不知道是做什么的 -->
        <ImageView android:id="@+id/restricted_padlock" 
            ...
            android:visibility="gone" />
        <!--  像是旁边的indicator，且默认不可见，怀疑是这个了 -->            
        <ImageView
            android:id="@+id/expand_indicator" 
            android:layout_marginStart="4dp"
            android:layout_width="18dp"
            android:layout_height="match_parent"
            android:src="@drawable/qs_dual_tile_caret"
            android:tint="?android:attr/textColorPrimary"
            android:visibility="gone" /> 
```
之后在QSTileView.java 类中找到使用的地方:

```java
    protected void createLabel() {
        mLabelContainer = (ViewGroup) LayoutInflater.from(getContext())
                .inflate(R.layout.qs_tile_label, this, false);
        ...
        mExpandIndicator = mLabelContainer.findViewById(R.id.expand_indicator); // here
        mExpandSpace = mLabelContainer.findViewById(R.id.expand_space);

        addView(mLabelContainer);
    }
```
以及控制显示的地方
```java
    @Override
    protected void handleStateChanged(QSTile.State state) {
        super.handleStateChanged(state);
        ...
        mExpandIndicator.setVisibility(state.dualTarget ? View.VISIBLE : View.GONE); // 显示与否依赖于 state.dualTarget
        mExpandSpace.setVisibility(state.dualTarget ? View.VISIBLE : View.GONE);
        mLabelContainer.setContentDescription(state.dualTarget ? state.dualLabelContentDescription
                : null);
        if (state.dualTarget != mLabelContainer.isClickable()) {
            mLabelContainer.setClickable(state.dualTarget); // 设置是否可以点击
            mLabelContainer.setLongClickable(state.dualTarget);
            mLabelContainer.setBackground(state.dualTarget ? newTileBackground() : null);
        }
        mLabel.setEnabled(!state.disabledByPolicy);
        mPadLock.setVisibility(state.disabledByPolicy ? View.VISIBLE : View.GONE);
    }
```
可以看到indicator 的显示受state.dualTarget 的控制。
考虑到另外也有几个可以展开detail 的tile，比如Bluetooth等，我猜想state.dualTarget 一定会在那几个tile 里面赋值为true。我们搜一下，果然：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_O_dualTarget_tiles.png)

那我们在CellularTile 对应的位置也给dualTarget 赋值为true 应该就能达到我们最初想要的效果了。

# 结论和效果
添加`state.dualTarget = true;` 到CellularTile.java 的handleUpdateState() 方法后，效果如下：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android_O_mobile_data_tile_click.gif)

**state.dualTarget 控制了是否显示indicator 以及label 是否可以点击**，但点击有没有反应要看有没有实现相应的方法了。
点击mobile date 这里并不是凭空多出来一个功能哈，是我知道有这个功能，只是没显示出来。

注：文中代码基于mko-mr1，一个基于aosp 的android 8.1 分支。