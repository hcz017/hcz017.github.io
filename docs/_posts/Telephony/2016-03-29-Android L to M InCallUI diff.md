---
date: 2016-03-29 19:34
status: draft
tags: incallui Call
title: 'InCallUI difference between L and M（草稿）'
---

# 新增或者修改的文件
## 新增
1. updateButtonsStateic_HD_24.dp
新增了一个显示HD图标的资源文件，Reliance L上我们是自己画上去的，那么到M版本可能就用由官方提供的资源了。
3. CircularRevealFragment.java
4. InCallDateUtils.java
5. InCallUIMaterialControllerMapUtils.java
6. NotificationBrodcastReceiver.java
7. InCallAudioManager.java 单独讲
8. InCallMessageController.java

```java
/**
* This class listens to incoming events for the listener classes it implements. It should
* handle all UI notification to be shown to the user for any indication that is required to be
* shown like call substate indication, video quality indication, etc.
* For e.g., this class implements {@class InCallSubstateListener} and when call substate changes,
* {@class CallSubstateNotifier} notifies it through the onCallSubstateChanged callback.
*/
```
## 更名
1. AudioState.java - > CallAudioState.java Telecom
2. InCallVideoCallListener.java(InCallVideoCallCallback)
3. InCallVideoCallCallbackNotifier.java -> InCallVideoCallListenerNotifier.java
4. InCallPresenter实现了它的接口
5. InCallApp -> NotificationBroadcastReceiver
6. call_card_content.xml  > call_card_fragment.xml
布局文件更名，明明规则上更加统一了。
7. dtmf_twelve_key_view.xml > incall_dialpad_fragment.xml （VT dialpad变化很大下面有个commit message可以参考下）
1. InCallPhoneListener.java -> InCallServiceListener.java onAudioStateChanged从Phone.java移动到了InCallServiceImpl.java

## 内容变化
增加录音功能（CM可能跟QCOM不一样）
1. maybeAutoEnterFullscreen  高通貌似没有考虑hold state
2. updateAudioMode() 中打开speaker的代码被注释了，使用InCallAudioManager管理，新代码里也没考虑hold 
3. CallButton中的audioButton 由ImageButton改成了ToogleButton
4. QtiCallUtils VideoQuality displayToast (InCallMessage)
QCOM
InCallAudioStateManger.java
5. updaCallButtons updateVideoCallButton updateVoiceCallButton ->  updateButtonsState
6. InCallActivity 
7. InCallPresenterjava setActivity -> setActivity()+unsetActivity()+updateActivity()
InCallZoomController

 重新进入incall 退出全屏模式

可能比较重要的几个提交：
---
>87bb4d0 Tyler Gunn on 7/1/15 at 4:15 AM (committed by Linux Build Service Account on 10/6/15 at 5:20 PM)
**Show dialpad button for VT calls, cleanup dialpad on rotation.**

>1. Show the dialpad button for VT calls (this was easy).
>2. In testing I realized there were some other dialpad scenarios that
did not work properly:
>- The dialpad visibility state was not properly restored after rotation.
>- The auto-fullscreen code could hide the call card when the dialpad was
showing, resulting in an inability to hide the dialpad.
>- In landscape it was possible to tap between the call card and the
dialpad and cause the call card to be hidden.
>- If user goes to background in fullscreen mode and then opens dialer and
chooses to show the dialpad, the app is still in fullscreen and it is not
possible to hide the dialpad.
>3. Noticed an issue related to the fact mIsFullScreen in InCallPresenter
is static, and after rotation you're always defaulting to not fullscreen.
Fixed by clearing fullscreen state on rotation to match actuality.

>Bug: 21296950
>Change-Id: I8549ad0e6b2357bd5795cb02f867c08c58ee5762

---

>6e4513c Ravi Paluri on 10/1/15 at 9:56 PM (committed by Gerrit - the friendly Code Review server on 10/7/15 at 2:41 AM)
**IMS-VT: Show Answer UI on receiving upgrade request**

>Show Answer UI on receiving upgrade request to enable
the user to respond to the upgrade request.
>
>Change-Id: Iefb8b14253efd2f68b516fd38c7802fe9a50bf51
CRs-Fixed: 912672

L上的做法是显示notification？可能不是一种场景

---
>14b0add Tyler Gunn on 5/12/15 at 11:55 PM [InCallUI]
Handle fullscreen mode toggle when tapping contact photo.

>1. Moved the fullscreen mode state tracking and logic from
VideoCallPresenter to InCallPresenter (since it is more to do with the
entire InCall UI rather than just the video presenter).
>2. Added click handler for the contact card to toggle fullscreen video
mode.

>Bug: 20912417
Change-Id: If1656365f7fcfcee5a902c7f5d5d2862edee1661

把setFullScreen()从VideoCallPresenter移动到了InCallPresenter，但是其中一个调用地方是CallCardFragment.onContactPhotoClick()，这里有有点疑问，如果现在是视频模式的话，那点击的应该是Video而不是联系人Photo啊，如果不是视频模式，那没必要进入全屏模式啊。
另：是否自动进入全屏模式还是在VideoCallPresenter.java


>0ea6ed9 Santos Cordon on 4/16/15 at 3:44 AM (committed on 4/17/15 at 1:09 AM) [InCallUI]
Remove usage of Phone.java in InCallService APIs.

>Start using the direct methods of InCallService instead of using the
Phone object.  InCallService methods represent the public API which is
what In-Call needs to compile against.

>Bug: 20160495
Change-Id: I223347e239e5d5954b6118c7ba5befdaea2932a0

---




VideoCallFragment

```java
/**
* Adjusts the location of the video preview view by the specified offset.
*
* @param shiftUp {@code true} if the preview should shift up, {@code false} if it should shift
*      down.
* @param offset The offset.
*/
@Override
public void adjustPreviewLocation(boolean shiftUp, int offset) {
   if (sPreviewSurface == null || mPreviewVideoContainer == null) {
       return;
   }

   // Set the position of the secondary call info card to its starting location.
   mPreviewVideoContainer.setTranslationY(shiftUp ? 0 : -offset);

   // Animate the secondary card info slide up/down as it appears and disappears.
   mPreviewVideoContainer.animate()
           .setInterpolator(AnimUtils.EASE_OUT_EASE_IN)
           .setDuration(mAnimationDuration)
           .translationY(shiftUp ? -offset : 0)
           .start();
}
```