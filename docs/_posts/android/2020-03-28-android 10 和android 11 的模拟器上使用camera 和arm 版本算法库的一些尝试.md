---
date:  2020-03-28 10:40
title:  'android 10 和android 11 的模拟器上使用camera 和arm 版本算法库的一些尝试'
---


# 一，模拟器上使用 camera

## 0. 模拟器上 front camra id 是0，back camera id 是1
通常真机上后置主摄camera id 是0，前摄camera id 是1，如果后摄有ultrawide，tele 的话，他们的 id 为2，3。但这是物理camera id（physical camera id），还有个逻辑camera id（logical camera id）概念，逻辑camera id 用一个camera id 代表真实的多个camera 组合。  
比如相机app 打开camera id 5，实际上对应的是ultrawide+wide+tele 的组合，多个摄像头组成一套可以切换的变焦系统，又如打开camera id 4，实际打开的是 wide + tele 的组合，这是双摄组成的人像模式。

## 1. android 10 和android 11 的模拟器都可以使用物理摄像头。
左为android 10 右为android 11。右上角的预览是申请的YUV_420_888 格式数据转成NV21 后canvas.drawBitmap 画上去的。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/simulator_camera_app.png)

物理摄像头的support HW level为 LEVEL_LIMITED，但是不影响app 申请并获得YUV_420_888 格式的数据（YUV_420_888toNV21 转换方式和qcom 不同，所以看到上图颜色有问题）。

## 2. android 11 上back camera 选Emulated， 支持HW  Level 3。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/simulator_camera_log.png)

## 3.使用VirtualScene 模式可以自己添加图片到虚拟场景里

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/simulator_camera_virtual_scene.png)

## 4. 三方app 使用物理摄像头时无法拍照，系统相机可以拍照，但是三方app 改成和系统相机一样的尺寸也无法拍照（原因待查找）。

# 二，android 11 对arm 应用的支持情况

## 1. 标定apk 可以在android 11 模拟器上运行，在android 10 模拟器上无法安装。

app run 之前在10 和11 上都是显示(Device support x86_64,x86, but APK only supports armabi-v7a,arm64-v8a)，但是android 11 版本 run 之后就提示就消失了，如下图。

![image.png](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/simulator_as_screenshot.png)

## 2. 暂未找到root 方法，无法push 离线测试程序测试
