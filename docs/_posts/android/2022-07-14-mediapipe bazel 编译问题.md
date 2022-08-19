---
date:  2022-07-14 19:27
title:  'mediapipe bazel 编译问题'
---

因为某些原因要**回退**mediapipe 版本，但是bazel 编译依赖很多仓库的代码已经更新了，所以编译时出现了一些仓库或者变量找不到的错误提示。下面整理一些解决方法。

# 分支更名

完整错误如下:

```bash
ERROR: An error occurred during the fetch of repository 'rules_foreign_cc':
   java.io.IOException: Error extracting /mnt/android_source/bazel_cache/_bazel_edison/bd6da075c4965688e2daae750238d36b/external/rules_foreign_cc/master.zip to /mnt/android_source/bazel_cache/_bazel_edison/bd6da075c4965688e2daae750238d36b/external/rules_foreign_cc: Prefix "rules_foreign_cc-master" was given, but not found in the archive. Here are possible prefixes for this archive: "rules_foreign_cc-main".
```

其实错误提示也比较明显了“ Prefix "rules_foreign_cc-master" was given, but not found in the archive. Here are possible prefixes for this archive: "rules_foreign_cc-main".”

这是由于zzzq 原因一些仓库的“master” 分支更名为了“**main**” 分支。

我们在 WORKSPACE 文件中将 `strip_prefix = "rules_foreign_cc-master"`修改为`strip_prefix = "rules_foreign_cc-main"`就可以解决此类问题。

对应的url 不需要修改。

# 使用离线的依赖仓库

重新执行编译命令，有新的错误：

```bash
Repository rule http_archive defined at:
 /mnt/android_source/bazel_cache/_bazel_edison/bd6da075c4965688e2daae750238d36b/external/bazel_tools/tools/build_defs/repo/http.bzl:336:31: in <toplevel>
ERROR: cannot load '@rules_foreign_cc//:workspace_definitions.bzl': no such file
```

翻译一下就是 rules_foreign_cc 这个仓库里面 workspace_definitions.bzl 文件不存在。

打开Github上的[rules_foreign_cc](https://github.com/bazelbuild/rules_foreign_cc) 仓库，果然这个文件不存在，但历史上肯定是有的，不然当时不会这么写这段代码。通过将代码切换到tag [0.2.0](https://github.com/bazelbuild/rules_foreign_cc/tree/0.2.0) 之后，这个文件出现了，说明我们应该使用这个tag 对应的版本。我们可以选择下载当前节点的仓库，下载一个zip 包存到本机目录，然后修改WORKSPACE 文件如下（git diff 展示）：

```git
 http_archive(
    name = "rules_foreign_cc",
-   strip_prefix = "rules_foreign_cc-master",
-   url = "https://github.com/bazelbuild/rules_foreign_cc/archive/master.zip",
+   strip_prefix = "rules_foreign_cc-0.2.0",
+   urls = ["file:////home/edison/Downloads/rules_foreign_cc-0.2.0.zip"],
 )
```

即：将网络上的地址修改为本地地址。注意 strip_prefix 那里也要跟上tag 名，其实就是文件名不含后缀。

这种方法比较适用于编译环境网络不通，或者不想频繁访问网络的条件下。

# 拉取依赖仓库指定commit 节点的代码

重新执行编译新的错误又来了，不慌，看看是什么。

```bash
Repository rule http_archive defined at:
  /mnt/android_source/bazel_cache/_bazel_edison/bd6da075c4965688e2daae750238d36b/external/bazel_tools/tools/build_defs/repo/http.bzl:336:31: in <toplevel>
ERROR: /mnt/android_source/bazel_cache/_bazel_edison/bd6da075c4965688e2daae750238d36b/external/rules_cc/cc/private/rules_impl/cc_flags_supplier.bzl:16:1: file '@bazel_tools//tools/cpp:toolchain_utils.bzl' does not contain symbol 'use_cpp_toolchain' (did you mean 'find_cpp_toolchain'?)
```

看样子是文件存在，但是文件内容被修改了，不适当前版本编译了。继续使用回退大法。

只不过这次我们不下载仓库了，毕竟步骤还是稍微繁琐了一些，我们可以修改代码取拉去对应仓库指定commit 节点的代码！

去GitHub 上的 [rules_cc](https://github.com/bazelbuild/rules_cc) 仓库找到对应文件，查看文件修改历史，发现报错提示“use_cpp_toolchain” 是最近修改的，那我们可以指定它的上一个节点编译试试。

修改WORKSPACE 文件如下（git diff 展示）：：

```git
+_RULES_CC_GIT_COMMIT = "f95239adde29680236afa22b4abaf1d04234f61a"
 http_archive(
     name = "rules_cc",
-    strip_prefix = "rules_cc-master",
-    urls = ["https://github.com/bazelbuild/rules_cc/archive/master.zip"],
+    strip_prefix = "rules_cc-%s" % _RULES_CC_GIT_COMMIT,
+    urls = ["https://github.com/bazelbuild/rules_cc/archive/%s.tar.gz" % _RULES_CC_GIT_COMMIT],
 )
```

重新编译，成功！