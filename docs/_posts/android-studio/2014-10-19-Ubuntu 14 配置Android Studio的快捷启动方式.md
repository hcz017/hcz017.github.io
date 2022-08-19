---
date: 'October 19 2014 11:09 PM'
status: public
tags: 'android-studio'
title: 'Ubuntu 14 配置Android Studio的快捷启动方式'
---

在window7下安装配置了Android Studio之后就想把ubuntu下的Android Studio整舒服。

Ubuntu下解压Android Studio压缩包后有个名为"Install-Linux-tar.txt"的说明文件,里面有这么一段:

>  1. Unpack the Android Studio distribution archive that you downloaded to
     where you wish to install the program. We will refer to this destination
     location as your {installation home} below.

>  2. Open a console and cd into "{installation home}/bin" and type:

>       ./studio.sh

>     to start the application.

>  3. [OPTIONAL] Add the "{installation home}/bin" to your PATH environmental
     variable so that you may start Android Studio from any directory.
     
如果只做前两步的话每次启动Android Studio都要用终端进入Android Studio的文件夹运行**"./studio.sh"**,这是很麻烦的。
- -  -
第一次：修改**/etc/profile**文件,添加"{installation home}/bin"到环境变量。重启系统使其生效。

事实证明不作死就不会死啊...重启以后被卡在了输密码登陆的界面,无限循环输密码吗进不去系统。

解决办法:1.Ctrl+Alt+F1进入命令界面, 2.输入**sudo vi /etc/profile**还原为修改前的内容, 3.输入**:wq**保存, 4.输入**reboot**重启系统[1]。
- - -
第二次：修改**/etc/environment**添加"{installation home}/bin" 到PATH环境变量，此时可以从任意文件夹启动Android Studio了，但是还是要在终端里面。

顺便profile和environment的区别：系统是先执行/etc/environment，后执行/etc/profile。/etc/environment是设置整个系统的环境，而/etc/profile是设置所有用户的环境。系统应用程序的执行与用户环境可以是无关的，但与系统环境是相关的[2]。
- - -
Google到一篇名为**How to add Android Studio to the launcher?**的文章[3]，里面有一段Answers内容为：

Here is my AndroidStudio .desktop file which works from the launcher.

    [Desktop Entry]
    Version=1.0
    Type=Application
    Name=Android Studio
    Exec="/home/username/Programs/AndroidStudio/bin/studio.sh" %f
    Icon=/home/username/Programs/AndroidStudio/bin/idea.png
    Categories=Development;IDE;
    Terminal=false
    StartupNotify=true
    StartupWMClass=jetbrains-android-studio
    Name[en_GB]=android-studio.desktop
Alternatively, you can also open Android Studio, click on Configure（如果已经打开了AS，此处就改为Tools） -> Create Desktop Entry. This should create an entry on the dash:

![screenshoot](http://i.stack.imgur.com/bueXQ.png)

**AndroidStudio.desktop**文件放桌面上。做了这些之后就可以从桌上和dash里启动Android Studio了。如果提示**未信任的应用启动器的问题**，这时只要右键该应用的desktop文件，单击属性，在权限选项卡中勾选“允许作为程序执行文件”即可[4]。

- - -
我发现这个makedown编辑器略坑爹,不会自动保存也不提示保存,这是我第二遍写这段文字了。


**参考:**
1.  [Ubuntu 14.04解决登录界面无限循环的方法](http://www.linuxidc.com/Linux/2014-05/101749.htm)
2. [linux中/etc/profile 与/etc/environment文件的区别](http://blog.csdn.net/davidsky11/article/details/24272715)
3. [How to add Android Studio to the launcher?](http://askubuntu.com/questions/298857/how-to-add-android-studio-to-the-launcher)
4. [解决ubuntu下提示未信任的应用启动器的问题](http://jingyan.baidu.com/article/6079ad0e62f8fd28ff86dbeb.html)