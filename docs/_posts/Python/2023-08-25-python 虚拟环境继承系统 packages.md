---
Date: 2023-08-25
tags:
  - python
  - venv
title: python 虚拟环境继承系统 packages
---
# Pycharm 新建项目时勾选inherit global site-packages 避免重复安装 packages，节省磁盘体积

使用 Pycharm 创建一个新的工程时默认会创建一个虚拟环境。虚拟环境的好处我们这里就不再多说了。
在创建了虚拟环境之后，我们安装的 packages 都会存放在 venv/Lib/site-packages/ 目录下，如果你使用了较多或者较大的packages 的话，这个venv 目录将会占用不少空间。
比如我之前有个项目的 requirements.txt 文件里的内容如下
```
opencv-python==4.8.0.76
opencv-contrib-python==4.8.0.76
opencv-contrib-python-headless==4.8.0.76
numpy~=1.21.3
scipy==1.7.3
```
合计占用了 406M 空间。试想一下，如果你有多个项目的话，累计占用的空间是以G 计的。
其实在通过Pycharm 创建虚拟环境时多勾选一个 **inherit global site-packages** 选项就可以帮我们节省空间，这个选项的意义是让我们当前的项目可以使用系统环境下安装的packages。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/python/_image/pycharm_venv_config.jpg)
在重新创建了勾选给 的venv 后，虚拟环境占用14M，大大减小体积，当然前提是你系统库里面有安装当前项目所需要的packages。
# 使用命令创建虚拟环境并给予系统packages 使用权限

只需在创建虚拟环境时加上 `--system-site-packages` 选项：
```shell
$  python -m venv --system-site-packages venv3
```
另外虚拟环境目录下会有个 pyvenv.cfg 文件，其中 `include-system-site-packages = true` 就代表是否可以使用系统packages，也就是和 `--system-site-packages` 对应的配置。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/python/_image/pycharm_venv_pyvenv_cfg.jpg)
注意：不建议将所有的packages 都安装在系统环境下，这样虚拟环境就失去了意义。