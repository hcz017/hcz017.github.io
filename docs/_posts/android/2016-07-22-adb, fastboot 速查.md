---
Date: 2016-07-22
Tags: 命令 adb
---

https://developer.android.com/studio/releases/platform-tools.html
google 把adb tool 从SDK中抽出来了一份成一个小压缩包，windows下不想下大体积SDK的，可以用这个配置adb 环境。

# 常用adb命令

## adb devices 列出连接的设备

```shell
adb devices
List of devices attached
LS5501  device
```

## adb root 获得root权限

```shell
adb root
restarting adbd as root
```

## adb remount

重新挂在手机内存储，通常要先执行adb root 才能成功

```shell
adb remount
remount succeeded
```

## adb shell 进入到android shell模式

可以执行简单的linux命令

```shell
adb shell
root@LS-5501:/ #
```

## adb push <本地文件> <手机目录>

把本地文件push到手机目录

```shell
adb push WCNSS_qcom_cfg.ini /sdcard/
161 KB/s (7768 bytes in 0.046s)
```

## adb pull  <手机文件> <本机目录>

把手机指定目录的文件 pull到电脑本地指定目录

```shell
adb pull /sdcard/PokemonGO.3987.apk F:\ckt\apk
3285 KB/s (60876068 bytes in 18.093s)
```

## adb reboot 重启手机

## 录屏

```shell
adb shell screenrecord /sdcard/demo.mp4
adb shell screenrecord --size 1920x480 /sdcard/demo.mp4
adb shell screenrecord --time-limit 10 /sdcard/demo.mp4
adb shell screenrecord --bit-rate 6000000 /sdcard/demo.mp4
```

## 截图到手机

```shell
adb shell screencap -p /sdcard/screen.png
```

## 截图到电脑

```shell
adb shell screencap -p | sed 's/\r$//' > screen.png
```

## 杀camera 服务进程

```
adb shell "ps -A| grep cam |cut -c 14-20 | xargs kill"
```

## adb shell stop，adb shell start android 层开关机

## adb reboot bootloader 重启到bootloader模式

可以刷boot.img

## adb kill-server 杀掉adb进程

一般在adb命令不起作用，或者adb生成的文件被占用的时候使用

## adb start-server 启动adb进程

常与上一条命令连用

## adb logcat -c 清空缓存的adb log

之后再执行 adb logcat 保存文件的话可以减少log体积

## adb logcat -v time -b main

-v 指格式 -b指输出log类型，main radio system events

## adb shell setprop 设置属性

一般设置logtag，例 adb shell setprop log.tag.InCall V 重启android层生效

```shell
adb shell setprop log.tag.InCall V
```

## 调整动画

```shell
# 调慢 0.75x
adb shell settings put global window_animation_scale 0.75
adb shell settings put global transition_animation_scale 0.75
adb shell settings put global animator_duration_scale 0.75
# 恢复成 1.0x
adb shell settings put global window_animation_scale 1
adb shell settings put global transition_animation_scale 1
adb shell settings put global animator_duration_scale 1
# 调快到 1.25x
adb shell settings put global window_animation_scale 1.25
adb shell settings put global transition_animation_scale 1.25
adb shell settings put global animator_duration_scale 1.25
```

# 常用fastboot命令

1. fastboot flash boot boot.img 刷boot.img 一般用来获得root权限
    先用`adb reboot bootloader`进入bootloader模式，不需要root权限

2. fastboot reboot 刷完boot.img以后重启手机

以下属于扩展，刷不同的镜像文件

- fastboot flash recovery recovery.img 
- fastboot flash userdata userdata.img
- fastboot flash system system.img
