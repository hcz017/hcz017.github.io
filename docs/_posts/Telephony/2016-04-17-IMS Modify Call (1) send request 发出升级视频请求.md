---
date: 2016-04-18 20:12
Tags:
  - ims
status: draft
title: IMS Modify Call (1) send request 发出升级视频请求
---

QCOM IMS ModifyCall (1) send upgrade/downgrade request

upgrade/downgrade 升降级，其实就是ModifyCall

# 整体示意

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/IMS_modify%20call%20show.jpg)
图中有两种情况，一种是MT对收到的升级请求做出响应，一种是超时后服务器给双发发送消息。
一个完整的Modify call(upgrade)可以分为4个部分，本文主要讲第一部分发出升级请求。

看一下实际效果图，这个号码是印度的哦大家不要随便打（08.26更新，我们的代码中界面已经修改了）。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/IMS_MdifyCall.gif)

# 流程图

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/IMS_MdifyCall.jpg)
界面入手

## packages/apps/InCallUI

packages/apps/InCallUI/src/com/android/incallui/CallButtonFragment.java

CallButtonFragment.java onClick()

```java
case R.id.changeToVideoButton:
   if (call == null) {
       Log.i(this, "Call was null");
       return;
   }
   if (getResources().getBoolean(R.bool.config_regional_number_patterns_video_call) && //默认为false
       !CallUtil.isVideoCallNumValid(call.getNumber())) {
       Toast.makeText(this.getActivity(),
               R.string.toast_change_video_call_failed, Toast.LENGTH_LONG).show();
       return;
   }
   getPresenter().changeToVideoClicked();//切换为视频通话，实际是ModifyCall 后面可以看到调用
   break;
```

packages/apps/InCallUI/src/com/android/incallui/CallButtonPresenter.java

changeToVideoClicked()

本来还有一个changeToVoiceClick()的，不过没有使用。

```java
public void changeToVideoClicked() {
   final Context context = getUi().getContext();
   if (QtiCallUtils.useExt(context)) {//为true，可以看下面的代码断
       QtiCallUtils.displayModifyCallOptions(mCall, context);//这里继续
       return;
   }

   VideoCall videoCall = mCall.getVideoCall();
   if (videoCall == null) {
       return;
   }
   int currVideoState = mCall.getVideoState();
   int currUnpausedVideoState = CallUtils.getUnPausedVideoState(currVideoState);
   currUnpausedVideoState |= VideoProfile.STATE_BIDIRECTIONAL;

   VideoProfile videoProfile = new VideoProfile(currUnpausedVideoState);
   videoCall.sendSessionModifyRequest(videoProfile);
   mCall.setSessionModificationState(Call.SessionModificationState.WAITING_FOR_RESPONSE);
}
```

packages/apps/InCallUI/src/com/android/incallui/QtiCallUtils.java

QtiCallUtils.java

```java
/**
* Checks the boolean flag in config file to figure out if we are going to use Qti extension or
* not
*/
public static boolean useExt(Context context) {
   if (context == null) {
       Log.w(context, "Context is null...");
   }
   return context != null && context.getResources().getBoolean(R.bool.video_call_use_ext); //默认为true
}
```

displayModifyCallOptions() 显示选项（早先是在overflow menu中有个modify call选项，点击后弹出4个选项供选择）

