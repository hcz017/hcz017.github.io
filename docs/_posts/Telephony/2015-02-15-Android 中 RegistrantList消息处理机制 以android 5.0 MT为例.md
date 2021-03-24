---
date:  2015-02-15 10:54
status: public
tags: Call
title: 'Android 中 RegistrantList消息处理机制 以android 5.0 MT为例'
---

这其实是观察者模式的一种实现形式  
先明确两个身份 1.**RefistrantList** 通知者 2.**Registrant** 观察者，这是一个一对多的关系，在有事件更新时，凡是通知者列表内的对象，都会收到通知。
RegistrantList 作为通知者支持对Registrant 的**增加**（add/addUnique）**删除**（remove），并且能够发出**通知**（notifyRegitrants），而Registrant作为观察者，**响应**通知者发出的通知，调用internalNotifyRegistrant()把add()时携带的Message发出去处理。
总体思想是：一个对象中开辟一个空间用于存放Message，当调用regist方法时将Message存放进去，当调用notify方法时将所有Message取出并发送到MessageQueue中等待处理。
下面我们以android 5.0上 来电流程为例讲一下RegistrantList机制的使用。
#注册为观察者
1.PstnIncomingCallNotifier这个类中调用mphoneBase中的registerForNewRingingConnection方法注册为观察者，android中的注册为观察者的方法通常写为**registerFor\*\*\*()**形式，即为某事件注册消息通知。
PstnIncomingCallNotifier.java
```java
   	private void registerForNotifications() {
        Phone newPhone = mPhoneProxy.getActivePhone();
        if (newPhone != mPhoneBase) {
            unregisterForNotifications();
            if (newPhone != null) {
                Log.i(this, "Registering: %s", newPhone);
                mPhoneBase = newPhone;
                // 调用registerForNewRingingConnection()方法往RegistrantList中添加Registrant，**注意参数包括handler和message**
                mPhoneBase.registerForNewRingingConnection(
                        mHandler, EVENT_NEW_RINGING_CONNECTION, null);
                mPhoneBase.registerForCallWaiting(
                        mHandler, EVENT_CDMA_CALL_WAITING, null);
                mPhoneBase.registerForUnknownConnection(mHandler, EVENT_UNKNOWN_CONNECTION,
                        null);
            }
        }
    }
```
2.然后我们看registerForNewRingingConnection()内部做了哪些操作：
调用addUnique()方法添加为观察者。
phoneBase.java
```java
    // Inherited documentation suffices.
    @Override
    public void registerForNewRingingConnection(
            Handler h, int what, Object obj) {
        checkCorrectThread(h);
        mNewRingingConnectionRegistrants.addUnique(h, what, obj);
    }
```
而这个mNewRingingConnectionRegistrants是什么呢？
```java
    protected final RegistrantList mNewRingingConnectionRegistrants = new RegistrantList();
```
mNewRingingConnectionRegistrants是一个**RegistrantList** 。

3.添加到通知者列表
用传进来的三个参数新建一个观察者，继续调用RegistrantList内部的add()方法
RefistrantList.java
```java
    public synchronized void
    addUnique(Handler h, int what, Object obj)
    {
        // if the handler is already in the registrant list, remove it
        remove(h);
        // 新建Registrant
       add(new Registrant(h, what, obj));
    }
```
3.1新建一个Registrant观察者
```java
    public
    Registrant(Handler h, int what, Object obj)
    {
        refH = new WeakReference(h);//Handler 泛型WeakReference
        this.what = what;//消息类型
        userObj = obj;//Object数据对象，用于封装传递的数据
    }
```
3.2添加到通知者要通知的列表中，用列表保存观察者。**registrants是一个ArrayList**。
```java
    ArrayList registrants = new ArrayList();      // of Registrant
    public synchronized void
    add(Registrant r)
    {
        removeCleared();
        registrants.add(r);
    }
```
registerForNewRingingConnection()方法完成了往RegistrantList中添加Registrant的操作。
同时我们也看到，**RegistrantList管理了一个Registrants列表，Registrants保存了多个Registrant**。

