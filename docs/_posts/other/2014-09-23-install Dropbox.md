---
Title: 头疼的Dropbox Ubuntu 14.04
Date: September 23 2014 6:20 PM
Tags: dropbox
---
在Ubuntu下安装的dropbox，简直是。。。。
装了好几天了,各种尝试就是装不上。

---
~~
1. 一开始在在Ubuntu软件中心安装的。安装之后点击Dropbox图标，弹出一个授权窗口，授权之后变没有任何反应了。卸载后重试效果一样的，还是不能用；
2. 百度了若干方法尝试后依然无果；
3. 到Dropbox官网上下载deb安装包。感谢大天朝的GFW此路走不通了，虽然通过修改hosts可以访问Dropbox官网了，但是下载页面无法下载安装包。WTF！
4. 机智如我一下子想起来用手机访问网站下载，手机上有fqrouter的嘛。将手机上下载的deb安装包拷到电脑上，双击安装，确实安装了，而且跟软件中心安装的不是一个版本，但是还是TMD没法使用啊；
5. 好吧，咱也试一次从源码变异安装。还是在手机上下载的源码，在官方的说明下解压后编译。
	- 问题来了**“No package 'libnautilus-extension' found”**，我觉得更嘲讽的是，这条结果搜不到与ubunu相关的结果，一条匹配度较高的语句是在[Fedora问题总结](http://http://www.2cto.com/os/201308/235179.html)找到的。但是系统不一样解决方法不适用啊摔！按照ubuntu命令行装软件的习惯，尝试了几条命令后，用“sudo apt-get install libnautilus*”（不要问我为什么用*,因为我用其他的都提示找不到。。。T_T）装了一堆乱七八糟的东西，这条错误提示没了。但是。。。
	- ** “configure: error: couldn’t find docutils”**好吧，问题来了就想办法解决，接着安装“sudo apt-get install python-docutils”，恩，这条错误也被消灭了。然后按照惯例，你知道什么是惯例么？惯例就是消除一个问题，再出现新的问题！
	-  **“/usr/bin/install: 无法创建普通文件"/usr/share/icons/hicolor/16x16/apps/dropbox.png": 权限不够”** sudo也权限不够？

等等，问题好像不在这。。。
~~

- - -

###推倒重来。

以下是官方的安装说明：

    通过命令行安装 Dropbox
    Dropbox 守护程序可在所有 32 位与 64 位 Linux 服务器上正常运行。若要安装，请在 Linux 终端运行下列命令。

    32-bit:

    cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86" | tar xzf -
    64-bit:

    cd ~ && wget -O - "https://www.dropbox.com/download?plat=lnx.x86_64" | tar xzf -
    接着，从新建的 .dropbox-dist 文件夹运行 Dropbox 守护程序。

    ~/.dropbox-dist/dropboxd

wget是为了得到 “dropbox-lnx.x86-2.10.30.tar.gz”文件，既然电脑访问不到，那就用手机吧。
下载后解压到用户文件夹下， "~/.dropbox-dist/dropboxd"运行dropbox。

有若干行提示：
```
Gtk-Message: Failed to load module "overlay-scrollbar"
Gtk-Message: Failed to load module "unity-gtk-module"

(dropbox:9746): Gtk-WARNING **: 无法在模块路径中找到主题引擎：“murrine”，
.......................
Gtk-Message: Failed to load module "canberra-gtk-module"
```
百度得到的一些处理方法：
```
前两条没找到
sudo apt-get install gtk2-engines-murrine
sudo apt-get install libcanberra-gtk3-module
```
还在启动中，然后dropbox的界面出来了，接下来是一系列设置与教程。

**目前的情况是，Dropbox可以同步文件了，但是系统托盘不显示Dropbox的程序图标。**

Q：Dropbox托盘图标不显示
A：sudo apt-get install python-appindicator
上面的命令未能解决

###参考：
- [Centos安装nautilus-Dropbox出错的解决方法](http://http://www.myzhenai.com.cn/post/798.html/comment-page-1)
- [Fedora问题总结](http://http://www.2cto.com/os/201308/235179.html)
- [Ubuntu上安装dropbox](http://www.educity.cn/wenda/565884.html)

其他
Q:
E: 无法获得锁 /var/lib/dpkg/lock - open (11: 资源暂时不可用)
E: 无法锁定管理目录(/var/lib/dpkg/)，是否有其他进程正占用它？”

A:
方法1 终端输入 ps  -aux ，列出进程。找到含有apt-get的进程，直接sudo kill PID。解决。
方法2 强制解锁,命令
sudo rm /var/cache/apt/archives/lock
sudo rm /var/lib/dpkg/lock