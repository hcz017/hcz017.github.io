
---
date: 2020-04-02 21:53
status: public
title: 'Cuttlefish 虚拟 Android 设备'
---

# Cuttlefish 虚拟 Android 设备

[https://source.android.google.cn/setup/create/cuttlefish?hl=zh-cn](https://source.android.google.cn/setup/create/cuttlefish?hl=zh-cn)

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/cutfish_what.png)

有区别于Android 模拟器
图片中链接指向

[https://android.googlesource.com/device/google/cuttlefish/](https://android.googlesource.com/device/google/cuttlefish/)
该页面内配置环境的步骤：

## 单独配置cuttlefish

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/cutfish_config.png)
配置过程中出现问题，abort。目前为止并没有说配置和使用过程中要使用到网络上的信息。
然后页面内的github 链接指向：

[https://github.com/google/android-cuttlefish](https://github.com/google/android-cuttlefish)

## 源码下的脚本配置cuttlefish
![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/cutfish_under_source.png)

该命令又指向源码中的执行文件，不过路径需要更新下，device/google/**cuttlefish**/tools/create_base_image.sh
执行过程中提示要配置**Google Cloud Platform** (gcloud)相关信息，account 和project id 等，后续步骤中要绑定 信用卡，abort。

# Android 开发者 Codelab

我是在这里看到 Codelab 的
https://source.android.google.cn/setup/start?hl=zh-cn#create_acloud_instance

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/cutfish_Codlab.png)
这里写到Acloud 用于协助用户创建虚拟Android 设备。
不过Acloud 还是依赖于源码下能先编译出系统镜像。否则会报错
acloud.errors.GetLocalImageError: No image found(Did you choose a lunch target and run `m`?): /mnt/android_source/aosp/out/target/product/vsoc_x86_64.

# 基于目前情报的总结 

1. cuttlefish 是一个android 虚拟设备，有别于android 模拟器（ designed to run on Google Compute Engine）。可以用安装包单独安装，也可以在源码目录下build 安装（需要配置gcloud 相关信息）。
2. Acloud 用于协助用户创建虚拟Android 设备，但是前提需要先编译成功必须的系统镜像。
