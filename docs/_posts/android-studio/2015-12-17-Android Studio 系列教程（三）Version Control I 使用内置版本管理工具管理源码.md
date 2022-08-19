---
date: 2015-12-17 15:33
status: public
tags: 'android-studio'
---

开发android系统源码的同学都知道，我们的工作都是很多人协同工作，因此git版本管理及历史修改查阅异常重要！甚至比开发app重要的多！
此文旨在介绍一下用AS中自带的Version Control工具来管理android系统源代码，鉴于android源码的体积庞大与开源特性，其中一个app的git仓库里就可能包含上千条修改，而且很多历史修改年代久远，且不是身边同事的修改，使用git相关命令查看修改记录效率低下且界面不够友好。而AS中的版本管理工具则相当之优秀，不仅功能强大而且多数功能都做出了可视化的效果，使用之后大大提升了协同开发的效率。希望各位读者看了此文以后能有所收获。
正文开始：
 
这次讲一下在Android Studio中进行适当的配置，使用内置Version Control工具对源码目录进行版本管理。
 
# 一、 配置仓库目录
点开“设置”切换到 Version Control，点击右侧“+”号添加要管理的目录，（我们这里一Telephony目录为例）
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_settings_version_controll.png)
 
选择目录

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_chose_path.png)
 
如果选择的目录是有版本控制工具的配置的，会自动识别出是git或者SVN等。
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_chose_git_repo.png)
 
# 二、 Version Control功能
 
## Log
之后在程序的底部会出现一个Version Control 标签，这里是显示版本信息的主要区域。
当前标签会显示所有添加到Version Control管理的目录的Log信息。各种关键信息以不同颜色高亮区分。
选中其中一条提交后，在右侧出现当次提交修改的文件，双击右侧文件可打开对比窗口显示出更改的内容（下面有展示）。在Log主窗口下面是Commit Message窗口，这个窗口显示当次提交的commit信息。
 1）修改过的文件颜色会变蓝（图中红色数字“1”）
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_git_log.png)
PS：当前在Version Control的Log标签下，因为当时光标焦点在编辑区，所以本该是白色高亮的“Log”在图中看起来是灰色的。
 
2）显示出Log详细信息，需要注意的是如果你用Version Control管理了多个目录，那么这些目录下面的log信息是全部在一起显示的，这样看起来可能比较乱，这时候你可以像下图一样选择一下路径，就可以只看一个目录（仓库）下的log信息。
 
在Path左边还有其他功能，分支，提交作者，日期，请自行体验。
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_filter.png)
 
3）选中一条修改记录后，右侧会显示出对应修改过的文件，双击文件可打开该次提交修改了什么内容。如下图：
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_diff.jpg)
 
## Local Change
 
在local Change标签中查看本次修改，点击下图红色旁边的按钮，可以打开右侧预览窗口。PS：预览窗口中是可以直接编辑的。
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_local_change.png)
 
## Show History
 
这个功能可看单个文件的修改记录，点击下图中“1”旁边的Show History可打开对应标签。
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_history.png)
 
从图中我们可以看到单个文件修改记录的详细信息，同时在双击某条修改记录的时候可以打开窗口对比选中版本对比上一个版本修改了哪些内容。
 
在一条记录上右键选择 Select in Git Log 则会跳转到Log标签下，同时右边可以看到当此修改还修改了其他哪些文件。
 
由于很多源码是从Cyanogenmod上拉下来的，这是你在Commit Message里能过获得Commit id的话，甚至可以到review.Cyanogenmod.org上搜素那次修改，查看当时review代码情况时候的讨论记录！
 
图中红色数字“2”下面对应的很多按钮，功能包括 1.文件对比 2.与本地文件对比 3.创建Patch等。
 
## 不同分支比较

其实这不是一个比较大的功能，但是对于多分支的系统源码级开发来说确实相当实用。
在AS右下角会显示当前工作所在分支：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_current_git_branch.png)

以上图红色圈内为入口点击出现下图：
红色数字1是我们要比较的仓库（对钩表示当前查看的文件是属于这个仓库下面的）；数字2是我们输入的过滤关键字；3是所要比较的目标分支；4则是“比较”选项。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_branch_compare.png)

比较结果：
我们这个结果比较巧合哈，其中一个分支的提交全都合并到了另一个分支，不过从图中下半部分显示出了两个分支log的不同之处，而且选中其中一条log右侧会显示出修改的文件，双击则可以查看修改内容。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_branch_compare_Log.png)

Diff标签下则是直接显示两个分支文件的不同之处，同样的还是双击文件可查看修改内容。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_branch_compare_Diff.png)

当不同分支编译出的结果表现不一样的时候，这个功能可以快速定位到可以的代码块。

# 三、上传代码
修改完成后，点击图中红色数字“1”旁边的Commit Changes按钮，弹出Commit Changes窗口，填写相应信息后，点击右下commit按钮可以提交，这个过程中AS会对代码进行分析并给出分析结果（默认的，可取消），提醒你review代码或者直接提交（因为源代码中引用的一些资源找不到，所以分析结果中一定是有错误的，不用太在意，但是写app的时候如果分析结果又错，那就一定要review一下了）。
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_vertion_upload.png)
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android-studio/_image/AS3_code_analysis.png)
 
更多功能不再详述，请自行体验。
 
PS:大家有新发现的话，欢迎补充上来啊~