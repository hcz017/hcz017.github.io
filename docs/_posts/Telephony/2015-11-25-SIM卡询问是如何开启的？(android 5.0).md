---
date: 2015-11-25 09:18
status: public
tags: Call
title: 'SIM卡询问是如何开启的？(android 5.0)'
---

本篇解决两个问题
* SIM卡询问是如何开启的？
* 通话中再添加通话为何不会再次弹出SIM卡询问框

PS:只考虑SIM卡账户的情况，不考虑有其他账户（sip等）的情况。

#SIM卡询问是如何开启的？

##SIM卡询问框弹出的条件
我们知道SIM卡是在SelecPhoneAccountDialogFragment.java中new出来并显示的，那么是如何走到这一步的呢？
跟踪拨打电话的流程我们知道在InCallActivity.java中的internalResolveIntent()方法中有一行
`Call pendingAccountSelectionCall = CallList.getInstance().getWaitingForAccountCall();`
这个pendingAccountSelectionCall 做什么的呢？
如果不为空的话，也即有一个PRI_DIAL_WAIT状态的call的话，就会调用`SelectPhoneAccountDialogFragment.showAccountDialog()`去显示一个选择框。
这里我们有一个疑问，我们还么有拨打出电话的时候为什么CallList里面就有Call了呢？

##设置call的状态
如果你熟悉MO流程的话，那么你应该知道一个完整的MO流程是从dialer开始经telecom然后才到incallui的，而InCallActivity正是在InCallUI中的。因此我们推断，这个PRI_DIAL_WAIT的状态是在Telecomm中设置进去的。
经过跟踪流程发现，在CallsManager.java里面有个关键的Call类型的方法startOutgoingCall()，这中间有个变量控制着是否需要显示sim卡询问框needsAccountSelection ，部分代码如下：
```java
    boolean needsAccountSelection = phoneAccountHandle == null && accounts.size() > 1 &&
                !isEmergencyCall;
        Log.i(this, "Voice needsAccountSelection: "+needsAccountSelection);
        if (needsAccountSelection) {
            // This is the state where the user is expected to select an account
            call.setState(CallState.PRE_DIAL_WAIT);
            extras.putParcelableList(android.telecom.Call.AVAILABLE_PHONE_ACCOUNTS, accounts);
        } else {
            call.setState(CallState.CONNECTING);
        }
```

如果我们没有选定默认通话的SIM卡的话，phoneAccountHandle就为null，因此CallList中就有了一个状态为PRE_DIAL_WAIT的call，因此后面走到InCallActivity里的时候会执行显示SIM卡选择狂的代码。

#通话中添加通话为何不会再弹出SIM卡选择？

由前面的分析我们知道，是否弹出SIM卡选择框关键在于有没有PRE_DIAL_WAIT状态的call，而call的状态是否要设置为PRE_DIAL_WAIT取决于needsAccountSelection 的值，
`boolean needsAccountSelection = phoneAccountHandle == null && accounts.size() > 1 &&!isEmergencyCall;`
显然当phoneAccountHandle 不为null的时候就不需要显示SIM卡选择，我们往上看一段代码
```java
    if (phoneAccountHandle == null) {
            // No preset account, check if default exists that supports the URI scheme for the
            // handle.
            PhoneAccountHandle defaultAccountHandle =
                    mPhoneAccountRegistrar.getDefaultOutgoingPhoneAccount(
                            scheme);
            TelephonyManager.MultiSimVariants msimConfig =
                    TelephonyManager.getDefault().getMultiSimConfiguration();
            if (((msimConfig == TelephonyManager.MultiSimVariants.DSDS) ||
                    (msimConfig == TelephonyManager.MultiSimVariants.TSTS)) &&
                    (mForegroundCall != null) && (mForegroundCall.isAlive())) {
                defaultAccountHandle = mForegroundCall.getTargetPhoneAccount();
            }
            if (defaultAccountHandle != null) {
                phoneAccountHandle = defaultAccountHandle;
            }
        }

        call.setTargetPhoneAccount(phoneAccountHandle);
```

如果手机是DSDS或者TSTS的时候，会将forgroundCall的TargetPhoneAccount赋值给phoneAccountHandle，phoneAccountHandle不为null，也就不需要显示SIM卡选择框，然后phoneAccountHandle 设为新的call的TargetPhoneAccount，此时新的call的TargetPhoneAccount和当前存在的forgroundCall的TargetPhoneAccount相同。

那么PhoneAccountHandle到底是干啥的？

PS：
*  Dual SIM Dual Standby     双卡双待单通
*  DSDA - Dual SIM Dual Active双卡双待双通
*  TSTS - Triple SIM Triple Standby三卡三待