---
date: 2015-09-06 15:33
status: public
tags: 'android-studio'
---

本文是以源码中development/tools/idegen/README作为指导文档，给出了使用Android Studio导入Android源码的方法步骤。
由于Android Studio（以下简称AS）是基于IntelliJ IDEA开发的，所以本文也适用于IntelliJ IDEA。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/0.AS_startup.jpg)

### 零、为啥用Android Studio
1. 智能代码提示
2. 自动保存
3. 多设备实时预览
4. 内置终端
5. UI漂亮
6. 自带git github svn插件
7. 强大的搜索

### 一、修改AS的配置
由于Android源码太大了，在过导入源码和后续工作中，AS需要占用大量内存，所以我们要先做些设置。
在2.3.1以后的版本中修改/home/username/.AndroidStudio2.3/studio64.vmoptions文件，增加-Xms748m -Xmx4096m，也可通过help->Edit Custom VM Options修改。

### 二、生成导入AS所需配置文件(*.ipr)
为了成功将源码导入AS，我们需要先生成AS可是别的项目工程配置文件。
在源码根目录依次执行  
```bash
source build/ensetup.sh  
make idegen
development/tools/idegen/idegen.sh
```
之后会出现类似下面的结果:
```bash
Read excludes: 5ms  
Traversed tree: 44078ms
```
这时会在源码的根目录下生成android.ipr，android.iws和android.iml三个文件。

### 三、导入源码
android源码代码量较大，全部导入比较耗时，因此在打开前可以先去除一些不想导入的代码文件夹。
如果我们一直都不会去修改prebuilt文件夹下的代码，可以先修改android.iws，在合适的位置添加：`<ignored path="$PROJECT_DIR$/prebuilt/" />`。
之后再打开源码目录下的android.ipr。
另外，第一次导入源码需要生成index，因此花费的时间比较长，完整扫描一次以后再打开就没这么慢了。

### 四、配置AS的JDK、SDK
在上一步操作之后的等待期间刚好让我们来配置一下JDK和SDK。
在AS中参照下图Project Structure设置，在SDKs设置中加入必须的JDK，SDK。
创建一个新的JDK,可以取名为1.7(No Libraries)，然后删除classpath标签页下面的jar文件。 这样可以确保使用Android源码里的库文件。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/1.AS_JDK_Nolibs.jpg)


之后将1.7(No Libraries)作为Android SDK要使用的Java SDK。如下图

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/2.AS_JDK_Android.jpg)


之后在Project标签中的Project SDK中选择对应的Android API版本。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/3.AS_JDK_SDK.jpg)


### 五、debug源码
我们可以通过给刚导入的工程在'Modules'中添加'Android Framework'来让AS将它作为一个Android工程，从而方便我们调试代码。选择途中Framework目录下的Android

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/6.AS_Module_Android.jpg)


在代码中加断点，然后选择'Run'->'Attach debugger to Android process'或者直接点击下图中的手机上有个虫子的图标。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS2_AS_toolsbar_debug.jpg)


在弹出的选择进程(Choose Process)对话框中，先勾选显示所有进程，然后选择要debug的代码所在的进程，点击OK即可（可同时debug多个进程）。

### 六、解决源码中跳转错误问题
设置'Modules'的依赖

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/4.AS_Modules.jpg)


将除了Module source和Android API以外所有的依赖删掉。
点击上图中'+'并选择'Jars or directories'选项，将frameworks文件夹添加进来。如:

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/5.AS_Module_fram5.ework.jpg)


如果之后还是遇到了代码跳转错误，请仿照上面的步骤将相应代码的路径或jar文件添加到其Dependencies标签页中。

最近我在几个英文网站上看到把out/target/common/R文件夹标记为Sources，我猜测是为了解决找不到R资源的问题，但是我测试没有生效。读者自行选择吧。

### 七、快捷键
快捷键是利器啊！熟悉了快捷键效率飙升！不过有些与系统快捷键冲突了，若要实行请自行修改。
全部快捷键请看另一篇文章：[IDEA 快捷键 Android Studio快捷键](http://blog.csdn.net/aaa111/article/details/43791481)

---

Tips:
因为Android Studio 的配置和缓存文件存在home/.AndroidStudio文件夹中，时间长了可能会导致系统磁盘吃紧，若要修改默认存储位置（比如改到其他挂在盘），需修改android-studio/bin/idea.properties文件中相关的配置信息，修改内容参考：
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS2_config_cache.png)

注：前面生成的文件分别作用为：
①android.iws 包含工作区的个人设置，比如打开过的文件，版本控制工具的配置，本地修改历史，运行和debug的配置等。
②android.ipr 一般保存了工程相关的设置，比如modules和modules libraries的路径，编译器配置，入口点等。
③android.iml 用来描述modules。它包括modules路径、 依赖关系，顺序设置等。一个项目可以包含多个 *.iml 文件。

你可能需要的链接：
1. [Android Studio系列教程](http://stormzhang.com)
2. [如何使用Android Studio开发/调试Android源码](http://www.cnblogs.com/Lefter/p/4176991.html)
3. [Ubuntu下配置Android Studio的快捷启动方式](http://blog.csdn.net/aaa111/article/details/41833179)
4. [Android Studio简单设置](http://ask.android-studio.org/?/article/14)
5. [Android Studio 常用功能介绍](http://ask.android-studio.org/?/article/23)
6. [配置Android Studio](https://developer.android.com/studio/intro/studio-config.html)

感谢原作者！