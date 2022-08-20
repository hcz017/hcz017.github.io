---
date:  2017-03-28 19:19
title:  '【译】Quick Settings Tiles on Android 7.0'
---

译者：为了避免读者对tile，Tile，TileService分不清，这里简单做个解释。  
tile：人类语言，中文就是“图块”，英文就是“tile”，是指我们看到的快速设置面板那些单个的开关；  
Tile：“图块”在android系统中的名字，你可以把它当成是指一个“控件”，源码中是有Tile.java这个类的；  
TileService：介于Tile和system之间的服务，使Tile工作。  
非逐字翻译，略有“本土”优化。

-----

用户可以打开app是在2008年。早期的Android中就提供了小部件和通知，即使app没有打开，小部件和通知提供的额外面板也能用来显示重要的开关和信息。在新的Android 7.0（API 24）中，任何应用都可以创建**快速设置图块（tile）**，以便快速访问通知托盘上可用功能的重要信息。

除了一直显示app提供的信息到界面之外，点击一个tile还可以出发后台工作，打开dialog，甚至打开一个activity。

# 怎样的tile才是好tile

决定是否放一个quick setting tile一般考虑两个条件，**紧迫性和高频性**。显然同时具有紧迫性和高频性的tile是最好的，一个紧迫性较高但使用频率没那么高的tile仍被视为有价值的快速设置tile。因为频率高低对不同用户来说可能截然不同（一个用户可能一天用很多次Google Cast，而另一个用户可能一周才用一次），决定什么条件使快速设置tile成为较好的tile的时候，**优先考虑紧迫性和重要性**（包括信息显示和动作设置）。

![trans-qs-tile.jpg](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/trans-qs-tile.png)

因为快速设置tile可以持久的和你的应用交互，因此应保证你的tile在你的应用安装以后是长期有用处的。这意味着把类似于一次性的“设置”任务写成tiles是不合适的。

# 创建TileService
每一个tile都和一个TileService关联，而TileService正是系统和tile的交互，推送更新到tile的方式。
和其他Service一样，你必须在在AndroidManifest里添加声明：
```xml
<service
  android:name=".AwesomeTileService"
  android:icon="@drawable/ic_tile_default"
  android:label="@string/tile_name"
  android:permission="android.permission.BIND_QUICK_SETTINGS_TILE">
  <intent-filter>
    <action
      android:name="android.service.quicksettings.action.QS_TILE"/>
  </intent-filter>
</service>
```
需要注意一点 **androd:icon和android:label 是TileService中非常关键的部分**。当用户选择tiles添加到快速设置面时，看到的正是这两个属性的值。图标和标签应当告诉用户这个tile实际上是干什么的--使用和app一样的图标和标签一般吸引不到什么用户（译者：我翻译的时候不禁怀疑是不是翻错了）。

![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/trans-qs-tile-label.png)

理想情况下，选择标签应当尽量短（小于18字符不然会超出显示范围），图标应选择透明背景的纯白矢量图（只有透明度可以调整）。请记住，tile只能用在API 24+的版本上，所以这正是你避免使用png图像的好时机（译者：这是叫你尽量用矢量图）。

# TileService的生命周期
TileService是一个绑定的服务，因此其生命周期主要由Android系统控制。 TileService有三个阶段：被添加，监听和被删除。

## 第一阶段：
第一次是用户添加tile到快速设置面板的时候。此时系统会绑定你的TileService，然后调用onTileAdded()，这个地方是适合做初始化工作。

## 第二阶段
添加tile之后，在每一次tile可见的时候都会回调onStartListening()，这时候一般期望更新tile，注册其他需要的listners。然后每次不可见的时候回调 onStopListening()。

## 第三阶段
最后一个阶段，也是我不希望看到的阶段，是移除tile，回调 onTileRemoved()（译者：在回调onTileRemoved()之前还会回调一次onStopListening()）。此时，你的应用应该停下所有和tile相关的工作，直到tile再次被添加到快速设置面板（也许移除是误操作？我们希望如此吧）。

有一点很重要，在tile不可见的时候TileService几乎肯定会被解绑（destroyed），不要以为你写的Service会在 onStartListening()/onStopListening()之外继续存在。

# TileSerivce另外的life：active mode（激活状态）
默认当tile可见的时候才会绑定你的TielService，但是如果你准确的知道什么时候需要更新tile（基于后台数据同步），强烈推荐你把TileService设置成active状态，添加META_DATA_ACTIVE_TILE到manifest里面：
```xml
<service
  android:name=".AwesomeActiveTileService"
  android:icon="@drawable/ic_tile_default"
  android:label="@string/tile_name"
  android:permission="android.permission.BIND_QUICK_SETTINGS_TILE">
  <intent-filter>
    <action
      android:name="android.service.quicksettings.action.QS_TILE"/>
  </intent-filter>
  <meta-data
    android:name="android.service.quicksettings.ACTIVE_TILE"
    android:value="true" />
</service>
```
在active模式下，TileService仍将会绑定到onTileAdded()和onTileRemoved() (还有点击事件)。然而惟一一次调用到onStartListening()的时候是在调用静态方法TileService.requestListeningState() 之后。然后在回调onStopListening()之前你都可以准确的更新tile。这提供了一种简单的one-shot的能力，在数据变化的时候准确地去更新tile，无论此时tile是否可见。

