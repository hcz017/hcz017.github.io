---
date: 2016-03-20 13:29
status: public
tags: audio
title: 'Audio in Video Call(Android_L)'
---

之前有一篇博客提到了audioButton，其中涉及了一部分audio相关的内容，这次主要将一下视频电话中的audio。
 
以下内容以QCOM android L为基础。
下面是AudioState中AudioRoute对应的设备
```java
    /** Direct the audio stream through the device's earpiece. */
    public static final int ROUTE_EARPIECE      = 0x00000001;

    /** Direct the audio stream through Bluetooth. */
    public static final int ROUTE_BLUETOOTH     = 0x00000002;

    /** Direct the audio stream through a wired headset. */
    public static final int ROUTE_WIRED_HEADSET = 0x00000004;

    /** Direct the audio stream through the device's speakerphone. */
    public static final int ROUTE_SPEAKER       = 0x00000008;

    /**
     * Direct the audio stream through the device's earpiece or wired headset if one is
     * connected.
     */
    public static final int ROUTE_WIRED_OR_EARPIECE = ROUTE_EARPIECE | ROUTE_WIRED_HEADSET;

    /** Bit mask of all possible audio routes. */
    private static final int ROUTE_ALL = ROUTE_EARPIECE | ROUTE_BLUETOOTH | ROUTE_WIRED_HEADSET |
            ROUTE_SPEAKER;

```
##What we got?

 首先说明一点，在我们起初拿到的代码中，在整个VideoCallPresenter中只有一个方法中设置了setAudioRoute，并且这个方法中只有一处明确设置了打开打开Speaker，代码段:
 

```java
     private void updateAudioMode(boolean enableSpeaker) {
        if (!isSpeakerEnabledForVideoCalls()) {
            Log.d(this, "Speaker is disabled. Can't update audio mode");
            return;
        }

        final TelecomAdapter telecomAdapter = TelecomAdapter.getInstance();
        final boolean isPrevAudioModeValid =
            sPreVideoAudioMode != AudioModeProvider.AUDIO_MODE_INVALID;

        Log.d(this, "Is previous audio mode valid = " + isPrevAudioModeValid + " enableSpeaker is "
            + enableSpeaker);

        // Set audio mode to previous mode if enableSpeaker is false.
        if (isPrevAudioModeValid && !enableSpeaker) {
            telecomAdapter.setAudioRoute(sPreVideoAudioMode);
            sPreVideoAudioMode = AudioModeProvider.AUDIO_MODE_INVALID;
            return;
        }

        int currentAudioMode = AudioModeProvider.getInstance().getAudioMode();

        // Set audio mode to speaker if enableSpeaker is true and bluetooth or headset are not
        // connected and it's a video call.
        if (!isAudioRouteEnabled(currentAudioMode,
            AudioState.ROUTE_BLUETOOTH | AudioState.ROUTE_WIRED_HEADSET) &&
            !isPrevAudioModeValid && enableSpeaker && CallUtils.isVideoCall(mPrimaryCall)) {
            sPreVideoAudioMode = currentAudioMode;

            Log.d(this, "Routing audio to speaker");
            telecomAdapter.setAudioRoute(AudioState.ROUTE_SPEAKER);
        }
    }

```

稍微注意一下我们就会发现，在最初的版本中，**updateAudioMode()传进来是ture的话，有可能会打开Speaker，但是传入false的话就一定不会打开Speaker**，记住这一点，后面实现一些需求的时候需要注意这点。
然后，updateAudioMode()只在三处被调用，（AS截图：）

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_updateAudioMode.jpg)
图1: updateAudioMode()被调用
并且只在enterVideoMode()的时候才传入true，至此我们知道：**只有在进入视频模式的时候才有可能打开Speaker。**

##What we want？
接下来说一下videoCall中不同操作的期望现象，这里我们不以现有代码中已经实现的功能推导现象（代码不够完善），而是以一种用户的角度去看待我做了某个动作以后希望得到什么现象。

###从语音电话升级到视频电话Speaker要打开
 
因为这个步骤会让call进入视频模式，执行enterVideoMode()方法，同时更新会执行updateAudioMode()方法更新audioMode，同样由视频电话降级为语音电话会退出视频模式执行exitVideoMode()，逻辑如下图：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_VO_to_VT.png)
图2: VoiceCall和VideoCall互相切换

