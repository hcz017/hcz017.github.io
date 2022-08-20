---
date:  2018-09-19 20:14
status: public
---
Windows 中VMWare 安装的虚拟机Ubuntu 里面使用Samba 共享文件夹十分方便，基本上就是在文件夹上右键选择共享就可以了（可能会提示安装软件）。而换到Manjaro 后右键属性中并没有此选项，本文记录一下如何在Manjaro 下配置Samba 共享文件夹给windows 系统。

主要内容参考自[Using Samba in your File Manager](https://wiki.manjaro.org/index.php?title=Using_Samba_in_your_File_Manager#KDE).

# 安装软件

```shell
sudo pacman -S samba gvfs-smb thunar-shares-plugin
```

`thunar-shares-plugin` 可以`thunar-shares-plugin-manjaro`代替，我这边是因为换了软件源，前一个软件找不到才换的。

# Manjaro 配置

安装 `manjaro-settings-samba`，安装这个软件后会自动做一些配置：

```shell
sudo pacman -S manjaro-settings-samba
```


重点来了，怎么编写/etc/samba/smb.conf 配置文件。下面是一个例子，前面的部分都是自动生成的，最后一块是新增的。
```shell
[global]
   workgroup = WORKGROUP
   dns proxy = no
   log file = /var/log/samba/%m.log
   max log size = 1000
   client max protocol = NT1
   server role = standalone server
   passdb backend = tdbsam
   obey pam restrictions = yes
   unix password sync = yes
   passwd program = /usr/bin/passwd %u
   passwd chat = *New*UNIX*password* %n\n *ReType*new*UNIX*password* %n\n *passwd:*all*authentication*tokens*updated*successfully*
   pam password change = yes
   map to guest = Bad Password
   usershare allow guests = yes
   name resolve order = lmhosts bcast host wins
   security = user
   guest account = nobody
   usershare path = /var/lib/samba/usershare
   usershare max shares = 100
   usershare owner only = yes
   force create mode = 0070
   force directory mode = 0070

[homes]
   comment = Home Directories
   browseable = no
   read only = yes
   create mask = 0700
   directory mask = 0700
   valid users = %S

[printers]
   comment = All Printers
   browseable = no
   path = /var/spool/samba
   printable = yes
   guest ok = no
   read only = yes
   create mask = 0700

[print$]
   comment = Printer Drivers
   path = /var/lib/samba/printers
   browseable = yes
   read only = yes
   guest ok = no
[m_sharee]
   comment = MShare Directories
   path = /home/{username}/m_share
   browseable = yes
   read only = no
   create mask = 0700
   directory mask = 0700  

```

read only 属性设置为no 是为了有写入文件权限

添加分享用户并设置密码

```shell
gpasswd sambashare -a username
smbpasswd -a username
```

启用smaba 服务

```SHELL
systemctl enable smb nmb
systemctl start smb nmb
```

之后再做修改后你可能需要Log out 再Log in

# Windows 设置

打开资源管理器，计算机，选择“映射网络驱动器”，输入在Manjaro 的ip 地址（ip addr show），形如“\\\\192.168.137.12*”

或者你可以参考这个人的配置[Samba in KDE, how to share from Linux to Windows](https://forum.manjaro.org/t/samba-in-kde-how-to-share-from-linux-to-windows/40941/52)

# 参考链接

1. [Using Samba in your File Manager](https://wiki.manjaro.org/index.php?title=Using_Samba_in_your_File_Manager#KDE)
2. [smb.conf — The configuration file for the Samba suite](https://www.samba.org/samba/docs/current/man-html/smb.conf.5.html)