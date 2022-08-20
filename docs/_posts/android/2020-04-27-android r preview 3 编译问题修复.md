---
date:  2020-04-27 12:08
title:  'android R preview 3 编译问题修复'
---


android 11 的x86 模拟器支持运行arm 应用了，但是官方提供的模拟器不能root，于是想要自己编译userdebug 版本。  
没那么顺利，开始编译后立刻报错了。

**不关心过程的直接看最后总结**

# 错题提示
错误关键输出
```bash
error: external/seccomp-tests/Android.bp:20:13: unrecognized property "arch.mips"
error: external/seccomp-tests/Android.bp:23:15: unrecognized property "arch.mips64"
error: external/linux-kselftest/Android.bp:53:13: unrecognized property "arch.mips"
error: external/linux-kselftest/Android.bp:56:15: unrecognized property "arch.mips64"
```

截图

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-1-error.png)

如果你单纯的去这两支文件里屏蔽掉相关代码段还会报其他错误。

# 寻找解决方案
期间看到这问朋友的博客：[Android R preview编译失败](https://blog.csdn.net/u013398960/article/details/105216011)，看到 build/soong/android 目录下可能有点东西。  
那我们去build 目录下找找线索，有个叫Elliott 的老哥貌似有在做相关修改。


![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-Elliott-commit.png)

我们去aosp gerrit 上搜他。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-Elliott-gerrit.png)

这老哥好像最近专门在搞这个，可以看到已经在master 分支上做了修改。那么我们本地preview 3 的分支是什么情况呢？

查看git log，本地preview 3 分支上这两个目录的上次提交还是在一年前，遂**checkout master 分支，把对应的修改拉下来**。  
（上图和下图的时间对不上，那是因为commit 的时间和push 的时间不一样，change id 是一样的）

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-branch.png)

build....

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-build-1.png)

---

更新编译进度

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-jni-error.png)

fixing...

把这个提交cherry-pick 下来

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-fix-jni.png)

building...

---

更新编译进度

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-build-2.png)

**这里是返回一个布尔值，暴力点的话可以直接强制return true 解决。**

正规方法的话，上aosp 上扒提交，简单说就是两个timezone 的更换与更新。但是提交不止这两个。所以我强制改返回值了。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-build-2-fix.png)

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-build-2-fix2.png)
building...

---
**成了！**

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/and-r-cpl-success.png)

# 解法总结
1. external/linux-kselftest 仓库切换到master 分支，如果master 太新有其他问题就只cherry-pick mips 那条提交；
2. externl/seccomp-tests 仓库切换到master 分支，如果master 太新有其他问题就只cherry-pick mips 那条提交；
3. libcore 仓库从master 分支cherr-pick 一个提交修复错误，"Replace jniStrError with strerror_r"；
4. 强制修改 frameworks/base/core/java/android/timezone/CountryTimeZones.java 中的matchesCpuntryCode 那行 renturn true；
5. 换个高配电脑，否则可能因内存不足编译终止。