由于active的tile不必在每次可见的时候绑定，**因此active tiles对系统健康运行是有好处的**。构建active模式的tiles意味着每次快速设置面板可见时，系统需要绑定的进程更少了。（当然，系统已经根据可用内存限制了快速设置可以绑定的TieleService的数量，例如，但到那时候已经快要接近内存抖动（memory thrashing）了，那你不想达到的情况。）

# 更新tile
介于onStartListening() 和 onStopListening()之间时，你可以更新tile的UI。每个TileService都有一个单独的Tile来表示tile的UI，可以调用getQsTile()获得Tile。

tile界面的主要组成部分包括图标（icon），标签（label）和内容描述（content description）（辅助服务/无障碍功能）。图标使用的是Marshmallow中引入的Icon类。这表示你的图标可以创建自Bitmap,，content URI， byte array， file path或者通过静态createWith方法获得的资源。

UI的另外组成部分是tile的**状态（state）**，一个tile可以是以下三种状态：
- STATE_ACTIVE “开”或者“启用”状态
- STATE_INACTIVE “关”或者“禁用”状态
- STATE_UNAVAILABLE 禁用点击事件

根据tile的状态，tile会自动着色（准确的着色将有制作者指定，但是STATE_ACTIVE 要显示的最突出，STATE_INACTIVE 次之，STATE_UNAVAILABLE 一点都不突出）。

最重要的是：你必须调用updateTile()来更新tile。这是系统解析所有你更新到Tile中的数据并刷新UI的提示。
```java
@Override
public void onStartListening() {
  Tile tile = getQsTile();
  tile.setIcon(Icon.createWithResource(this,
    R.drawable.ic_title_started));
  tile.setLabel(getString(R.string.tile_label));
  tile.setContentDescription(
    getString(R.string.tile_content_description);
  tile.setState(Tile.STATE_ACTIVE);
  tile.updateTile();
}
```
记住，tile可以在安全锁定的设备的锁屏界面之上显示。如果有敏感信息，考虑检查isSecure()的值，以此确定设备是否安全。
# 点击事件的处理
当然，除非是不可用的tile，不然用户都可以点击tile触发一个事件。当onClick()被回调的时候，你有许多选择。最明显的一个是在后台工作。要记住，onClick()工作在UI线程，所以你要把复杂的工作移到另一个线程（或者Service-比如IntentService）。

然而，快速设置图块（tile）也有几个机制来显示UI。第一个是通过showDialog()，这个方法允许tile显示一个dialog。 只有少数几种情况dialog显得特别有意义/必要：
- 如果你的动作需要额外输入信息（比如从一组选项中选择）
- 如果你的动作影响了其他设备——确保用户知道他们将要改变其他设备
- 如果你的动作需要特定用户同意（比如通过流量下载体积很大的东西，或者上传用户还没有同意的数据）

基本上，**dialog 是要添加context到你的动作中**。你能做的最差的事情是给用户一个简单的方式搬起石头砸自己的脚。

如果你谨记快速设置图块（tile）“关键+高频”的特点，你可能会觉得用startActivityAndCollapse()来启动activity会有点怪。如果每次用户点击的时候都要启动activity那确实是有点怪，但是如果作为点击dialog中“更多...”选项的反应的时候则十分有用，或者你有一个独特的专门适用于设备锁屏之后的UI的时候也是十分有用的（记住：你有一个启动图标，点击tile的时候别仅仅启动（应用））。

**在已锁屏设备上使用的时候有一些限制**。当isLocked()返回true的时候，你不能显示dialog，activity则必须设置flag  FLAG_SHOW_WHEN_LOCKED才能在锁屏界面之上显示。unlockAndRun()这个方法可用来提示用户解锁他们的设备，在用户解锁设备之后才允许运行代码（比如显示dialog）。

Note:你会发现有些系统tile有另外的自定义的UI会覆盖掉整个快速设置面板。不幸的是，现在第三方还不能这样做。

快速设置tile上的长点击事件默认会跳转到“应用信息”界面。开发者可以通过添加值为ACTION_QS_TILE_PREFERENCES的`<intent-filter>`到activity来覆盖这个跳转。

# 指尖上的便利
快速设置tile给了开发者一个全新的和用户交互的界面，并且允许开发者最热心的用户快速访问关键和高频使用的选项。
在开发过程中一定要留些额外的时间去实际使用tile几天：确保在app的整个生命周期中，tile都是长期有用和直观的。

原文链接： [Quick Settings Tiles on Android 7.0](https://medium.com/google-developers/quick-settings-tiles-e3c22daf93a8#.nywjn7kjz)