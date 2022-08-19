---
date:  2022-06-17 10:41
title:  'Android 源码环境下用Android.mk 导入aar'
---

本文讨论两个问题：
1. 如何在Android 源码环境下使用Android.mk 的方式把aar 导入apk（apk 引用aar 的内容）；
2. 含有so 文件的aar 在源码环境下编译后so 文件不会被打包进apk，如何解决app 运行时找不到so 的问题。

现在新的源码中很多app 已经切换到了Android.bp，不过目前Android.mk 还是兼容的。下面介绍在Android.mk 中导入aar 包。

源码环境是qcom SM8450
# 在Android.mk 中导入aar 包
第一步：先声明 aar 包的位置

```makefile
include $(CLEAR_VARS)
LOCAL_PREBUILT_STATIC_JAVA_LIBRARIES += aar_alias:libs/aar-release_1.0.aar
include $(BUILD_MULTI_PREBUILT)
```
注意上面的代码整段都需要，而不只是中间那一行。

第二步：引用我们声明的 aar 变量

```makefile
LOCAL_STATIC_JAVA_AAR_LIBRARIES := aar_alias
```
第三步：添加引用的 aar 包里面的资源

```makefile
LOCAL_AAPT_FLAGS += \
         --auto-add-overlay \
         --extra-packages com.example.aar
```
com.example.aar 为aar 的包名。

第二步和第三步是添加到apk 的 include \$(CLEAR\_VARS) 和include \$(CLEAR\_VARS) 中间。

---
# 运行时找不到so（aar 里的so）
如果aar 中含有so 文件的话，通常使用Android Studio 构建apk 的话，so 会打包到apk的lib 目录下（解压后可以查看到），而在源码下使用Android.mk 的方式编译的话，编译出的apk 是不含so 文件的，也就意味着如果你adb install 编译出来的apk，它是不能按预期运行的。

两种方法解决这个问题

## 方案一 预置apk + 预置aar 里的so 到/system/lib64/ 目录
使用Android.mk 的方式在源码下编译，大概率也是要预置这个apk 了，可以在编译apk 的mk 文件中增加预置so 文件的代码。当然，首先要把aar 里的so 文件解压出来（集成交付可能不太友好，除了更新aar 还要再一次把aar 里的so 解压出来）。

```makefile
#  jni so
include $(CLEAR_VARS)
LOCAL_MODULE := libGet_Point-jni
LOCAL_MODULE_TAGS := optional
LOCAL_SRC_FILES := libs/$(LOCAL_MODULE).so
LOCAL_MODULE_STEM := $(LOCAL_MODULE)
LOCAL_MODULE_SUFFIX := $(suffix $(LOCAL_SRC_FILES))
LOCAL_SHARED_LIBRARIES := liblog libxt_get_point
LOCAL_MODULE_CLASS := SHARED_LIBRARIES
LOCAL_MULTILIB := 64
include $(BUILD_PREBUILT)
```
不要忘了app 依赖的代码

```makefile
# jni so
LOCAL_JNI_SHARED_LIBRARIES := libGet_Point-jni
```
so 文件将会被编译预置到/system/lib64/ 目录下，将apk push 到/system/app/AarTest/ 目录下验证可以运行。

注意，so 放到 /vendor/lib64/ 下面访问不到（估计需要app 编译声明为VENDOR app），如果app 是install 安装的，那也没有权限访问/system/lib64/ 下面的so

## 方案二 将aar 中的so 打包进apk？(未成功)
按照网上文章说的，添加 LOCAL\_JNI\_SHARED\_LIBRARIES := libGet\_Point-jni 后so 会自动打包进so，然而实测并没有；

预置so 文件的make 代码写法出了上面列出的一种以外，还搜到一个

```makefile
LOCAL_PREBUILT_LIBS := libGet_Point-jni:libs/libGet_Point-jni.so
```
但是假如这个so 还依赖其他so 的话就会编译不过（回头看上面第一种，LOCAL_SHARED_LIBRARIES := liblog 还依赖了liblog）。同样的，假如上面的代码不会导致编译错误，也还需要

```makefile
# jni so
LOCAL_JNI_SHARED_LIBRARIES := libGet_Point-jni
```
不写这一行也能push apk 成功运行，但是编译出来apk 目录下没有 lib 软链
添加了 LOCAL\_JNI\_SHARED\_LIBRARIES 之后编译出出的apk 目录下有lib 软链，apk 本身是不含so 文件的

**so 打包进apk 失败**
如有大佬打包so 到apk 成功，恳请赐教！ 
# 参考链接
1. [Android 程序开发 总结一： 源码编译引用包含so文件的AAR文件\_alexchen840131的博客-CSDN博客](https://blog.csdn.net/alexchen840131/article/details/84928154)
2. [【安卓开发系列 -- 系统开发】AOSP 源码环境开发系统 APP 预置带有 SO 的 AAR 包\_奋斗企鹅CopperSun的博客-CSDN博客\_libpl\_droidsonroids\_gif.so](https://blog.csdn.net/qq_27788177/article/details/103719570)
3. [Android源码 app预制带so和assets文件的aar包(sdk)\_wd623894174的博客-CSDN博客](https://blog.csdn.net/wd229047557/article/details/96141623)
4. [Android打包so文件到apk\_IT先森的博客-CSDN博客](https://blog.csdn.net/tkwxty/article/details/17039145)
5. [在Android源码中如何吧so库打包编译进入apk， 集成第三方库（jar和so库）\_WangMark的博客-CSDN博客](https://blog.csdn.net/petib_wangwei/article/details/72844515)
6. [Android.bp 文件中引入aar、jar、so库正确编译方法(值得收藏)\_cczhengv的博客-CSDN博客\_android.bp 编译jar](https://blog.csdn.net/u012932409/article/details/108119443)