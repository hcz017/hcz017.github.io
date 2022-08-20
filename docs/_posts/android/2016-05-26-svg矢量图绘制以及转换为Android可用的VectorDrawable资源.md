---
date:  2016-05-26 00:20
title:  'svg矢量图绘制以及转换为Android可用的VectorDrawable资源'
---

项目需要 要在快速设置面板里显示一个图标（为了能够区分出来图形，我把透明的背景填充为黑色了）

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_1_vowifi.png)

由于普通图片放大后容易失真，这里我们最好用矢量图（SVG(Scalable Vector Graphics)）来做图标，而系统状态栏图标多是用vectorDrawable绘制，所以我们的目的就是绘制一个上图中样式的VectorDrawable图标。尤其是这种资源文件体积小放大又不失真，干嘛不用呢。
# VectorDrawable
Android L开始提供了新的API VectorDrawable 可以使用SVG类型的资源，也就是矢量图。在xml文件中的标签是<vector>
google官方API介绍：
https://developer.android.com/reference/android/graphics/drawable/VectorDrawable.html



This lets you create a drawable based on an XML vector graphic. It can be defined in an XML file with the <vector> element.

The vector drawable has the following elements:

具体属性和方法请参考官方说明

下面是一个官方例子：
```xml
<vector xmlns:android="http://schemas.android.com/apk/res/android"
    android:height="64dp"
    android:width="64dp"
    android:viewportHeight="600"
    android:viewportWidth="600" >
    <group
        android:name="rotationGroup"
        android:pivotX="300.0"
        android:pivotY="300.0"
        android:rotation="45.0" >
        <path
            android:name="v"
            android:fillColor="#000000"
            android:pathData="M300,70 l 0,-70 70,70 0,0 -70,70z" />
    </group>
</vector>
```
显示效果（背景色应为透明）

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_2-sample.png)



# 绘制svg图
如果想了解绘制原理，调至请调至文末点击W3C的连接。


接下来介绍一些常用的svg绘图工具
## 1.Inkscape
开源的多平台矢量图绘图工具，支持windows OS X Linux。支持导出为svg等格式图片，功能强大，与后面两个将要介绍的比较就是体积有点大，安装包就接近百兆了。
另外用这个生成的SVG文件，会带一些默认的属性，转化成VectorDrawable以后xml文件里也会有一些默认的属性，虽不影响显示效果，但会多出一些不必要的代码。
工作界面：
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_3-Inkscape.png)

官网：https://inkscape.org/
## 2.Boxy SVG

是一个Chrome应用（推荐）。支持导入，另存为，可以选中单个控件调整属性等。可能不好的地方就是你得安装Chrome浏览器吧，还有下载这个应用的时候得翻墙。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_4-Boxy-SVG.png)

## 3.Janvas - The Online Vector Graphics Editor
也是Chrome应用，不过其实就是一个链接，打开后指向下面的地址
http://www.janvas.com/XOSYSTEM/PROJECTS/janvas_apps_suite_3.0_public/janvas_application.php
但是这个在线编辑器好像只能打开和保存文件到google driver，不推荐

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_5_Janvas.jpg)

## 4.？？
这个东西没找到名字，点击下面的连接试用。添加到收藏夹，随时可用。便捷。
http://www.yyyweb.com/ctools/demo.php?t=http%3A%2F%2Feditor.method.ac%2F&h=2000&c=&n


# 转换为VectorDrawable
找到两个在线转换的工具，都是Github上的开源项目。
## 1.Android SVG to VectorDrawable
Convert SVG to Android VectorDrawable XML resource file.

可能是这个工具开发比较早，有很多Star，基本的图形转换是可以的，但是，不支持文字！也就是说上面的图，如果我们转换的话，得到的结果只是一个椭圆，文字会丢失。  
在线工具：http://inloop.github.io/svg2android/  
源码地址：https://github.com/inloop/svg2android  

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_6_SVG-to-VectorFrawable.png)

## 2.SvgToVectorDrawableConverter.Web
Batch converter of SVG images to Android VectorDrawable XML resource files.

这个就比较比较强大了，支持文字转换，貌似转换后的格式也比较规范。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_7_SvgToVectorDra.png)

在线工具：http://a-student.github.io/SvgToVectorDrawableConverter.Web/  
源码地址：https://github.com/a-student/SvgToVectorDrawableConverter  

# 效果图
这里我把颜色改回了白色。使用的是Boxy SVG绘制，SvgToVectorDrawableConverter.Web转换。
Android Studio支持直接预览VectorDrawable矢量图，有了实时预览，也方便进行一些简单的修改。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/svg_8_result.png)

这个图标最后应用到下拉的快速设置里面，在手机上的效果图就不上了。

# 总结
本文简单介绍了几款工具，目的能让新手快速的了解一下如何制作出自己需要的矢量图资源文件，在有需要做一张应用到Android应用/系统的矢量图时不至于措手不及。当然如过你牛逼到直接用记事本“绘图”的话，本文应该不适合你。

我发现我特别喜欢发掘一些能够提高生产力的小工具啊，哈哈哈。

# 其他
知其然不知其所以然？想要了解其原理，请跳转到W3C查看Scalable Vector Graphics (SVG) 1.1 (Second Edition)
https://www.w3.org/TR/SVG11/Overview.html