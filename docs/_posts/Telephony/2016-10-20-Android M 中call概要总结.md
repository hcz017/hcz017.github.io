---
date: 2016-10-20 21:53
status: public
tags: Call
title: 'Android M 中Call 的概要总结（目录结构/界面组成/call状态转化上报/常见log分析）'
---

主要内容：
1. Call涉及的目录结构及框架结构
2. InCallUI层的基本架构（所涉及的Presenter、Fragment及Activity）
3. Call的几种状态（对应phone状态）及上报流程
4. GSM call与IMS MO流程的差异
5. 分析问题的常用log

希望你在看完本篇以后能够：

1. 快速找到Call界面某部分内容对应的fragment及presenter
2. 结合log定位当前call的状态

# 1. Call涉及的目录结构及框架结构

## 1.1 目录结构

packages/**apps**/Dialer/
packages/**apps**/InCallUI

packages/**services**/Telecomm
packages/**services**/Telephony

**framework**/base/telecomm
**framework**/opt/telephony
（vendor/…/ims   Ims Call）

**Dialer**
拨打电话的入口，来电不会经过Dialer。但是拨打电话的出口不光是Dialer，在联系人和短信里也有拨打电话的出口。代码运行在dialer进程。

**InCallUI**
负责显示通话界面的信息，来电信息。dialer进程。

**Telecomm**
处理Intent，发送广播，设置call的状态，audio状态。system_process和telecomm:ui进程。

**Telephony**
向下层传递拨号，注册了很多广播，申请很多权限（service data sms wap network）。 phone进程

**telecomm** 
提供placeCall的接口(自android M开始)，创建outgoingCall的connection，通知上层成功建立connection

**telephony**
拨号 也就是dial命令的下发，但是如果是Ims网络就会有下面一步

**Vendor/.../ims**
创建ImsConnection，ImsCall，拨号。phone进程。

## 1.2 框架结构

这只是框架上的一个大致结构

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_telephony_struc.png)

在实际的流程中并不一定是自上而下或者自下而上的，也有可能跳过某个模块直接传递信息。
比如在Dialer拨号的时候，就是直接调用framework/base/telecmm中TelecomManager的placeCall接口拨打电话。

# 2. InCallUI层的基本架构（所涉及的Presenter、Fragment及Activity）

## 2.1 InCallUI内有多个对应的Presenter和Fragment

Presenter和Fragment的关系：Fragment直接控制界面上的控件，并处理一些简单的逻辑，对应的Presenter处理稍复杂的逻辑。

CallCardFragment
CallCarfPresenter
    CallButtonFragment
    CallButtonPresenter
CallCard和CallButton是绑定在一起显示的，有CallCard就一定会有CallButton
CallCard显示拨打的电话的信息，包括联系人姓名，号码，通话时长，通话类型等（稍后有展示），CallButton包括可以对当前通话进行的操作，如通话保持，通话静音，通话录音等。

VideoCallFragment
VideoCallPresenter
主要是视频内容，在进入退出视频模式时同时处理camera、audio相关信息。

AnswerFragment
AnswerPresenter
显示新来电，或者视频升级请求，重点在于对不同类型的通话显示不同的选项（也可根据运营商进行不同配置）。

DialpadFragment
DialpadPresenter
拨号盘，添加通话的时候显示。一般没什么问题，之前有该显不显的问题，同时dialpad的显示会影响endCallButton的大小，两者是同步变化的。

ConferenceManagerFragment
ConferenceMangerPresenter
没遇到过这里的问题，只是准备PPT的时候才看到这个。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_ConferenceManagerFragment.png)
InCallActivity启动的时候，在显示完动画后，调用showCallCardFragment()显示CallCardFragment（CallCardFragment本身也有动画）然后Fragment内new Presenter。
其他几个也有各自的显示流程，不一定是在InCallActivity的动画显示后立刻show出来。
另：**VideoCallFragment不是**通过showFragment()显示的。

## 2.2 通话界面布局分析 

注：界面被轻量定制，并非和原生完全一样。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_InCallUI.png)
图1 CallCardFragment和CallButtonFragment

图中除了priamry_card_info.xml和call_button_fragment.xml是布局文件，其他标注均为控件id。
AnswerFragment和DialpadFragment共用一个FramLayout
@+id/answer_and_dialpad_container
在显示的时候覆盖在CallCard的上层，比如CallCard中的photo（联系人头像）就是被dialpad给覆盖。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_VideoCall.png)
图2 VideoCallFragment（真人出境啦）

