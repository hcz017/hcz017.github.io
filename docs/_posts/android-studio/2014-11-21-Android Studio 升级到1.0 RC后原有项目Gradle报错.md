---
date: 2014-11-21 17:57
status: public
tags: android-studio
Title: Android Studio升级1.0后,如何从GitHub导入项目以及对原项目的Gradle配置修改
url: gradle4Androidstudio1.0
---

今天Android Studio更新到了1.0 RC版。官方说明翻译过来如下：

---
>此版本包括大量的 bug 修复；更新了 splash 屏幕和全新的图片，包括新的 logo；IDE 目录也从 AndroidStudioBeta 改为了 AndroidStudio，当你首次运行这个版本的时候，应该要从 beta 设置目录导入你的设置。

>![Image Title](http://www.php-z.com/data/attachment/portal/201411/21/100837jxfa7fnxtewxreef.png)

>现在，我们绑定了一个本地 Maven 库，包括 Gradle 插件和其所有的依赖，没有网络连接的情况下也可以创建新项目（这也是为什么这个 patch 会这么大的原因）。
此版本在 Canary Channel 可以下载。

>![Image Title](http://www.php-z.com/data/attachment/portal/201411/21/100837oo6mnb36zo3blblj.png)

---

但是软件更新后gradle频繁出错。是因为新版AS要求Gradle 0.14.+版本，AS 0.8要求0.12+版本，升级软件之后导致之前的一些属性和配置要修改。
以下这些属性改名，原先的不能用:
>runProguard -> minifyEnabled (是否混淆)  
zipAlign -> zipALignEnabled (是否zip对齐)  
packageName -> applicationId  
jniDebugBuild-> jniDebuggable  
renderscriptDebug->renderscriptDebuggable  
renderscriptSupportMode->renderscriptSupportModeEnabled  
renderscriptNdkMode->renderscriptNdkModeEnabled

---
# “Check out from GitHub”错误 #
如果出现以下或其他错误提示的（错误图列不一一列举），请参照下面的说明解决。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/QQ截图20141121180248.png)
**解决方法：**
点击上面错误提示的OK，然后按照下图设置，之后先不要点OK，手动去修改文件。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/importprojectfromgradle.png)
1. 修改项目目录下的build.gradle。修改后内容如下：

        classpath 'com.android.tools.build:gradle:0.12.2'         
为：

        classpath 'com.android.tools.build:gradle:0.14.4'
        
2. 修改项目目录下gradle\wrapperapp\gradle-wrapper.gradle

        distributionUrl=http\://services.gradle.org/distributions/gradle-1.12-all.zip
为：
        
         distributionUrl=http\://services.gradle.org/distributions/gradle-2.2-all.zip
    
3. 修改app目录下的build.gradle
        
            buildTypes {
            release {
        //      runProguard false
              proguardFiles getDefaultProguardFile('proguard-android.txt'), 'proguard-rules.pro'
            }
          } 
   注释掉runProguard，或者按照上面的说明修改属性名。如果漏了其他属性没有修改，点击OK后Android Studio 会给出相应的提示，`Gradle DSL method not found: “***()” ，把***()注释或修改就好了。`
4. 如果有library文件（库）把库文件夹下的build.gradle做同3一样的修改。

# 原有的项目错误 #

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/gradlebaocuo.jpg)
通常这时候“Jump to source”后把“runProguard()”（或其他的）删掉就好了。如果不行请尝试一下上面的1234步骤。
PS：如果项目比较特殊，以上方法不保证解决出现的问题。

PPS：反正我是这么改的，成不成你们试试看（逃）。