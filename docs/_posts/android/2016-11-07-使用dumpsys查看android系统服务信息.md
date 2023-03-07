---
date:   2016-11-07 17:30
title:  '使用dumpsys查看android系统服务信息'
---


# 1. 什么是dumpsys 
The dumpsys tool runs on the device and provides information about the status of system services.

dumpsys这个工具可以查看当前设备系统服务信息。
 
# 2. 如何使用dumpsys 
如果你直接运行adb shell dumpsys的话，会得到所有系统服务的输出，输出结果比你想要的多得多。为了控制输出内容，需要指定想要查看的服务。如：

` $ adb shell dumpsys input`

# 3. dumpsys能查看到的信息有那些 
用下面的命令可以列出所支持查看的系统服务
```bash
    $ adb shell dumpsys -l
    Currently running services:
    DockObserver
    SurfaceFlinger
    accessibility
    account
    activity
    alarm
    …
```
常用命令举例：
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_16-28-52.jpg)
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_16-28-58.jpg)


不同服务可以跟不同的命令选项，以activity为例：
![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_activety.jpg)

## 获取Activity信息：
`adb shell dumpsys activity`
加上-h可以获取帮助信息
获取当前界面的activity详情，可以用：
`adb shell dumpsys activity top`(控件都看得到)
对了，还有一种情况，就是当前界面是一个activity动态加载的fragment的话，这时候就算知道了activity也对应不上界面的内容，不过这时候还是可以用这个命令，去看输出结果的Fragment关键字，排在最上面的就是当前界面对应的Fragment。
要获取当前界面的Activity（结果显示的是最顶层的activity，但activity可能不是当前界面上的最顶层，自行体会）：
` adb shell dumpsys activity top | findstr ACTIVITY`这个其实就是把上面命令的结果过滤一下只显示第一行
（findstr不可用的话用grep）
最近activity：

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_16-29-53.jpg)

## 获取电源管理信息：
 `adb shell dumpsys power`
 
可以获取到是否处于锁屏状态：mWakefulness=Asleep或者mScreenOn=false  
亮度值：mScreenBrightness=255  
屏幕休眠时间：Screen off timeout: 60000 ms  
屏幕分辨率：mDisplayWidth=1440，mDisplayHeight=2560  
**wake lock**：Wake Locks:size =  

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_16-30-37.jpg)

GsmConnection是一个tag，在new wake_lock的时候自行定义，通过pid可以确定其所在的服务进程。

## 查看手机telephony状态：
可以看网络注册状态，数据链接状态，是否漫游，信号强度，等等，参数我就不一一解毒了，跟android系统版本也有关系
![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_telephpny.png)

## 查看notification：
`adb shell dumpsys notification`

查看是由哪个应用发出的通知。比如你用着手机的时候出来一个神烦的广告，但是你却不知道它是哪个应用弹出的，那去卸载哪个应用呢？

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_notification.gif)

我们截图输出结果的一部分
```xml
NotificationRecord(0x01b93b62: pkg=com.tencent.mobileqq user=UserHandle{0} id=121 tag=null score=0 key=0|com.tencent.mobileqq|121|null|10125: Notification(pri=0 contentView=null vibrate=null sound=null tick defaults=0x0 flags=0x11 color=0x00000000 vis=PRIVATE))
      uid=10125 userId=0
      icon=Icon(typ=RESOURCE pkg=com.tencent.mobileqq id=0x7f0204bf) / com.tencent.mobileqq:drawable/name
      pri=0 score=0
      key=0|com.tencent.mobileqq|121|null|10125
      seen=true
      groupKey=0|com.tencent.mobileqq|121|null|10125
      contentIntent=PendingIntent{bc254f3: PendingIntentRecord{e9265b0 com.tencent.mobileqq startActivity}}
      deleteIntent=PendingIntent{5711c29: PendingIntentRecord{873abae com.tencent.mobileqq broadcastIntent}}
      tickerText=Flour_Mo(MoKee Project):[挖鼻孔]
```
比较关键的是contentIntent和deleteIntent，这两个类型都是PendingIntent。注意这里能看到的是**应用的包名**，并不能看到点击后会跳转到哪个activity（大家代码都会混淆的好嘛，想看也看不到吧）。

## 查看SurfaceFlinger：
`adb shell dumpsys SurfaceFlinger`

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_surfaceflinger.png)

用来查看当前界面上有几个frame，分别的源是什么。手机界面莫名其妙弹出一个dialog？试试这个命令吧，可以看到是由那个进程的那个Activity弹出的。

## 查看wifi信息（连接记录）：
`adb shell dumpsys wifi`

这个厉害了，可以看看他/她有没有连过闺蜜/老王加的wifi！

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_wifi1.png)

最后一个乱码，因为是中文wifi，不过后面也可以看到中文名

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_wifi2.png)

可以看到很多信息的，自己去发掘吧（密码没有，要看密码去看系统文件吧）。

# 4. 使用到dumpsys的案例

## 4.1 接听视频来电屏幕会休眠

一般来说，处于视频电话的时候，屏幕应该保持常亮的，这样才能方便用户查看视频内容。  
问题出现的时候根据以往的经验，知道这很有可能是某个wake_lock没有申请到，比较MO和MT的wake_lock看到：
`adb shell dumpsys power`

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_MOMT_POWER.png)

Mt端少了SCREEN_BRIGHT_WAKE_LOCK，而它在PowerManager.java中的定义正是：

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_powermanager_code.png)

直接搜关键字SCREEN_BRIGHT_WAKE_LOCK没有在InCallUI，Telecomm，Telephony搜到相关的wake_lock类型，但是我们看到这个wake_lock和FLAG_KEEP_SCREEN_ON有联系，多数应用都是用的这个flag，那么我们再搜这个关键字的收，就在InCallUI中找到了出问题的地方，找到问题了就好改啦~

## 4.2 设置双卡询问，弹出的dialog属于Dialer还是InCallUI
双卡手机在设置了拨号前每次询问的时候，会弹出一个dialog（当然也有厂家定制的在拨号盘上之间诶选择），我们的入口是Dialer，而通话界面属于InCallUI，那这个选择SIM卡的dialog属于谁？

用下面的命令可以查看

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_find_parent_activity.png)

如果你觉得这个方法不是那么有必要，那你应该还没有遇见过没title没内容的dialog/斜眼笑  
还有这个方法适合于大多数场景，但也有例外。比如这个顶层界面（不一定是dialog）上的东西不是从Activity中显示出来的，（纳尼哦？！居然有这种事？！）这时候可以尝试用`adb shell dumpsys SurfaceFlinger`，这个命令可以查看界面是由哪些内容绘制的（想知道原理的可以去看SurfaceFlinger机制）。

![这里写图片描述](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/dumpsys_case_surfaceflinger.png)

上图的信息显示，当前界面有状态栏，导航栏，还有一个联想的com.lenovo.ideafriend/com.lenovo.ideafriend.alias.DialtactsActivity，那状态栏和导航栏都很好认，界面上剩下的就是那个DialtactsActivity了。

感谢阅读。