#发出通知
1.handlePollCalls方法根据RIL发出的Call List对象判断Call的状态，并发出不同的通知，
有新的来电将执行： phone.notifyNewRingingConnection; 形如**notify\*\*\*()**也是惯用写法。
GsmCallTracker.java
```java
    Connection newRinging = null; //or waiting
    handlePollCalls(){
	...
	if (newRinging != null) {
        mPhone.notifyNewRingingConnection(newRinging);
    }
```
2.GSMPhone.java
```java
    public void notifyNewRingingConnection(Connection c) {
        super.notifyNewRingingConnectionP(c);
    }
```
调用父类 PhoneBase.java notifyNewRingingConnectionP()发出来电通知 
```java
    mNewRingingConnectionRegistrants.notifyRegistrants(ar);
```
前面有说过mNewRingingConnectionRegistrants是一个RegistrantList。
```java
    /**
    * Notify registrants of a new ringing Connection.
    * Subclasses of Phone probably want to replace this with a
    * version scoped to their packages
    */
    public void notifyNewRingingConnectionP(Connection cn) {
        if (!mIsVoiceCapable)
            return;
        AsyncResult ar = new AsyncResult(null, cn, null);
        // 调用RegistrantLis 的notifyRegistrants() 方法
        mNewRingingConnectionRegistrants.notifyRegistrants(ar);
    }
```
通知者RegistrantList.java
```java
    public /*synchronized*/ void
    notifyRegistrants(AsyncResult ar)
    {
        internalNotifyRegistrants(ar.result, ar.exception);
    }
```
一般来说观察者不止一个，所以用for循环遍历观察者，调用观察者内部的internalNotifyRegistrant()响应通知
```java
    private synchronized void
    internalNotifyRegistrants (Object result, Throwable exception)
    {
       for (int i = 0, s = registrants.size(); i < s ; i++) {
            Registrant  r = (Registrant) registrants.get(i);
            r.internalNotifyRegistrant(result, exception);
       }
    }
```

#响应通知消息
Registrant.java
这里是处理RegistrantList 的通知，主要是将message发出出去。
这里的handler和message是之前调用RegistrantList 的addUnique() 方法时添加进去的（add(new Registrant(h, what, obj));）。
```java
    /*package*/ void
    internalNotifyRegistrant (Object result, Throwable exception)
    {
       // 得到handler
        Handler h = getHandler();
        if (h == null) {
            clear();
        } else {
            Message msg = Message.obtain();
            msg.what = what;
            msg.obj = new AsyncResult(userObj, result, exception);
            // 发送消息            
            h.sendMessage(msg);
        }
    }
```
PstnIncomingCallNotifier.java
```java
	private final Handler mHandler = new Handler() {
        @Override
        public void handleMessage(Message msg) {
            switch(msg.what) {
                case EVENT_NEW_RINGING_CONNECTION:
                    handleNewRingingConnection((AsyncResult) msg.obj);
                    break;
                case EVENT_CDMA_CALL_WAITING:
                    handleCdmaCallWaiting((AsyncResult) msg.obj);
                    break;
                case EVENT_UNKNOWN_CONNECTION:
                    handleNewUnknownConnection((AsyncResult) msg.obj);
                    break;
                default:
                    break;
            }
        }
    };
```
至于响应通知后续做了什么工作不是这次的重点。

=====补充======
```java
Message msg = Message.obtain():
Handler h = getHandler();
h.sendMessage(msg);
```
从obtain()的源代码中我们可以知道,它是静态方法,而且只有在spool = null 的情况下才会new出一个Message(),返回一个Message对象,如果在不为空的情况下,Message的对象都是从Message对象池里面拿的实例从而重复使用的,这也为了Android中的Message对象能够更好的回收。

使用Handler中的sendMessage (Message msg)方式来发送消息.

我们可以知道android 中发送消息不管是Message中的几种重载的obtain()方式，还是Handler中的几种重载的sendMessage最终都是通过Handler.sendMessage来发送的,而Handler中的几种sendMessage()重载方法最终都会调用到sendMessageAtTime()方法来完成消息的入队操作。

发送一个消息到消息队列的对尾，它会在处理这个时间的线程中的handleMessage(Message),方法中被接受到并且处理。