左侧为普通视频电话，右侧为视频会议电话。
**注意**，虽然右侧看起来像是有4个小窗口显示内容，但实际上和左边一样只有两个控件显示视频，只不过右侧上方的**视频内容**被分成了3块。我们把黑色的背景色调成蓝色，就容易区分出是两个控件了。
至于为什么两张图incomingVideo的位置不一样，CallCard一个显示一个没显示，是因为，点击屏幕（或者等5s）后CallCard会隐藏，然后incomingVideo会自动调整位置到中央。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_answer.png)
图3 AnswerFragment

更多的界面就不一一详细说明了。
如果你看到了不常见的控件，那它在代码中的位置，和临近位置的控件的代码位置也是临近的（有点拗口= =）。
 

## 2.3 InCallPresenter和InCallActivity
这个为什么要单独列成一条呢？因为这个Presenter比前面几个Presenter都重要，它连结这界面的各个部分。
**InCallPresenter**
从CallList中获取更新然后通知到InCallActivity，由InCallActivity控制界面显示。
同时各个Fragment和Presenter可以获得InCallPresenter的实例，进而更新/获得数据。单独的Presenter负责对应的界面的逻辑，InCallPresenter负责整个界面的显示，协调各个Fragment/Presenter之间的工作。
以视频电话全屏为例，在VideoCallPresenter中有隐藏CallCard进入全屏模式的需求，在VideoCallPresenter中会通过InCallPresenter调用CallCardPresenter的onFullscreenModeChanged()方法，进而调用CallCardFragment的方法隐藏CallCard
**InCallActivity**
InCallActivity是InCallUI的主体，可以说所有的类都是围绕着或者是为了这个Activity工作的，各个Fragment附着在InCallActivity上，InCallPresenter通过InCallActivity提供的get***Fragmment的方法，获得具体Fragment对象， 更新界面。
Fragment是通过InCallActivity创建和显示的，但是各个Presenter和InCallPresenter之间的交互**不一定全都经过InCallActivity**（比如可以通过listener调用）
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_InCallUI_struc.png)
图4 InCallUI基本架构

我们举两个例子 来说明界面上更新走过的流程：
1.设置CallStateLabel上的文本，比如"requesting video"，代码的执行经过InCallPresenter> CallCardPresenter> CallCardFragment，跳过了InCallActivity；
2.弹出dialog，比如“网络不可用” ，代码执行经过IncallPresenter > InCallActivity

还有另外一个比较重要的没有体现在图上的类叫做**TelecomAdapter。**
它的职责主要是把界面上操作的命令向下层传递，具体点就是把对call的操作传给Telecomm，包括接听，挂断，拒接，静音，切换audioRoute等。

# 3. Call的几种状态（对应phone状态）及上报流程

## 3.1 Call的状态

下面这张图不用记，知道有这么多就行了。
不同模块的call的状态不一样，有些能对应上，有些是独有的对应不上的。
比较重要的是InCallUI里的**Call.State**和InCallPresenter的内部类**InCallState**
InCallPresenter控制着界面显示，这里的InCallState就是界面显示的依据。
TelephonyManager向外提供访问手机状态的接口，只有三种状态。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_Call_State.png)
图 5 Call的状态

下图表现的是InCallUI Call状态的转化以及对应的TelephonyManager.CALL_STATE_***状态
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_call_state_change.jpg)
 图6 InCallUI 中的Call状态转化

其中CONNECTING/SELECT_PHONE_ACCOUNT这两个是**二者取其一**。如果是双卡手机并设置每次询问，则拨号后的状态为SELECT_PHONE_ACCOUNT，如果有是单卡或者默认某一张卡拨打，则状态为CONNECTING，并且这个状态很短暂，很快就转换为DIALING。
 

## 3.2 上报流程
都是以UNSOL_RESPONSE_CALL_STATE_CHANGED消息上报为开始，
//CS
```xml
10:52:42.093 D/RILJ    ( 4017): [UNSL]< UNSOL_RESPONSE_CALL_STATE_CHANGED [SUB1]
```

/PS
```xml
D/ImsSenderRxr( 4499): [UNSL]< UNSOL_RESPONSE_CALL_STATE_CHANGED [id=1,DIALING,toa=129,norm,mo,0,voc,noevp,,cli=1,,3Call Details = 3 2 callSubState 0 videoPauseState2 mediaId0 Local Ability Peer Ability Cause code 0,CallFailCause Code= 501,CallFailCause String= null, ECT mask: 0] [SUB0]
```

