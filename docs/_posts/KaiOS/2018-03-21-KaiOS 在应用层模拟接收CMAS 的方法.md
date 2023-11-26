---
date: 2018-03-21 21:53
status: public
title: 'KaiOS 在应用层模拟接收CMAS 的方法'
---


# 需求
一般来说测试CMAS 都需要去实验室用Agilent 8960 仪器连接手机天线发送消息测试的。
但是去实验室需要至少3样东西，白卡、射频线和门禁卡。这3样东西我一个都没有、、而且就算去测试的话也比较花时间。
所以为了省时省力省心，我写了一段在应用层模拟接收CMAS 消息的代码。

# 模拟接收
在模拟接收之前，我们需要先了解一下CMAS 消息的接收流程是怎样的，这样才好到合适的位置和方法模拟。

在之前的接收CMAS 消息流程中我们已经知道，gecko层是通过“广播”把广播消息传给上层的。
android/gecko/dom/system/gonk/RILSystemMessenger.jsm
```javascript
  /**
   * Wrapper to send 'cellbroadcast-received' system message.
   */
  notifyCbMessageReceived: function(aServiceId, aGsmGeographicalScope, aMessageCode,
                                    aMessageId, aLanguage, aBody, aMessageClass,
                                    aTimestamp, aCdmaServiceCategory, aHasEtwsInfo,
                                    aEtwsWarningType, aEtwsEmergencyUserAlert, aEtwsPopup) {
    // Align the same layout to MozCellBroadcastMessage
    let data = {
      serviceId: aServiceId,
      gsmGeographicalScope: this._convertCbGsmGeographicalScope(aGsmGeographicalScope),
      messageCode: aMessageCode,
      messageId: aMessageId,
      language: aLanguage,
      body: aBody,
      messageClass: this._convertCbMessageClass(aMessageClass),
      timestamp: aTimestamp,
      cdmaServiceCategory: null,
      etws: null
    };
    if (aHasEtwsInfo) {
      data.etws = {
        warningType: this._convertCbEtwsWarningType(aEtwsWarningType),
        emergencyUserAlert: aEtwsEmergencyUserAlert,
        popup: aEtwsPopup
      };
    }
    if (aCdmaServiceCategory !=
        Ci.nsICellBroadcastService.CDMA_SERVICE_CATEGORY_INVALID) {
      data.cdmaServiceCategory = aCdmaServiceCategory;
    }
    this.broadcastMessage("cellbroadcast-received", data); // 发送广播
  },
```
上层apps 在manifest.webapp 里注册了cellbroadcast-received，而且在cmas_alert_startup.js

的构造方法监听所以它能接收到消息。
manifest.webapp
```xml
"messages": [
    { "cellbroadcast-received": "/index.html"} 
  ],
```
cmas_alert_startup.js

```javascript
  index() {
    window.navigator.mozSetMessageHandler('cellbroadcast-received',
      this.onCellbroadcast.bind(this)); // 
  }

  }
  onCellbroadcast(message) {
    this.debug(`onCellbroadcast message -> ${JSON.stringify(message)}`);
    if (!Utils.isEmergencyAlert(message)) {
      this.hide();
    }
    this.checkMessage(message);
  }
```
所以我们要实现的代码需要满足至少要满足下面条件中的一个：
1. 构建一个CMAS 消息，通过broadcastMessage的形式发送出来；
2. 构建一个CMAS 消息，之后调用network-alert 的onCellbroadcast，并把CMAS 消息作为参数传进去；

其实做到了第一条第二条就自动满足了，而且第一条是相对来说比较合理的方法，这种方法不需修改network-alert 内部代码。但是你可能需要想一下怎么触发broadcastMessage，也就是写在哪里比较合适。我没有用这个方法，读者在用这个方法之前可能需要注意下broadcastMessage 是不是可以直接使用。

第二个方法，首先我们仿造上面的代码构造一个假的CMAS 消息，然后在构建完之后，调用  this.onCellbroadcast(message) 模拟消息被接收到：
```javascript
fakeCMAS() {
  dump('_cmas fake cmas --1');
  let message = {
    "serviceId":0,
    "gsmGeographicalScope":"cell",
    "messageCode":1,
    "messageId":4379,
    "language":"en",
    "body":"CMAS_AMBER_A",
    "messageClass":"normal",
    "timestamp":5266606,
    "cdmaServiceCategory":0,
    "etws":null
  };
  dump('_cmas fake cmas --2');
  this.onCellbroadcast(message);
  dump('_cmas fake cmas --3');
}
```
下面要考虑的就是何时调用这个fakeCMAS()，我的方法是在构造函数内写了一个计时器循环调用fakeCMAS()这个方法。
```javascript
  constructor(props) { 
    super(props);

...
+    var self = this;
+    setInterval(function(){
+         dump('_cmas setInterval');
+         self.fakeCMAS();
+       }, 33000);
    document.body.classList.toggle('large-text', navigator.largeTextEnabled);
    this.index();
  }
```
但是这种在应用层模拟接收CMAS 消息有一个局限性，就是没法测试“重复消息检测机制”，因为这个功能是在modern 实现的（应用层也实现了，但目前可以说是“废的”），我们可以在QXDM log中看到有关重复消息检测的信息。

# 效果
测试效果还可以，每隔33s会收到相同的CMAS 消息。因为应用层拿不到serialNumber，所以每次判断结果都是非重复消息。
但是读者应当结合实际情况具体分析，有些不适合模拟的，或者没把握的还是要去实验室实际测试，保险为准。