```java
/**
* The function is called when Modify Call button gets pressed. The function creates and
* displays modify call options.
*/

public static void displayModifyCallOptions(final Call call, final Context context) {
   //一些判断条件
   if (call == null) {
       Log.d(LOG_TAG, "Can't display modify call options. Call is null");
       return;
   }

   if (isTtyEnabled(context)) {
       Log.w(LOG_TAG, "Call session modification is allowed only when TTY is off.");
       displayToast(context, R.string.video_call_not_allowed_if_tty_enabled);
       return;
   }

   if (context.getResources().getBoolean(
           R.bool.config_enable_enhance_video_call_ui)) {    //false，后面可能会改。这里会影响到CallButton，见下面解释
       // selCallType is set to -1 default, if the value is not updated, it is unexpected.
       if (selectType != -1) {
           VideoProfile videoProfile = new VideoProfile(selectType);
           Log.v(LOG_TAG, "Videocall: Enhance videocall: upgrade/downgrade to "
                   + callTypeToString(selectType));
           changeToVideoClicked(call, videoProfile);
       }
       return;
   }

   final ArrayList<CharSequence> items = new ArrayList<CharSequence>();
   final ArrayList<Integer> itemToCallType = new ArrayList<Integer>();
   final Resources res = context.getResources();

   // Prepare the string array and mapping.//开始准备modify call的dialog
   if (hasVoiceCapabilities(call)) {
       items.add(res.getText(R.string.modify_call_option_voice));
       itemToCallType.add(VideoProfile.STATE_AUDIO_ONLY);//仅语音
   }

   if (hasBidirectionalVideoCapabilities(call)) {
       items.add(res.getText(R.string.modify_call_option_vt_rx));
       itemToCallType.add(VideoProfile.STATE_RX_ENABLED);//接收视频，也就是只显示对面的视频

       items.add(res.getText(R.string.modify_call_option_vt_tx));
       itemToCallType.add(VideoProfile.STATE_TX_ENABLED);//发送。只发送不接收

       items.add(res.getText(R.string.modify_call_option_vt));
       itemToCallType.add(VideoProfile.STATE_BIDIRECTIONAL);//双向，双方视频互相传送。这里客户可能会希望把单向的去掉
   }

   AlertDialog.Builder builder = new AlertDialog.Builder(context);
   builder.setTitle(R.string.modify_call_option_title);
   final AlertDialog alert;

   DialogInterface.OnClickListener listener = new DialogInterface.OnClickListener() {
       @Override
       public void onClick(DialogInterface dialog, int item) {
           Toast.makeText(context, items.get(item), Toast.LENGTH_SHORT).show();
           final int selCallType = itemToCallType.get(item);
           Log.v(this, "Videocall: ModifyCall: upgrade/downgrade to "
                   + callTypeToString(selCallType));
           VideoProfile videoProfile = new VideoProfile(selCallType);
           changeToVideoClicked(call, videoProfile);//在弹出的窗口上点击以后
           dialog.dismiss();
       }
   };
   final int currUnpausedVideoState = CallUtils.getUnPausedVideoState(call.getVideoState());
   final int index = itemToCallType.indexOf(currUnpausedVideoState);
   builder.setSingleChoiceItems(items.toArray(new CharSequence[0]), index, listener);
   alert = builder.create();
   alert.show(); //显示alert dialog
}
```

关于 R.bool.config_enable_enhance_video_call_ui 这个值，如果为true的话，modify的几个选项会会直接在CalButton上显示（通常是在overFlow menu里）。ehhance UI嘛，就是直接在UI上把那些发送，接收，双向的button或者选项列出来。

changeToVideoClicked()

这个方法同时也被 downgradeToVoiceCall()调用了。可见这部分不能光看方法名字理解它的作用了。

```java
/**
* Sends a session modify request to the telephony framework
*/
private static void changeToVideoClicked(Call call, VideoProfile videoProfile) {
   VideoCall videoCall = call.getVideoCall();//这里有一个疑问，如果起先是语音电话呢，那这里还能拿到“VideoCall”么？
   if (videoCall == null) {
       return;
   }
   videoCall.sendSessionModifyRequest(videoProfile);//发送切换/更改请求
   call.setSessionModificationState(Call.SessionModificationState.WAITING_FOR_RESPONSE);
   InCallAudioManager.getInstance().onModifyCallClicked(call, videoProfile.getVideoState());//同步更改Audio状态，InCallAudioManager可看之前的博客
}
```

## frameworks/base/telecomm/java/android/telecom/VideoCallImpl.java

VideoCallImpl.java