从上图可以看到，每次进入视频模式的时候都会去updateAudioMode()的，我们希望的当然是它能走到`telecomAdapter.setAudioRoute(AudioState.ROUTE_SPEAKER);`这一步，执行到的话万事大吉，但是从代码中也可以看到，前面有一层层的判断条件，而假如不用顾及那么多的话，完全可以重写方法实现进入视频打开Speaker的功能。
而实际上上面那段代码的执行结果：**首次从语音升级到视频的时候Speaker是可以打开的，但如果降级在升级到视频就不会打开了**，为啥呢？isPrevAudioModeValid，这个变量产生了影响，非首次进入视频模式的情境下，这个变量有值了，导致代码不能执行到if里面对audioRoute的设置。但是轻易的去掉这个判断条件会引起其他问题（旋转屏幕会对Speaker产生影响）， 为了实现每次进入视频模式都打开Speaker，我在本地发起升级视频请求并成功升级和接受对方发来的升级到视频的请求中增加代码打开了Speaker。
 
###视频电话通话降级为语音通话后要把开启着的Speaker关掉

从图1和图2中我们可以看到只有从语音到视频的时候才会更新audio，反过来切换是不会更改audioRoute的，也就是说audio不会产生任何变化。
 
因此我们要自己实现这个功能。简单粗暴点的就是在exitVideoMode()的方法中调用TelecomAdapter中的setAudioRoute方法换成一个非Speaker的audioMode（这里是不能够用“关闭Speaker”这种方式的，只能是用一种新的的取代旧的），当然这里要考虑一下是否有外接音频设备，一般来说蓝牙的优先级是最高的。没有连接虑蓝牙的话，可以设置`TelecomAdapter.getInstance()..setAudioRoute(AudioState.ROUTE_WIRED_OR_EARPIECE);`。
 
再加一点条件，如果现在audioMode不是Speaker呢？那我们什么都不做就是了。
 
于是这一部分的逻辑流程大致如下图，下面蓝色大框里面是需要增加的代码。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_videoCall%20audio_expend.png)
图3: VT to VO close Speaker

###用户在hold当前视频通话之后在取消hold之后应还原之前的audioMode状态

这个操作执行示意图：

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_hold_unhold.png)
图4: hold/unhold
点击CallButton上的hold按钮会把call至于HOLD状态，之后call会退出视频模式（这里说明一下，其实是call先进入HOLD/ACTIVE状态，然后才exitVideoMode()/enterVideoMode()），把视频窗口都关掉，因为做了上面图3那种修改，在exitVideoMode中加了代码关闭Speaker，所以hold这一步操作也会关掉Speaker。并且在取消hold重新进入视频模式的时候会再次打开Speaker。这里看似符合期望现象，但是如果我们在hold之前Speaker不是开着的呢？实际上这已经造成了一种新的不和常理的现象：不管之前是audioMode是什么，只要是从HOLD的状态中回到ACTIVE，Speaker都会打开。
那么如何修改呢？我们可以采取“保存-恢复”的方法，即hold的时候保存一下当前的audioMode，在unhold的时候取出保存的audioMode并setAudioRoute重新设置一下。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_VT_HOLD_details.png)
图5: HOLD<->ACTIVE转换
保存audioMode的地方可以自己选，数据不会丢应该就可以。可以保存在Call里面，这样只要call存在，状态就不会丢。

###通话中收到新的视频来电不应该把Speaker打开

现象：
在通话中收到另一个视频电话来电，原本非开启状态的Speaker会切换为开启。
原因：
从现有代码中可以看到如下调用流程，可以看到在来电的时候会进入视频模式enterVideoMode(），从前面的分析我们知道enterVideoMode()会调用updateAudioMode(true)，于是Speaker被打开。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/audio_enterVideoMode.jpg)
图6: 来电更新AudioMode

解法：
在`telecomAdapter.setAudioRoute(AudioState.ROUTE_SPEAKER);`外面套一层条件：
```java
    if (state != Call.State.INCOMING && state != Call.State.CALL_WAITING) {
        Log.d(this, "Routing audio to speaker");
        telecomAdapter.setAudioRoute(AudioState.ROUTE_SPEAKER);
    }
```
在非来电情况下才执行到打开Speaker

---
关于旋转屏幕对audioMode的影响，暂时还没理透彻（直接用了QCOM PATCH），就先不写了。
暂时这么多。