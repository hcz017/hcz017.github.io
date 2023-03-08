---
date: 2017-01-11 11:30
status: public
title: 本地git 仓库关联github仓库后无法push
tags: git
---

在Github上新建一个仓库后有以下指导
>**…or create a new repository on the command line**
>echo "# learn_git" >> README.md
>git init
>git add README.md
>git commit -m "first commit"
>git remote add origin https://github.com/hcz017/learn_git.git
>git push -u origin master
>**…or push an existing repository from the command line**
>git remote add origin https://github.com/hcz017/learn_git.git
>git push -u origin master

现在的情况是本地有一个已经存在的仓库，但是push不到github上。

1.先删掉以前关联的的远程仓库

```shell
$ git remote remove origin
```

2.添加新的远程仓库地址

```shell
$ git remote add origin https://github.com/hcz017/learn_git.git
```

3.push本地代码到远程

```shell
$ git push -u origin master
```

这时候就报错了

```shell
$ git push -u origin master 
error: src refspec master does not match any
```

试了好几次都这样，google也没查到解法，一般别人是因为本地没有代码才会有这个提示，但我现在本地有代码啊。

忽然意识到，可能是因为**我没有master分支？**（因为我本来就没有master分支，我一开始checkout出来的就是dev分支）

如果说指定了master分支的话，那我确实也可以算本地没有代码。检查一下我本地还真没有master分支。

本地新建一个master分支之后在push就成功了。

```shell
$ git checkout -b master
$ git push -u origin master
```

----

其实这是一个很低级的错误。就是对命令不熟悉，只知道照抄，加上我一开始没注意到本地分支名不是master。

可以看下面的解释

```shell
$ git push origin master
```

上面命令表示，将本地的master分支推送到origin主机的master分支。如果后者不存在，则会被新建。

更多git内容可参考[Git远程操作详解](http://www.ruanyifeng.com/blog/2014/06/git_remote.html)
