---
date:  2020-09-11 12:43
title:  'gradlew 编译中的ANDROID NDK 环境变量'
---
系统环境 **windows 10 + gradle 6.1.1**
# Android Studio 配置
当前使用Android Studio 构建app，使用NDK 的话，会有两处配置项（其实非必须配置）。


1. local.properties 用ndk.dir 指定ndk 路径（含版本号）  
![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_local.properties.png)
2. 删除local.properties 中的dir配置，在app/build.gradle 中配置 android{ndkVersion}。注意此版本号对应的ndk 版本需存在，否则会报错。  
![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_build.gradle.png)


# 使用环境变量
注意 windows 环境下一定要用cmd，使用Android Studio 的terminal 或者Windows Terminal 的话环境变量不会起作用。
## ANDROID_NDK_HOME
关于使用ANDROID_NDK_HOME环境变量配置，指定ndk 路径（含版本）的，经验证这个环境变量无效。

![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_NDK_build_mid_file.png)

虽然可以找到环境变量，但是deprecated，最后编译的时候还是会去找到sdk 目录下的较新版ndk。
搞笑的是，此时如果用Android Studio 去编译的话，由于没有ndk.dir 和build.gradle 的配置，编译的时候回去sdk 目录下找ndk 目录，然后默认会用一个ndk 版本（猜测和Android Studio 版本有关），如果这个默认版本不存在，编译会报错。

## ANDROID_NDK_ROOT
这个ANDROID_NDK_ROOT 也是无效，虽然google 论坛 [Recommended NDK](https://groups.google.com/g/android-ndk/c/qZjhOaynHXc) Directory? (2013)上说 没有 ANDROID_NDK_HOME

甚至开源项目 [openssl](https://github.com/openssl/openssl/issues/11205) 也按照这个修改了。但是实测修改后无效（无论包不包含版本文件夹），而且会报错默认版本的ndk 找不到，还不如ANDROID_NDK_HOME呢。

## 官方资料
[NDK download](https://developer.android.com/ndk/downloads) 页面示例的是修改app/build.gradle 指定ndk 版本。

![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_official_NDK_DL.png)

[Get started with Vulkan](https://developer.android.com/ndk/guides/graphics/getting-started) 页面显示 ANDROID_NDK_ROOT 已经弃用，未来会被移除。
不过这里同框居然出现了ANDROID_NDK_HOME，excuse me???

![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_Vulkan_page.png)

最后在[Install and configure the NDK and CMake](https://developer.android.com/studio/projects/install-ndk#default-version) 页面下又看到如果你Android Gradle plugin 版本高于3.5， 那就把local.properties 文件里的ndk.dir 删掉把，我们（指Google 把它也弃用了）

![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/nkd_cfg_install_NDK_and_Cmake.png)


捋一下， 2013年的时候google 论坛上的大佬说不存在什么ANDROID_NDK_HOME，应该用ANDROID_NDK_ROOT。但是Google 自己网页上和工具上实际检查/使用的的还是ANDROID_NDK_HOME。然后Android Studio 这边起先用的local.properties  里的 ndk.dir，后来gradle 插件升级了，又换了用 app/build.gradle 里配置 ndkVersion 的方式。

王德发--