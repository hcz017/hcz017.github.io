---
date: 2017-05-20 21:53
status: public
tags: Call
title: 'Call Screening(来电过滤过滤）on Android N'
---

之前在看Android官方文档系统的时候看到Android N 一系列新增和改变的特性，因为工作中负责的部分和通话有关，就尤其注意到了这个Call Screening。

下面来简单介绍一下Call Screening是什么，以及它在通话流程中起了哪些作用。

PS：一下代码全部来自Mokee mkn-mr1，

# 来自android官方的解释

>Call Screening
>---
>Android 7.0 allows the default phone app to screen incoming calls. The phone app does this by implementing the new CallScreeningService, which allows the phone app to perform a number of actions based on an incoming call's [Call.Details](https://developer.android.com/reference/android/telecom/Call.Details.html), such as:
>- Reject the incoming call
>- Do not allow the call to the call log
>- Do not show the user a notification for the call
>
>For more information, see the reference documentation for [CallScreeningService](https://developer.android.com/reference/android/telecom/CallScreeningService.html).


翻译一下

>来电过滤（Call Screening）
>Android 7.0 允许默认的手机应用过滤来电。手机应用执行此操作的方式是实现新的 CallScreeningService，该方法允许手机应用基于来电的 [Call.Details](https://developer.android.com/reference/android/telecom/Call.Details.html) 执行大量操作，例如：
>- 拒绝来电
>- 不允许来电到达通话记录
>- 不向用户显示来电通知
>
>如需了解详细信息，请参阅可下载的 [API 参考](https://developer.android.com/preview/setup-sdk.html#docs-dl)中的 android.telecom.CallScreeningService。

读者可能会奇怪 Call Screening翻译成“呼叫筛选”可能更合适些，来电过滤 或许应该是IncomingCall filter的翻译啊（其实这也是有道理的，看了代码就知晓了）。


上面的文字有两个重点 based on an incoming call's Call.Details和 CallScreeningService。

应该是从Call.Details来判断需不需要过滤，先不看去判断哪些具体的条件，我们先看看CallScreeningService在代码中是什么样子的。

# CallScreeningService

public abstract class CallScreeningService 
extends [Service](https://developer.android.com/reference/android/app/Service.html)

java.lang.Object
   ↳	android.content.Context
 	   ↳	android.content.ContextWrapper
 	 	   ↳	android.app.Service
 	 	 	   ↳	android.telecom.CallScreeningService

---
This service can be implemented by the default dialer (see [getDefaultDialerPackage()](https://developer.android.com/reference/android/telecom/TelecomManager.html#getDefaultDialerPackage%28%29)) to allow or disallow incoming calls before they are shown to a user.

Below is an example manifest registration for a CallScreeningService.
```xml
<service android:name="your.package.YourCallScreeningServiceImplementation"
          android:permission="android.permission.BIND_SCREENING_SERVICE">
      <intent-filter>
          <action android:name="android.telecom.CallScreeningService"/>
      </intent-filter>
 </service>
 ```
首先它是一个services，需要在拨号软件中实现，在AndroidManifest中的声明如上。

# 分析在源码中的使用
下面到源码中看这个是怎么使用的。

但是得到的第一个就是失望的消息，，

目前只有测试相关的代码 **cts/tests**/tests/telecom/src/android/telecom/cts/MockCallScreeningService.java 注册了CallScreeningService，也就是说我们在Android N中并看不到这个功能的实际使用。

但是我们依然可以窥探一下CallScreeningService的功能以及在现有代码中的作用

PS：所有代码来自Mokee mkn-mr1
```java
        public static class Builder {
            private boolean mShouldDisallowCall;
            private boolean mShouldRejectCall;
            private boolean mShouldSkipCallLog;
            private boolean mShouldSkipNotification;


            /*
             * Sets whether the incoming call should be blocked.
             */
            public Builder setDisallowCall(boolean shouldDisallowCall) {
                mShouldDisallowCall = shouldDisallowCall;
                return this;
            }


            /*
             * Sets whether the incoming call should be disconnected as if the user had manually
             * rejected it. This property should only be set to true if the call is disallowed.
             */
            public Builder setRejectCall(boolean shouldRejectCall) {
                mShouldRejectCall = shouldRejectCall;
                return this;
            }


            /*
             * Sets whether the incoming call should not be displayed in the call log. This property
             * should only be set to true if the call is disallowed.
             */
            public Builder setSkipCallLog(boolean shouldSkipCallLog) {
                mShouldSkipCallLog = shouldSkipCallLog;
                return this;
            }


            /*
             * Sets whether a missed call notification should not be shown for the incoming call.
             * This property should only be set to true if the call is disallowed.
             */
            public Builder setSkipNotification(boolean shouldSkipNotification) {
                mShouldSkipNotification = shouldSkipNotification;
                return this;
            }


            public CallResponse build() {
                return new CallResponse(
                        mShouldDisallowCall,
                        mShouldRejectCall,
                        mShouldSkipCallLog,
                        mShouldSkipNotification);
            }
       }
```

四个重要的方法：setDisallowCall(), setRejectCall(), setSkipCallLog(), setSkipNotification(), 从这4个方法也可以大概知道，这个“来电过滤”能做的具体的事情是什么：
1. 完全阻止来电（类似黑名单），
2. 是否拒接来电， 
3. 是否在CallLog中显示，
4. 是否要显示未接通知（这跟前面不就是一样的嘛），也就是说 可以通过这4个方法设置来电的提醒方式。

# 在cts代码中使用

再看一下在cts代码中的使用的
```java
    private CallScreeningServiceCallbacks createCallbacks() {
        return new CallScreeningServiceCallbacks() {
            @Override
            public void onScreenCall(Call.Details callDetails) {
                mCallFound = true;
                CallScreeningService.CallResponse response =
                        new CallScreeningService.CallResponse.Builder()
                        .setDisallowCall(true)
                        .setRejectCall(true)
                        .setSkipCallLog(true)
                        .setSkipNotification(true)
                        .build();
                getService().respondToCall(callDetails, response);
                lock.release();
            }
        };
    }

```
2333 全是true、、

这个Call Screening其实可以看作是黑名但的补充功能（那么黑名单又叫做什么呢？Number Blocking-官方名号码屏蔽）。这么推测的话，这两者应该是协同工作的。

# 来电log
我们借用log来看一下来电的大体流程
```log
//正常来电
04-25 14:51:26.947 I/Telecom ( 2596): Event: Call TC@18: CREATED, null: TSI.aNIC@DOU
04-25 14:51:26.951 I/Telecom ( 2596): Event: Call TC@18: BIND_CS, ComponentInfo{com.android.phone/com.android.services.telephony.TelephonyConnectionService}: TSI.aNIC@DOU
04-25 14:51:26.962 I/Telecom ( 2596): Event: Call TC@18: CS_BOUND, ComponentInfo{com.android.phone/com.android.services.telephony.TelephonyConnectionService}: SBC.oSC@DOY
04-25 14:51:26.963 I/Telecom ( 2596): Event: Call TC@18: START_CONNECTION, tel:***********: SBC.oSC@DOY
//初始化过滤器
04-25 14:51:26.988 I/Telecom ( 2596): Event: Call TC@18: FILTERING_INITIATED, null: CSW.hCCC@DOg
//直接转至语音信箱初始化
04-25 14:51:26.988 I/Telecom ( 2596): Event: Call TC@18: DIRECT_TO_VM_INITIATED, null: CSW.hCCC@DOg
04-25 14:51:26.988 I/Telecom ( 2596): Event: Call TC@18: DIRECT_TO_VM_FINISHED, [Allow, logged, notified]: CSW.hCCC@DOg
//Screening
04-25 14:51:26.989 I/Telecom ( 2596): Event: Call TC@18: SCREENING_SENT, null: CSW.hCCC@DOg
//黑名单过滤初始化
04-25 14:51:26.989 I/Telecom ( 2596): Event: Call TC@18: BLOCK_CHECK_INITIATED, null: CSW.hCCC->ABCF.dIB@DOg_1
//Call Screening的 三个参数 允许来电，记录到通话记录，显示通知
04-25 14:51:26.991 I/Telecom ( 2596): Event: Call TC@18: SCREENING_COMPLETED, [Allow, logged, notified]: CSW.hCCC@DOg
//我们前面说过Call Screening并没有真正用起来
04-25 14:51:26.991 I/Telecom ( 2596): CallScreeningServiceFilter: There are no call screening services installed on this device.: CSW.hCCC@DOg
04-25 14:51:26.991 I/Telecom ( 2596): CallScreeningServiceFilter: Could not bind to call screening service: CSW.hCCC@DOg
04-25 14:51:27.007 I/Telecom ( 2596): Event: Call TC@18: BLOCK_CHECK_FINISHED, [Allow, logged, notified]: CSW.hCCC->ABCF.oPE@DOg_2
//过滤结束
04-25 14:51:27.009 I/Telecom ( 2596): Event: Call TC@18: FILTERING_COMPLETED, [Allow, logged, notified]: CSW.hCCC->ABCF.oPE->ICF.oCFC@DOg_2_0
//响铃 成功来电
04-25 14:51:27.011 I/Telecom ( 2596): Event: Call TC@18: SET_RINGING, successful incoming call: CSW.hCCC->ABCF.oPE->ICF.oCFC@DOg_2_0 //mk_in
04-25 14:51:27.016 I/Telecom ( 2596): Event: Call TC@18: START_RINGER, null: CSW.hCCC->ABCF.oPE->ICF.oCFC->CAMSM.pM_2002@DOg_2_0_1
//未接
04-25 14:51:40.063 I/Telecom ( 2596): Event: Call TC@18: SET_DISCONNECTED, disconnected set explicitly> DisconnectCause [ Code: (MISSED) Label: () Description: () Reason: (INCOMING_MISSED) Tone: (-1) ]: CSW.sDc@DQI
04-25 14:51:40.099 I/Telecom ( 2596): Event: Call TC@18: DESTROYED, null: CSW.rC@DQk
 
 
//黑名单号码来电
04-25 14:52:20.925 I/Telecom ( 2596): Event: Call TC@19: CREATED, null: TSI.aNIC@DRo
04-25 14:52:20.932 I/Telecom ( 2596): Event: Call TC@19: BIND_CS, ComponentInfo{com.android.phone/com.android.services.telephony.TelephonyConnectionService}: TSI.aNIC@DRo
04-25 14:52:20.950 I/Telecom ( 2596): Event: Call TC@19: CS_BOUND, ComponentInfo{com.android.phone/com.android.services.telephony.TelephonyConnectionService}: SBC.oSC@DRs
04-25 14:52:20.951 I/Telecom ( 2596): Event: Call TC@19: START_CONNECTION, tel:***********: SBC.oSC@DRs
04-25 14:52:20.973 I/Telecom ( 2596): Event: Call TC@19: CAPABILITY_CHANGE, Current: [[ sup_hld mut !v2a spd_aud]], Removed [[]], Added [[ sup_hld mut !v2a spd_aud]]: CSW.hCCC@DR0
04-25 14:52:20.974 I/Telecom ( 2596): Event: Call TC@19: FILTERING_INITIATED, null: CSW.hCCC@DR0
04-25 14:52:20.974 I/Telecom ( 2596): Event: Call TC@19: DIRECT_TO_VM_INITIATED, null: CSW.hCCC@DR0
04-25 14:52:20.975 I/Telecom ( 2596): Event: Call TC@19: SCREENING_SENT, null: CSW.hCCC@DR0
04-25 14:52:20.976 I/Telecom ( 2596): Event: Call TC@19: BLOCK_CHECK_INITIATED, null: CSW.hCCC->ABCF.dIB@DR0_0
04-25 14:52:20.980 I/Telecom ( 2596): Event: Call TC@19: SCREENING_COMPLETED, [Allow, logged, notified]: CSW.hCCC@DR0
04-25 14:52:20.981 I/Telecom ( 2596): Event: Call TC@19: DIRECT_TO_VM_FINISHED, [Allow, logged, notified]: TSI.aNIC->CILH.sL->CILH.oQC@DRo_1_0
//黑名单阻止 可以看到输出有Reject
04-25 14:52:20.994 I/Telecom ( 2596): Event: Call TC@19: BLOCK_CHECK_FINISHED, [Reject]: CSW.hCCC->ABCF.oPE@DR0_1
04-25 14:52:20.995 I/Telecom ( 2596): Event: Call TC@19: FILTERING_COMPLETED, [Reject]: CSW.hCCC->ABCF.oPE->ICF.oCFC@DR0_1_0
//响铃这一步 被block了
04-25 14:52:21.001 I/Telecom ( 2596): Event: Call TC@19: SET_RINGING, blocking call: CSW.hCCC->ABCF.oPE->ICF.oCFC@DR0_1_0 //mk_in
04-25 14:52:21.002 I/Telecom ( 2596): Event: Call TC@19: REQUEST_REJECT, null: CSW.hCCC->ABCF.oPE->ICF.oCFC@DR0_1_0
//本地拒接
04-25 14:52:21.395 I/Telecom ( 2596): Event: Call TC@19: SET_DISCONNECTED, disconnected set explicitly> DisconnectCause [ Code: (REJECTED) Label: () Description: () Reason: (INCOMING_REJECTED) Tone: (-1) ]: CSW.sDc@DSI
04-25 14:52:21.400 I/Telecom ( 2596): Event: Call TC@19: DESTROYED, null: CSW.rC@DSM
```

PS: Event这个关键字是在M引入的，我只在前Android M MO的blog中提提到过，是专**门记录关键流程点/事件的log**，至于后面那一对缩写词还没去查。

从上面的log中看到Telecom Event log中已经加入了几种过滤相关的log，这也意味着虽然今天的主角虽然没有“暴露”出来，但是已经“安插”进去了。

# 来电流程中与来电过滤有关的部分

1.来电 创建过滤器
packages/services/**Telecomm**/src/com/android/server/telecom/CallsManager.java

```java
    public void onSuccessfulIncomingCallRewrite(Call incomingCall) {
        Log.d(this, "onSuccessfulIncomingCall");
        if (incomingCall.hasProperty(Connection.PROPERTY_EMERGENCY_CALLBACK_MODE)) {//如果是紧急号码回拨则不过滤
            Log.i(this, "Skipping call filtering due to ECBM");
            onCallFilteringComplete(incomingCall, new CallFilteringResult(true, false, true, true));
            return;
        }


//过滤器s
        List<IncomingCallFilter.CallFilter> filters = new ArrayList<>();
        filters.add(new DirectToVoicemailCallFilter(mCallerInfoLookupHelper));
        filters.add(new AsyncBlockCheckFilter(mContext, new BlockCheckerAdapter()));
        filters.add(new CallScreeningServiceFilter(mContext, this, mPhoneAccountRegistrar,//Call Screening
                mDefaultDialerManagerAdapter,
                new ParcelableCallUtils.Converter(), mLock));
        new IncomingCallFilter(mContext, this, incomingCall, mLock,//这三个过滤器都归属于IncomingCallFilter
                mTimeoutsAdapter, filters).performFiltering();//调用过滤方法
    }
```
2.准备过滤
packages/services/**Telecomm**/src/com/android/server/telecom/callfiltering/IncomingCallFilter.java

```java
    public void performFiltering() {
        Log.event(mCall, Log.Events.FILTERING_INITIATED);
        for (CallFilter filter : mFilters) {//遍历过滤器调用起各自的过滤方法
            filter.startFilterLookup(mCall, this);
        }
        // synchronized to prevent a race on mResult and to enter into Telecom.
        mHandler.postDelayed(new Runnable("ICF.pFTO", mTelecomLock) { // performFiltering time-out
            @Override
            public void loggedRun() {
                if (mIsPending) {
                    Log.i(IncomingCallFilter.this, "Call filtering has timed out.");
                    Log.event(mCall, Log.Events.FILTERING_TIMED_OUT);
                    mListener.onCallFilteringComplete(mCall, mResult);
                    mIsPending = false;
                }
            }
        }.prepare(), mTimeoutsAdapter.getCallScreeningTimeoutMillis(mContext.getContentResolver()));
    }
```
3.过滤器各自过滤
packages/services/**Telecomm**/src/com/android/server/telecom/callfiltering/**CallScreeningServiceFilter**.java

log我们看过了 Could not bind to call screening service，2333 没得搞，全剧终。

```java
    @Override
    public void startFilterLookup(Call call, CallFilterResultCallback callback) {
        if (mHasFinished) {
            Log.w(this, "Attempting to reuse CallScreeningServiceFilter. Ignoring.");
            return;
        }
        Log.event(call, Log.Events.SCREENING_SENT);
        mCall = call;
        mCallback = callback;
        if (!bindService()) {
            Log.i(this, "Could not bind to call screening service");
            finishCallScreening();
        }
    }
```

因为系统Dialer没有应用这个功能嘛，所以其实到这里是断了的。那我们假如bindService()成功了呢？

```java
    private void onServiceBound(ICallScreeningService service) {
        mService = service;
        try {
            mService.screenCall(new CallScreeningAdapter(),
                    mParcelableCallUtilsConverter.toParcelableCall(
                            mCall,
                            false, /* includeVideoProvider */
                            mPhoneAccountRegistrar));
        } catch (RemoteException e) {
            Log.e(this, e, "Failed to set the call screening adapter.");
            finishCallScreening();
        }
    }
```

frameworks/base/**telecomm**/java/android/telecom/CallScreeningService.java

```java
    private final class CallScreeningBinder extends ICallScreeningService.Stub {
        @Override
        public void screenCall(ICallScreeningAdapter adapter, ParcelableCall call) {
            Log.v(this, "screenCall");
            SomeArgs args = SomeArgs.obtain();
            args.arg1 = adapter;
            args.arg2 = call;
            mHandler.obtainMessage(MSG_SCREEN_CALL, args).sendToTarget();
        }
    }
```

```java
    private final Handler mHandler = new Handler(Looper.getMainLooper()) {
        @Override
        public void handleMessage(Message msg) {
            switch (msg.what) {
                case MSG_SCREEN_CALL:
                    SomeArgs args = (SomeArgs) msg.obj;
                    try {
                        mCallScreeningAdapter = (ICallScreeningAdapter) args.arg1;
                        onScreenCall(
                                Call.Details.createFromParcelableCall((ParcelableCall) args.arg2));
                    } finally {
                        args.recycle();
                    }
                    break;
            }
        }
    };
```

过滤，具体内容要看哪里实现了这个方法
```java
    /**
     * Called when a new incoming call is added.
     * {@link CallScreeningService#respondToCall(Call.Details, CallScreeningService.CallResponse)}
     * should be called to allow or disallow the call.
     *
     * @param callDetails Information about a new incoming call, see {@link Call.Details}.
     */
    public abstract void onScreenCall(Call.Details callDetails);
```
过滤完回调
```java
    /**
     * Responds to the given call, either allowing it or disallowing it.
     *
     * @param callDetails The call to allow.
     * @param response The {@link CallScreeningService.CallResponse} which contains information
     * about how to respond to a call.
     */
    public final void respondToCall(Call.Details callDetails, CallResponse response) {
        try {
            if (response.getDisallowCall()) {
                mCallScreeningAdapter.disallowCall(
                        callDetails.getTelecomCallId(),
                        response.getRejectCall(),
                        !response.getSkipCallLog(),
                        !response.getSkipNotification());
            } else {
                mCallScreeningAdapter.allowCall(callDetails.getTelecomCallId());
            }
        } catch (RemoteException e) {
        }
    }
```
packages/services/**Telecomm**/src/com/android/server/telecom/callfiltering/CallScreeningServiceFilter.java  

然后response.getDisallowCall()判断是否允许，接着处理结果

```java
    private class CallScreeningAdapter extends ICallScreeningAdapter.Stub {
        @Override
        public void allowCall(String callId) {
            Log.startSession("CSCR.aC");
            long token = Binder.clearCallingIdentity();
            try {
                synchronized (mTelecomLock) {
                    Log.d(this, "allowCall(%s)", callId);
                    if (mCall != null && mCall.getId().equals(callId)) {
                        mResult = new CallFilteringResult(
                                true, // shouldAllowCall
                                false, //shouldReject
                                true, //shouldAddToCallLog
                                true // shouldShowNotification
                        );
                    } else {
                        Log.w(this, "allowCall, unknown call id: %s", callId);
                    }
                    finishCallScreening();
                }
            } finally {
                Binder.restoreCallingIdentity(token);
                Log.endSession();
            }
        }


        @Override
        public void disallowCall(
                String callId,
                boolean shouldReject,
                boolean shouldAddToCallLog,
                boolean shouldShowNotification) {
            Log.startSession("CSCR.dC");
            long token = Binder.clearCallingIdentity();
            try {
                synchronized (mTelecomLock) {
                    Log.i(this, "disallowCall(%s), shouldReject: %b, shouldAddToCallLog: %b, "
                                    + "shouldShowNotification: %b", callId, shouldReject,
                            shouldAddToCallLog, shouldShowNotification);
                    if (mCall != null && mCall.getId().equals(callId)) {
                        mResult = new CallFilteringResult(
                                false, // shouldAllowCall
                                shouldReject, //shouldReject
                                shouldAddToCallLog, //shouldAddToCallLog
                                shouldShowNotification // shouldShowNotification
                        );
                    } else {
                        Log.w(this, "disallowCall, unknown call id: %s", callId);
                    }
                    finishCallScreening();
                }
            } finally {
                Binder.restoreCallingIdentity(token);
                Log.endSession();
            }
        }
    }
```
packages/services/Telecomm/src/com/android/server/telecom/callfiltering/CallScreeningServiceFilter.java

回调至IncomingCallFilter，解绑service

```java
    private void finishCallScreening() {
        if (!mHasFinished) {
            Log.event(mCall, Log.Events.SCREENING_COMPLETED, mResult);
            mCallback.onCallFilteringComplete(mCall, mResult);


            if (mConnection != null) {
                // We still need to call unbind even if the service disconnected.
                mContext.unbindService(mConnection);//解绑
                mConnection = null;
            }
            mService = null;
            mHasFinished = true;
        }
    }
```
回调至IncomingCallFilter

packages/services/**Telecomm**/src/com/android/server/telecom/callfiltering/IncomingCallFilter.java

```java
    public void onCallFilteringComplete(Call call, CallFilteringResult result) {
        synchronized (mTelecomLock) { // synchronizing to prevent race on mResult
            mNumPendingFilters--;
            mResult = result.combine(mResult);
            if (mNumPendingFilters == 0) {
                // synchronized on mTelecomLock to enter into Telecom.
                mHandler.post(new Runnable("ICF.oCFC", mTelecomLock) {
                    @Override
                    public void loggedRun() {
                        if (mIsPending) {
                            Log.event(mCall, Log.Events.FILTERING_COMPLETED, mResult);
                            mListener.onCallFilteringComplete(mCall, mResult);
                            mIsPending = false;
                        }
                    }
                }.prepare());
            }
        }
    }
```

结果返回到CallsManager

如果允许call，

如果不允许call，再从result里面读shouldReject等，然后就对应开头部分提到的四种现象

 
```java
    @Override
    public void onCallFilteringComplete(Call incomingCall, CallFilteringResult result) {
        // Only set the incoming call as ringing if it isn't already disconnected. It is possible
        // that the connection service disconnected the call before it was even added to Telecom, in
        // which case it makes no sense to set it back to a ringing state.
        if (incomingCall.getState() != CallState.DISCONNECTED &&
                incomingCall.getState() != CallState.DISCONNECTING) {
            setCallState(incomingCall, CallState.RINGING,
                    result.shouldAllowCall ? "successful incoming call" : "blocking call");
        } else {
            Log.i(this, "onCallFilteringCompleted: call already disconnected.");
            return;
        }


        if (result.shouldAllowCall) {
            if (hasMaximumRingingCalls(incomingCall.getTargetPhoneAccount().getId())) {
                if (shouldSilenceInsteadOfReject(incomingCall)) {
                    incomingCall.silence();
                } else {
                    Log.i(this, "onCallFilteringCompleted: Call rejected! " +
                            "Exceeds maximum number of ringing calls.");
                    rejectCallAndLog(incomingCall);
                }
            } else if (hasMaximumDialingCalls()) {
                Log.i(this, "onCallFilteringCompleted: Call rejected! Exceeds maximum number of " +
                        "dialing calls.");
                rejectCallAndLog(incomingCall);
            } else if (!isIncomingVideoCallAllowed(incomingCall, mContext)) {
                Toast.makeText(mContext, mContext.getResources().
                        getString(R.string.incoming_call_failed_low_battery), Toast.LENGTH_LONG).
                        show();
                rejectCallAndLog(incomingCall);
            } else {
                addCall(incomingCall);
                setActiveSubscription(incomingCall.getTargetPhoneAccount().getId());
            }
        } else {
            if (result.shouldReject) {
                Log.i(this, "onCallFilteringCompleted: blocked call, rejecting.");
                incomingCall.reject(false, null);
            }
            if (result.shouldAddToCallLog) {
                Log.i(this, "onCallScreeningCompleted: blocked call, adding to call log.");
                if (result.shouldShowNotification) {
                    Log.w(this, "onCallScreeningCompleted: blocked call, showing notification.");
                }
                mCallLogManager.logCall(incomingCall, Calls.MISSED_TYPE,
                        result.shouldShowNotification);
            } else if (result.shouldShowNotification) {
                Log.i(this, "onCallScreeningCompleted: blocked call, showing notification.");
                mMissedCallNotifier.showMissedCallNotification(incomingCall);//未接通知
            }
        }
    }
```
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/call_screening.png)


看到这里还有两个问题需要解决：
1. Call Screening  内部是如何工作的，也就是它如何匹配号码的？
2. 我们怎么将一个号码添加到需要过滤的那个“列表”里？