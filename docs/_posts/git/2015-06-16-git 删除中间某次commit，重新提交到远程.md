---
date:  2015-06-16 10:42
status: public
title: 'git 删除中间某次commit，重新提交到远程'
tags: git
---

**问题现象：**
在修改android源码的时候，我在本地同一分支上做了两次修改并提交到远程，分别取名A和B吧。然后在Gerrit上提交A review不通过，提交B review通过，虽然B同意入库，但是却依赖于A，因此也无法入库。偏偏我本地已经删除了当时做了修改的分支（repo abandon）。那么我该如何在不重新修改代码的情况下，然B可以入库呢？
 
**期望结果：**
在不改动代码的情况下，使提交B不在依赖于A并成功入库。
 
**步骤：**
（为了安全我们尽量避免在主分支上操作）
1. 找回已删除的提交
1）新建恢复分支
    repo start backup .
2)找回之前的提交
    git reflog 查看操作历史，找到之前提交B的 hash 值，然后 git reset --hard 到那个 hash 即可。
    使用git log查看，我之前的两次修改都找回了。
 
2. 删除提交A，只保留提交B
1）新建要上传的分支
    repo start upload .（如果没用repo工具管理，就用git branch 新建分支吧）
2）恢复提交B
    git cherry-pick （B的hash值）
3）git log查看效果
 
3. 如果没问题的话，就可以git commit --amend之后再 repo upload . 提交了
 
按照上面的步骤操作如果不能得到预期效果的话可以试试用别的命令。
中心思想为：
1. 找回已经删除的提交信息
2. 新建一个分支并只恢复提交B
4. 重新上传
 
**注：**因为我本地代码是repo管理，读者可根据自身情况使用对应的git命令。

参考链接：
1. [从Git仓库中恢复已删除的分支、文件或丢失的commit](http://www.linuxidc.com/Linux/2014-09/107293.htm)
2. [git 删除某次指定的提交](http://segmentfault.com/a/1190000002422158)
