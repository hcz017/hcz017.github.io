---
date: 2018-02-24 14:55
status: public
title: 'Wireless Emergency Alerts 介绍（CMAS介绍）'
---

# 前言
首先明确一下标题的含义，Wireless Emergency Alerts 无线紧急警报，它的前身是大家经常听到的CMAS（Commercial mobile alert system）。

先看一个事件：夏威夷误发“导弹预警”  
当地时间13日上午8时7分左右，夏威夷部分居民手机收到了政府部门“夏威夷应急管理中心”（Hawaii Emergency Management Agency）推送的一条 “导弹（Ballistic Missile）预警”通知。
Emergency alerts example.jpg

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-sample.png)

所以直观上讲，WEA 可以使民众可以快速收到人身安全收到威胁的警报。

那么WEA 具体的含义，接收条件，警报表现形式又是怎么样的呢？接下来我们一一介绍。

# 什么是WEA?
WEA 即Wireless Emergency Alerts 的缩写(之前称为Commercial Mobile Alert System (CMAS)  
WEA 是一个安全系统，使特定区域用户的手机/其他无线移动设备可以到其安全将收到威胁的警报。  
WEA 2008年建立，2012年运行。  
WEA 使政府可以对特定区域发送紧急警报。  

# WEA 信息由谁发布？
经授权的州或本地政府可以使用WEA发出关于公共安全紧急情况的警报，例如由于恶劣天气，恐怖主义威胁或化学品泄漏而发生的疏散令或避难点。

警报由政府官员发送给运营商，之后由运营商推送给用户的移动设备（当然这部分肯定做到了自动化/程序化）。
Wea offical to users.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-2-roles.png)

# 谁能够接收WEA？
该区域所有具有WEA接收功能的移动设备都能免费接收到警报，即使设备是漫游到此地。
WEA Area.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-3-devices.png)

# WEA 发送哪些警报？
WEA警报只涉及关键紧急情况。用户可能只会收到三种类型警报：  
1. 总统发出的警报
2. 涉及迫在眉睫的安全或生命威胁的警报
3. 安帕警报（安泊警报）

运营商或许会允许用户阻止除总统警报以外的警报。也就是说总统发出的警报强制接收。

PS：这里其实有一点“争议”。根据AT&T 的测试case，多语言开关会控制该语言下所有警报的接收情况。如关闭西班牙语言警报接收开关后，西班牙语总统级别的警报也不会接收显示。现在能肯定的是英语的总统级别的警报是绝对不能被拒绝接收的。

# 用户接收到WEA 会观察到什么？
WEA 警报以类似于文字消息的形式显示，内容包含的url，email 和phoneNumber 应当被高亮并可执行访问或拨打的操作。  
警报伴有独特的声音信号和震动频率，这对听力或视力有障碍的残疾人尤其有帮助。

补充：  
根据世界无线通讯解决方案联盟 (ATIS)与美国通信工业协会 (TIA)制定的 J-STD-100, 移动设备运行技术规范《JOINT ATIS/TIA CMASMOBILE DEVICE BEHAVIOR SPECIFICATION》中的规定。震动和警报音应满足下面的要求
Common Audio Attention Signal.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-4-signal.png)

震动的间隔也是这样的，就不贴图了。  
两种模式的时间模式不需要同步，具体的要求可以查看协议要求。  
上面介绍了一些用户层面上可体验到的东西，下面我们从代码/程序层面在介绍一下。  

# WEA 服务初始化（手机端）
WEA 基于小区广播服务。所以它的初始化基本上等同与CellBroadcast的初始化，它独有的地方在于要配置WEA 相关的mids（Message ID）或者说Channel ID 到modem。
而Modem端的mids来源于代码，SIM卡，NV值等
1. 代码中根据类型设置是否默认开启（预置）；
2. SIM 卡某一位置写了CBMI(Cell Broadcast Message ID)或者CBMIR（Cell Broadcast Message ID Range）；
3. NV 中某一段存了起始到结束的mids范围和size


比较好的方法是修改代码调整预置的mids。
CB intial.png
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-5-flow.png)

# Alerts 分类（根据mids）
综合查到的信息做一个整理

