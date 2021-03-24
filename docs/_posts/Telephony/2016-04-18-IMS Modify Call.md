---
date:  2016-04-18 20:12
status: draft
title: 'IMS Modify Call'
---

界面入手
packages/apps/InCallUI/src/com/android/incallui/CallButtonFragment.java
CallButtonFragment.java onClick()
```java
case R.id.changeToVideoButton:
...
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
displayModifyCallOptions()
```java
/**
* The function is called when Modify Call button gets pressed. The function creates and
* displays modify call options.
*/
public static void displayModifyCallOptions(final Call call, final Context context) {
  //一些判断条件
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
   alert.show();//显示alert dialog
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
frameworks/base/telecomm/java/android/telecom/VideoCallImpl.java
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
               mVideoQuality);//拿到当前的状态，为什么要当前的？

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
frameworks/opt/net/ims/src/java/com/android/ims/internal/ImsVideoCallProviderWrapper.java
重写
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
vendor/qcom/proprietary/telephony-apps/ims/src/com/qualcomm/ims/vt/ImsVideoCallProviderImpl.java
```java
@Override
public void onSendSessionModifyRequest(VideoProfile fromProfile, VideoProfile toProfile) {
   log("onSendSessionModifyRequest, videoState=" + toProfile.getVideoState()
           + " quality= " + toProfile.getQuality());
   mRequestProfile = toProfile;
   if (!isSessionValid()) return;

   // If video pause is requested ignore the call type//用toProfile判断是不是暂停视频的请求
   if (isVideoPauseRequested(toProfile)) {
       mImsCallModification.changeConnectionType(null, CallDetails.CALL_TYPE_VT_PAUSE, null);
   } else if (mImsCallModification.isLocallyPaused()) {
       // If UE is locally paused, means that this is a resume request//如果当前是暂停状态，那么新的请求就是恢复视频
       mImsCallModification.changeConnectionType(null, CallDetails.CALL_TYPE_VT_RESUME, null);
   } else {
       // Neither pause or resume, so this is upgrade/downgrade request//既不是暂停也不是恢复 那就应该是升级/降级的请求。
       Message newMsg = mHandler.obtainMessage(EVENT_SEND_SESSION_MODIFY_REQUEST_DONE);
       int callType = ImsCallUtils.convertVideoStateToCallType(toProfile.getVideoState());
       mImsCallModification.changeConnectionType(newMsg, callType, null);
   }//果然fromProfile没有用到么 o_o
}
```
转换关系举例
case VideoProfile.STATE_AUDIO_ONLY:
   callType = CallDetails.CALL_TYPE_VOICE;
vendor/qcom/proprietary/telephony-apps/ims/src/org/codeaurora/ims/ImsCallModification.java
```java

// MODIFY_CALL_INITIATE
public void changeConnectionType(Message msg, int newCallType, Map<String, String> newExtras)
{
   log("changeConnectionType " + " newCallType=" + newCallType + " newExtras= "
           + newExtras);
   // mImsCallProfile = mImsCallSessionImpl.getCallId().getCallProfile();
   mIndex = Integer.parseInt(mImsCallSessionImpl.getCallId());
   if (isVTMultitaskRequest(newCallType)) {
       // Video pause/resume request
       triggerOrQueueVTMultitask(newCallType);
   } else {
       // Regular upgrade/downgrade request
       if (isAvpRetryAllowed() && ImsCallUtils.isVideoCallTypeWithDir(newCallType)) {
           mAvpCallType = newCallType;
       }

       Message newMsg = mHandler.obtainMessage(EVENT_MODIFY_CALL_INITIATE_DONE, msg);
       if (callModifyRequest == null) {
           if (validateOutgoingModifyConnectionType(newMsg, newCallType)) {
               modifyCallInitiate(newMsg, newCallType, newExtras);//看这里
           }
       } else {
           Log.e(LOG_TAG,
                   "videocall changeConnectionType: not invoking modifyCallInitiate "
                           + "as there is pending callModifyRequest="
                           + callModifyRequest);

           String errorStr = "Pending callModifyRequest so not sending modify request down";
           RuntimeException ex = new RuntimeException(errorStr);
           if (msg != null) {
               AsyncResult.forMessage(msg, null, ex);
               msg.sendToTarget();
           }

       }
   }
}
```
modifyCallInitiate
```java
private void modifyCallInitiate(Message newMsg, int newCallType, Map<String, String> newExtras)
{
   if (!ImsCallUtils.isValidRilModifyCallType(newCallType)) {//判断请求的目标类型是支持的
       loge("modifyCallInitiate not a Valid RilCallType" + newCallType);
       return;
   }

   /*
    * CallDetails callDetails = new CallDetails(newCallType, getCallDetails().call_domain,
    * CallDetails.getExtrasFromMap(newExtras));
    */
   CallDetails callDetails = new CallDetails(newCallType, mImsCallSessionImpl.getCallDomain(),
           CallDetails.getExtrasFromMap(newExtras));
   CallModify callModify = new CallModify(callDetails, mIndex);
   // Store the outgoing modify call request in the connection
   if (callModifyRequest != null) {
       log("Overwriting callModifyRequest: " + callModifyRequest + " with callModify:"
               + callModify);
   }
   callModifyRequest = callModify;
   mCi.modifyCallInitiate(newMsg, callModify);
}
```
vendor/qcom/proprietary/telephony-apps/ims/src/org/codeaurora/ims/ImsSenderRxr.java
```java
public void modifyCallInitiate(Message result, CallModify callModify) {
   logv("modifyCallInitiate callModify= " + callModify);//log
   byte[] callModifyb = setCallModify(callModify);
   encodeMsg(ImsQmiIF.REQUEST_MODIFY_CALL_INITIATE, result, callModifyb);//编码并发送
}
```
```java
private byte[] setCallModify(CallModify callModify) {
   logv("setCallModify callModify= " + callModify);//log
   ImsQmiIF.CallDetails callDetailsIF = new ImsQmiIF.CallDetails();
   callDetailsIF.setCallType(callModify.call_details.call_type);
   callDetailsIF.setCallDomain(callModify.call_details.call_domain);

   ImsQmiIF.CallModify callModifyIF = new ImsQmiIF.CallModify();
   callModifyIF.setCallDetails(callDetailsIF);
   callModifyIF.setCallIndex(callModify.call_index);

   // This field is not used for outgoing requests.
   // callModifyIF.setError(callModify.error);

   byte[] callModifyb = callModifyIF.toByteArray();
   return callModifyb;
}
```