流程中多次用到RegistrantList消息处理机制，比较关键的一个地方是TelephonyConnection里，在外拨（或者来电）设置连接的时候注册了一个消息
getPhone().registerForPreciseCallStateChanged(mHandler, MSG_PRECISE_CALL_STATE_CHANGED, null);当收到这个消息的时候会调用updateState()更新状态。
updateState
在Telephony更新call的状态的时候不同的状态对应不同的方法，如setActiveInternal()
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/call_ims_call_state_update.png)
  图7 ims call state update（简图）

上图也印证了之前说的，虽然层次上packages叠在framework上面，但实际上他们并不是严格按照相邻的顺序去调用。有可能隔着一层就调用了，也可能反复调用。

下图以IMS电话接通状态上报为例
略
 图8 ims call state update（详图）

# 4. GSM call与IMS MO流程的差异

- 最大的区别是 IMS通话最后是通过高通私有代码ims.app中拨出去的。
- IMS MO也要经过GSMPhone，判断到IMS网络可用以后会调用到ImsPhone中的dial()方法。
- ImsSenderRxr.java相当于CS call中的RIL.java，DIAL的命令也是在此发送

     *07-18 15:35:04.977 D/ImsSenderRxr( 4499): [0247]> DIAL[SUB0]*

- ImsPhone中的log是在radio log中打印，但是ImsSenderRxr的log是在main log打印。
- 在ImsPhone的后续步骤中，会创建ImsPhoneConnection和ImsCall。

对比图
略
 图9 GSM MO和IMS MO 对比

箭头指向为对应步骤，可以很明显的看到Ims的MO流程在这部分更复杂一些。针对IMS网络下的通话，专门建立了ImsPhoneConnection ImsCall。

# 5. 分析问题的常用log
5.1 InCallUI
前面说过InCallUI的界面显示是以InCallState的状态为根据的，而查看InCallState的关键字是Phone switching state：

```xml
14:42:29.721 I/InCall (24960): InCallPresenter - Phone switching state: NO_CALLS -> NO_CALLS
14:42:29.864 I/InCall (24960): InCallPresenter - Phone switching state: OUTGOING -> OUTGOING
14:42:30.262 I/InCall (24960): InCallPresenter - Phone switching state: OUTGOING -> PENDING_OUTGOING
14:42:31.194 I/InCall (24960): InCallPresenter - Phone switching state: PENDING_OUTGOING -> OUTGOING
14:42:34.975 I/InCall (24960): InCallPresenter - Phone switching state: OUTGOING -> INCALL
14:42:35.630 I/InCall (24960): InCallPresenter - Phone switching state: INCALL -> INCALL
```

InCallUI会根据这个状态显示界面信息。
这个状态一般是跟CallList里的call同步，但也有例外，就是会**撤回状态**，关键字Undo the state change
Log.i(this, "Undo the state change: " + newState + " -> " + mInCallState);
14:42:35.998 I/InCall (24960): InCallPresenter – Undo the state change: INCOMING ->NO_CALLS
这时候就出现了界面显示和实际情况不一样，上面的log就对应来电没有界面。

注意：INCALL包括DISCONNECTING和DISCONNECTED，虽然这两个是短暂的状态，但是看到的最后几个INCALL状态可能call已经断开了。

CallList中的onUpdate()，这是常见的看InCallUI中call的数量和状态的地方，关键字CallList - onUpdate 

```xml
17:13:21.514 I/InCall ( 4166): CallList - onUpdate - [Call_0, CONNECTING, [Capabilities:
17:13:21.568 I/InCall ( 4166): CallList - onUpdate - [Call_0, DIALING, [Capabilities: CAP
17:13:29.997 I/InCall ( 4166): CallList - onUpdate - [Call_0, ACTIVE, [Capabilities: CAPA
17:13:30.255 I/InCall ( 4166): CallList - onUpdate - [Call_0, ACTIVE, [Capabilities: CAPA
17:13:39.896 I/InCall ( 4166): CallList - onUpdate - [Call_1, CONNECTING, [Capabilities:]
17:13:43.179 I/InCall ( 4166): CallList - onUpdate - [Call_1, CONNECTING, [Capabilities:
17:13:43.266 I/InCall ( 4166): CallList - onUpdate - [Call_1, DIALING, [Capabilities: CA
17:13:43.451 I/InCall ( 4166): CallList - onUpdate - [Call_1, DIALING, [Capabilities: CA
17:13:43.471 I/InCall ( 4166): CallList - onUpdate - [Call_0, ONHOLD, [Capabilities: CAP
17:13:43.492 I/InCall ( 4166): CallList - onUpdate - [Call_0, ONHOLD, [Capabilities: CAP
```