```java
/**
* Sends a session modification request to the video provider.
* <p>
* The {@link InCallService} will create the {@code requestProfile} based on the current
* video state (i.e. {@link Call.Details#getVideoState()}).  It is, however, possible that the
* video state maintained by the {@link InCallService} could get out of sync with what is known
* by the {@link android.telecom.Connection.VideoProvider}.  To remove ambiguity, the
* {@link VideoCallImpl} passes along the pre-modify video profile to the {@code VideoProvider}
* to ensure it has full context of the requested change.
*
* @param requestProfile The requested video profile.
*/
public void sendSessionModifyRequest(VideoProfile requestProfile) {
   try {
       VideoProfile originalProfile = new VideoProfile(mCall.getDetails().getVideoState(),
               mVideoQuality);//拿到当前的状态，为什么要当前的？（这个问题在后面有解答，但是被我删掉了。。在高通私有代码部分是没有用到这个参数的，只用了requestProfile）

       mVideoProvider.sendSessionModifyRequest(originalProfile, requestProfile);
   } catch (RemoteException e) {
   }
}
```

frameworks/base/telecomm/java/android/telecom/Connection.java

VideoProviderBinder

```java
/**
* IVideoProvider stub implementation.
*/
private final class VideoProviderBinder extends IVideoProvider.Stub {

    public void sendSessionModifyRequest(VideoProfile fromProfile, VideoProfile toProfile) {
       SomeArgs args = SomeArgs.obtain();
       args.arg1 = fromProfile;
       args.arg2 = toProfile;
       mMessageHandler.obtainMessage(MSG_SEND_SESSION_MODIFY_REQUEST, args).sendToTarget();
    }
}
```

VideoProviderHandler

```java
case MSG_SEND_SESSION_MODIFY_REQUEST: {
   SomeArgs args = (SomeArgs) msg.obj;
   try {
       onSendSessionModifyRequest((VideoProfile) args.arg1,
               (VideoProfile) args.arg2);
   } finally {
       args.recycle();
   }
   break;
}
```

frameworks/base/telecomm/java/android/telecom/Connection.java
处理modify请求，判断请求是否可用等。

```java
/**
* Issues a request to modify the properties of the current video session.
* <p>
* Example scenarios include: requesting an audio-only call to be upgraded to a
* bi-directional video call, turning on or off the user's camera, sending a pause signal
* when the {@link InCallService} is no longer the foreground application.
* <p>
* If the {@link VideoProvider} determines a request to be invalid, it should call
* {@link #receiveSessionModifyResponse(int, VideoProfile, VideoProfile)} to report the
* invalid request back to the {@link InCallService}.
* <p>
* Where a request requires confirmation from the user of the peer device, the
* {@link VideoProvider} must communicate the request to the peer device and handle the
* user's response.  {@link #receiveSessionModifyResponse(int, VideoProfile, VideoProfile)}
* is used to inform the {@link InCallService} of the result of the request.
* <p>
* Sent from the {@link InCallService} via
* {@link InCallService.VideoCall#sendSessionModifyRequest(VideoProfile)}.
*
* @param fromProfile The video profile prior to the request.
* @param toProfile The video profile with the requested changes made.
*/
public abstract void onSendSessionModifyRequest(VideoProfile fromProfile,
       VideoProfile toProfile);
```

## frameworks/opt/net/ims/src/java/com/android/ims/internal/ImsVideoCallProviderWrapper.java

重写方法

```java
/** @inheritDoc */
public void onSendSessionModifyRequest(VideoProfile fromProfile, VideoProfile toProfile) {
   try {
       mVideoCallProvider.sendSessionModifyRequest(fromProfile, toProfile);
   } catch (RemoteException e) {
   }
}
```

frameworks/opt/net/ims/src/java/com/android/ims/internal/ImsVideoCallProvider.java

```java
public void sendSessionModifyRequest(VideoProfile fromProfile, VideoProfile toProfile) {
   SomeArgs args = SomeArgs.obtain();
   args.arg1 = fromProfile;
   args.arg2 = toProfile;
   mProviderHandler.obtainMessage(MSG_SEND_SESSION_MODIFY_REQUEST, args).sendToTarget();
}
```

mProviderHandler

```java
/**
* Default handler used to consolidate binder method calls onto a single thread.
*/
private final Handler mProviderHandler = new Handler(Looper.getMainLooper()) {
   @Override
   public void handleMessage(Message msg) {

        case MSG_SEND_SESSION_MODIFY_REQUEST: {
           SomeArgs args = (SomeArgs) msg.obj;
           try {
               VideoProfile fromProfile = (VideoProfile) args.arg1;
               VideoProfile toProfile = (VideoProfile) args.arg2;

               onSendSessionModifyRequest(fromProfile, toProfile);
           } finally {
               args.recycle();
           }
           break;
        }
    ...
    }
```

