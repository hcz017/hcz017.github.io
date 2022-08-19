---
date:  2020-03-29 12:43
title:  'android 11 模拟器root 的一些尝试'
---

20200329
# 目的
往模拟器（最好是arm 模拟器）system 和vendor 分区push 文件。通常系统root 后才可以push。
# 现有资源
网上的方法基本上都是基于下面github 上的方法，大同小异。
[https://github.com/0xFireball/root_avd](https://github.com/0xFireball/root_avd)
# 已经做过的尝试
不过上面github 上的方法是在 Android 7.1 Nougat 上测试的，在android 11 上行不通。

按照上面github 上方法的步骤，遇到的主要问题是，我们需要push 一个su 文件到系统分区，但是始终拿不到system 分区的写权限。
即使使用命令行添加-writable-system 提示system image 可写，但实际上还是不可写。
https://developer.android.com/studio/run/emulator-commandline#deprecated
![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android-emulator-commandline.png)
```SHELL
PS D:\Android\Sdk\emulator> .\emulator.exe -avd Pixel_3_API_R -no-boot-anim -writable-system -selinux permissive -qemu
emulator: WARNING: System image is writable
Failed to open /qemu.conf, err: 2
Windows Hypervisor Platform accelerator is operational
```
实际测试开机后不可写，而且执行adb remount 后提示重启手机，但是一旦执行重启就会遇到手机打不开的情况（始终处于offline 状态，且模拟器上无内容）
```SHELL
PS D:\megvii\root_avd\SuperSU> adb root
restarting adbd as root
PS D:\megvii\root_avd\SuperSU> adb remount
Disabling verity for /system
Using overlayfs for /system
Using overlayfs for /vendor
Using overlayfs for /product
Using overlayfs for /system_ext
Now reboot your device for settings to take effect
remount succeeded
```
如果尝试单独挂载system 分区，则提示system 分区不存在
```SHELL
PS D:\megvii\root_avd\SuperSU> adb shell mount -o rw,remount /system
mount: '/system' not in /proc/mounts
```
ls 可以看到根目录system 文件夹，但是df 看不到
![在这里插入图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/android-shell-system.png)
system 分区没有挂载为可读，无法push 文件
```SHELL
PS D:\megvii\root_avd\SuperSU> adb push .\x64\su /system/xbin/su
adb: error: failed to copy '.\x64\su' to '/system/xbin/su': remote couldn't create file: Read-only file system
.\x64\su: 0 files pushed, 0 skipped. 22.4 MB/s (100520 bytes in 0.004s)
PS D:\megvii\root_avd\SuperSU>
```
vendor 分区也不能push 文件。
# 结论
暂时没有找到往模拟器vendor 和system 分区push 文件的可行方法。

**参考连接**
https://android.stackexchange.com/questions/171442/root-android-virtual-device-with-android-7-1-1
https://android.stackexchange.com/questions/110927/how-to-mount-system-rewritable-or-read-only-rw-ro/207200#207200
https://developer.android.com/studio/run/emulator-commandline#deprecated
https://stackoverflow.com/questions/5095234/how-to-get-root-access-on-android-emulator

https://www.jianshu.com/p/fd39ec466e88
https://developer.android.google.cn/studio/run/emulator-comparison?hl=zh-cn
https://developer.android.google.cn/studio/run/emulator?hl=zh-cn#screen-recording