|WEA Message Class	|Message ID（English）|	Message ID（Spanish）|	PWS Type|	Description|
|------------------|-------------------|-----------------|-----------------|--------------|
|Presidential Alert|    4370|   4383|	CMAS/CBS|	Presidential Level Alerts|
|Extreme Threat|    4371|   4384|	CMAS/CBS|	Extreme Alerts with Severity of Extreme, Urgency of Immediate and Certainty of Observed|
|Extreme Threat|    4372|	4385|	CMAS/CBS|	Extreme Alerts with Severity of Extreme, Urgency of Immediate and Certainty of Likely|
|Severe Threat |    4373|	4386|	CMAS/CBS|	Severe Alerts with Severity of Extreme, Urgency of Immediate and Certainty of Observed|
|Severe Threat |    4374|	4387|	CMAS/CBS|	Severe Alerts with Severity of Extreme, Urgency of Immediate and Certainty of Likely|
|Severe Threat |    4375|	4388|	CMAS/CBS|	Severe Alerts with Severity of Severe, Urgency of Immediate and Certainty of Observed|
|Severe Threat |    4376|	4389|	CMAS/CBS|	Severe Alerts with Severity of Severe, Urgency of Immediate and Certainty of Likely|
|Severe Threat |    4377|	4390|	CMAS/CBS|	Severe Alerts with Severity of Severe, Urgency of Expected and Certainty of Observed|
|Severe Threat |    4378|	4391|	CMAS/CBS|	Severe Alerts with Severity of Severe, Urgency of Expected and Certainty of Likely|
AMBER Alert    |    4379|	4392|	CMAS/CBS|	Child Abduction Emergency (or Amber Alert)|
RMT Alert      |    4380|	4393|	CMAS/CBS|	Required Monthly Test|
Exercise Alert |    4381|	4394|	CMAS/CBS|	CMAS Exercise|
Operator Alert/CMSP	|4382|	4395|	CMAS/CBS|	Operator defined use|

其中English 的Presidential Alert 默认开启切不允许关闭，Extreme Threat, Severe Threat, AMBER Alert默认开启，最后下面3类作为测试用默认关闭。

# AT命令设置/查看支持的mids

## 测试命令 AT+CSCB=?
返回：  
```
+CSCB:(list of supported <mode>s)
```

参数：  
```
<mode> 0 接收消息 1 不接收消息
```

## 读取命令 AT+CSCB?
这个命令用于快速查看当前支持的mids

返回：  
```
+CSCB: \<mode>,\<mids>,\<dcss>
```

参数：  
```
<mode> 0 接收消息 1 不接收消息  
<mids> 文本类型，是CMB message ids 的组合  
<dcss> 文本类型，是CMB message ids 支持的语言，为空的话表示接收所有语言编码的消息  
```
比如：  
```
+CSCB: 0, "4352-4355, 4370,4371-4373,4373-4379, 4383,...",""
```

## 写入命令 AT+CSCB=[< mode>[,<mids>[,dcss]]]
用这个命令设置新的mids方便临时测试。  
例如：
```
AT+CSCB=0,”15-17,50,86”,” ”
```

# WEA 消息接收流程
前面我们提到了WEA 是基于小区广播服务的，那它的接收也是和小区广播一样的。
WEA receive.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-6-recive-flow.png)

可以说底层没有区分消息是否是WEA 消息，它的流程和普通广播消息是一样的，到应用层才根据mids做了区分。
下面介绍两个影响用户是否能看到消息的条件

# 重复消息检测机制
根据3GPP 协议，应当对收到的消息进行重复消息检测（即使先收到的消息已被删除）
1. 如果messageId 和 serialNumber 相同则为重复消息
2. 消息内容是否一致（可选，ckt在7050上实现）

通常是对24小时内收到的消息判断，日本为1小时，不过T-Mobile 的表格里询问的却是是否对12小时内的消息检测。
WEA duplication detection.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-7-3ggp.png)

前面括号里写了，即使之前收到的消息已经被删除，也应作为和新消息比较的数据源。那么可能就要求开发者在接收到消息后在不同位置存储两份，一份用来显示，一份用来做重复消息检测（KaiOS是这么做的）。当然，方案不唯一。

# 语言开关
下图摘自AT&T 的文档
WEA language suppor.png

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/Telephony/_image/wea-8-att.png)

语言开关不会真正影响接收（mids）只会影响显示。就是说消息可能收到了并且存储了，只是没有显示给用户看。  
如果mids已经存在，真正影响消息接收的是AT+CSCB=[< mode>[,<mids>[,dcss]]] 最后的dcss