## vendor/qcom/proprietary/telephony-apps/ims/src/com/qualcomm/ims/vt/ImsVideoCallProviderImpl.java

高通私有代码，不贴了，一般也不会改下层的。

# 关键log

MO log

```xml
03-01 00:12:52.872  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) onSendSessionModifyRequest, videoState=0 quality= 4//请求转换为 AUDIO_ONLY
03-01 00:12:52.873  4918  4918 D VideoCall_ImsCallModification: validateOutgoingModifyConnectionType newCallType=0//CalllType 和videoState对应
03-01 00:12:52.873  4918  4918 D VideoCall_ImsCallModification: validateOutgoingModifyConnectionType modifyToCurrCallType = false isIndexValid = true isLowBattery = false
03-01 00:12:52.873  4918  4918 V ImsSenderRxr: modifyCallInitiate callModify=  1  0 2 callSubState 0 videoPauseState2 mediaId-1 Local Ability  Peer Ability  Cause code 0 0[SUB1]
03-01 00:12:52.873  4918  4918 V ImsSenderRxr: setCallModify callModify=  1  0 2 callSubState 0 videoPauseState2 mediaId-1 Local Ability  Peer Ability  Cause code 0 0[SUB1]
03-01 00:12:52.873  4918  4918 D ImsSenderRxr: [0043]> MODIFY_CALL_INITIATE[SUB1]
03-01 00:12:53.757  4918  5512 D ImsSenderRxr: [0043]< MODIFY_CALL_INITIATE [SUB1]
03-01 00:12:53.758  4918  4918 D VideoCall_ImsCallModification: EVENT_MODIFY_CALL_INITIATE_DONE received
03-01 00:12:53.758  4918  4918 D VideoCall_ImsCallModification: clearPendingModify imsconn=org.codeaurora.ims.ImsCallModification@1ea212
03-01 00:12:53.758  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) handleSessionModifyDone msg.what=0 //
03-01 00:12:53.758  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) Session modify success//成功切换

//upgrade
03-01 00:12:58.120  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) onSendSessionModifyRequest, videoState=3 quality= 4
03-01 00:12:58.120  4918  4918 D VideoCall_ImsCallModification: validateOutgoingModifyConnectionType newCallType=3//CalllType 和videoState对应
03-01 00:12:58.120  4918  4918 D VideoCall_ImsCallModification: validateOutgoingModifyConnectionType modifyToCurrCallType = false isIndexValid = true isLowBattery = false
03-01 00:12:58.120  4918  4918 V ImsSenderRxr: modifyCallInitiate callModify=  1  3 2 callSubState 0 videoPauseState2 mediaId-1 Local Ability  Peer Ability  Cause code 0 0[SUB1]
03-01 00:12:58.120  4918  4918 V ImsSenderRxr: setCallModify callModify=  1  3 2 callSubState 0 videoPauseState2 mediaId-1 Local Ability  Peer Ability  Cause code 0 0[SUB1]
03-01 00:12:58.120  4918  4918 D ImsSenderRxr: [0044]> MODIFY_CALL_INITIATE[SUB1]
03-01 00:13:02.989  4918  5512 D ImsSenderRxr: [0044]< MODIFY_CALL_INITIATE [SUB1]
03-01 00:13:02.989  4918  4918 D VideoCall_ImsCallModification: EVENT_MODIFY_CALL_INITIATE_DONE received
03-01 00:13:02.989  4918  4918 D VideoCall_ImsCallModification: clearPendingModify imsconn=org.codeaurora.ims.ImsCallModification@1ea212
03-01 00:13:02.990  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) handleSessionModifyDone msg.what=0
03-01 00:13:02.990  4918  4918 D VideoCall_ImsVideoCallProviderImpl: (1) Session modify success
```

# 小结

InCallUI会根据客户要求进行修改，建议提前熟悉代码，底层基本上不用动，大概跟一下流程就可以了。如果要改bug的话，底层的关键log还是要熟悉，尤其是关键log中携带的参数的意义。这个后面的博客中会提到。
