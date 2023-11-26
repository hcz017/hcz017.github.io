---
date:  2020-07-30 10:40
title:  '在CMakeLists 与Makefile 打印信息'
---


# CMakeLists 打印信息

看下面的例子，我们在cmake定义了一个变量“USER_KEY”，并打印此变量值。status表示这是一般的打印信息，我们还可以设置为“ERROR”，表示这是一种错误打印信息。

```
SET(USER_KEY, "Hello World")
MESSAGE( STATUS "this var key = ${USER_KEY}.")
MESSAGE( FATAL_ERROR"this var key = ${USER_KEY}.")
```

# Makefile & Android.mk文件中打印信息

在makefile中打印输出信息的方法是：
```
$(warning xxxxx)
$(error xxxxx)
```
输出变量方式为：
```
$(warning  $(XXX))
```

