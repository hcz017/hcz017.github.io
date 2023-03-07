---
date: 2016-08-19 10:59
status: public
title: Notification中Extra携带的参数不更新
---

# 问题：
创建一个notification在状态栏显示，这个notification在创建之时放了一个intExtra进去，每次创建的时候这个值会有变化，但是在点击notification后getExtra得出来的值却没有更新。
简单说就是Extra没有更新，百思不得解。
后来发现一个参数`PendingIntent contentIntent = PendingIntent.getActivity(mContext, 0, intent, 0);`，这里面的0是表示什么意思？
然后想到按照google官方[Build Notification](https://developer.android.com/training/notify-user/build-notification.html)写的demo却没有这个问题。于是对比参数发现了PendingIntent.FLAG_UPDATE_CURRENT的存在。
源码解释如下
```java
    /**
     * Flag indicating that if the described PendingIntent already exists,
     * then keep it but replace its extra data with what is in this new
     * Intent. For use with {@link #getActivity}, {@link #getBroadcast}, and
     * {@link #getService}. <p>This can be used if you are creating intents where only the
     * extras change, and don't care that any entities that received your
     * previous PendingIntent will be able to launch it with your new
     * extras even if they are not explicitly given to it.
     */
    public static final int FLAG_UPDATE_CURRENT = 1<<27;
```
修改后一切OK！