5.2 Telecom

另外自android 6.0开始，Telecomm加了一个关键字为Event 的log，会打印在Telecom内执行的关键步骤的log。打印出来如下

```xml
17:22:05.385 I/Telecom ( 769): Event: Call 8: CREATED, null
17:22:05.394 I/Telecom ( 769): Event: Call 8: SET_CONNECTING, ComponentInfo{com.android.phone/com.android.services.telephony.TelephonyConnectionService},
17:22:05.449 I/Telecom ( 769): Event: Call 8: AUDIO_ROUTE, EARPIECE
17:22:09.161 I/Telecom ( 769): Event: Call 8: BIND_CS, ComponentInfo{com.android.phone…
17:22:09.240 I/Telecom ( 769): Event: Call 8: CS_BOUND, ComponentInfo{com.android.phone…
17:22:09.241 I/Telecom ( 769): Event: Call 8: START_CONNECTION, tel:*****
17:22:09.399 I/Telecom ( 769): Event: Call 8: SET_DIALING, successful outgoing call
17:22:17.395 I/Telecom ( 769): Event: Call 8: SET_ACTIVE, active set explicitly
17:23:03.636 I/Telecom ( 769): Event: Call 8: REQUEST_DISCONNECT, null
17:23:04.211 I/Telecom ( 769): Event: Call 8: SET_DISCONNECTED, disconnected set explicitly> DisconnectCause [ Code: (LOCAL) Label: () Description: () Reason: (LOCAL) Tone: (27) ]
17:23:05.612 I/Telecom ( 769): Event: Call 8: DESTROYED, null
```

**DisconnectCause **是常用的查看通话断开原因的log。 

注意：Telecom的log是在system_process进程打印，用adb命令抓log的时候要加上-b system

5.3 MT 来电

在main log中我常看

```xml
00:01:21.154 D/Telephony( 1389): PstnIncomingCallNotifier: handleNewRingingConnection
```

如果这条看不到还可以查CallsManager中的setCallState，可以从状态转化中看到来电状态。

```xml
14:42:34.888 I/Telecom ( 2920): CallsManager: setCallState DIALING -> ACTIVE, call: [100747014, DIALING,
```

5.4 MO DIAL

IMS(PS)

```xml
18:30:35.325 D/ImsSenderRxr( 3325): [0056]> DIAL[SUB0]
18:30:35.365 D/ImsSenderRxr( 3325): [0056]< DIAL [SUB0
```

有DIAL回传才表示拨号成功

GSM(CS)

```xml
17:29:56.206 D/RILJ    ( 4017): [6301]> DIAL [SUB1]
17:29:56.250 D/RILJ    ( 4017): [6301]< DIAL  [SUB1]
```

5.5 HANGUP

```xml
15:32:37.899 D/ImsSenderRxr( 4499): [0240]> HANGUP[SUB0]
15:32:37.995 D/ImsSenderRxr( 4499): [0240]< HANGUP [SUB0]
```

5.6 IMS

在IMS网络下通话的很多log是在main log中打印的，其中几个比较关键的类包括ImsSenderRxr、ImsServiceSub，以这两个类名为关键字在log中搜索就好了，或者以关键字/ImsSe把这两个一起搜出来

```xml
18:30:37.522 D/ImsSenderRxr( 3325): [UNSL]< UNSOL_RESPONSE_CALL_STATE_CHANGED [id=1,DIALING,toa=129,norm,mo,0,voc,noevp,,cli=1,,3Call Details = 3 2 callSubState 0 videoPauseState2 mediaId5 Local Ability Peer Ability Cause code 0,CallFailCause Code= 0,CallFailCause String= null, ECT mask: 0] [SUB0]
18:30:37.522 D/ImsServiceSub( 3325): Message received: what = 1
18:30:37.522 D/ImsServiceSub( 3325): >handleCalls
18:30:37.523 D/ImsServiceSub( 3325): handleCalls: dc = id=1,DIALING,toa=129,norm,mo,0,voc,noevp,,cli=1,,3Call Details = 3 2 callSubState 0 videoPauseState2 mediaId5 Local Ability Peer Ability Cause code 0,CallFailCause Code= 0,CallFailCause String= null, ECT mask: 0
```