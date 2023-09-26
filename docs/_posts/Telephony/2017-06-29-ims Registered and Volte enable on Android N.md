---
date: 2017-06-29 10:44
status: public
Tags:
  - ims
title: ims Registered and Volte enable on Android N
---

# 文前
代码基于mkm-mr1 android 7.1.2  
本篇记录了调查ims Registered状态和VoLTE enable状态的一些取值和更新的关键方法。  
初衷是调查ims registered状态和 VoLTE Available有没有对外提供接口，以及这两者有什么逻辑上的关系。  
PS：本文采取“双线叙事”，同时调查ims Registered和Volte enable的关键方法，如果读者感觉有点乱，可以一次只看ims或者volte（这意味着你要看两遍，其实是我懒不想分开写）。

# 正文
TelephonyManager.java向系统其他应用（Settings，SystemUI）提供接口，为什么说是“系统其他应用”呢？
因为以下几个方法都是@hide方法，不对第三方应用公开。


**fw/base/telephony**

TelephonyManager.java  
以下两个方法都是获得ims registered的方法，区别在于第二个方法需传入subId作为参数。

```java
   /**
    * Returns the IMS Registration Status
    * @hide
    */
   public boolean isImsRegistered() {
       try {
           ITelephony telephony = getITelephony();
           if (telephony == null)
               return false;
           return telephony.isImsRegistered();
       } catch (RemoteException ex) {
           return false;
       } catch (NullPointerException ex) {
           return false;
       }
   }
```
and...
```java

   /**
    * Returns the IMS Registration Status
    * using subId
    * @hide
    */
   public boolean isImsRegisteredForSubscriber(int subId) {
       try {
           ITelephony telephony = getITelephony();
           if (telephony == null)
               return false;
           return telephony.isImsRegisteredForSubscriber(subId);
       } catch (RemoteException ex) {
           return false;
       } catch (NullPointerException ex) {
           return false;
       }
   }
```

下面这个方法返回的是VoLTE status
```java
    /**
     * Returns the Status of Volte
     * @hide
     */
    public boolean isVolteAvailable() {
       try {
           return getITelephony().isVolteAvailable();
       } catch (RemoteException ex) {
           return false;
       } catch (NullPointerException ex) {
           return false;
       }
   }
```
到现在为止我么知道TelephonyManager.java中有三个hide方法返回ims registered和VoLTE Status，但都是hide方法，不对第三方应用公开。


**packages/services/Telephony**

PhoneInterfacemanager.java  
到了这可以看到，两个方法调用的都是Phone.java 里的registered()方法（当然，实际调用的是ImsPhone.java里的同名方法）。
```java
    @Override
    public boolean isImsRegistered() {
        return mPhone.registered();
    }


    /*
     * {@hide}
     * Returns the IMS Registration Status based on subId
     */
    public boolean isImsRegisteredForSubscriber(int subId) {
        final Phone phone = getPhone(subId);


        if (phone != null) {
            return phone.isImsRegistered();
        }
        return false;
    }
```
然后下面这个是VoLTE的，**isVolteAvailable和isVolteEnabled居然是等价的**，意味着开启即可用？
```java
    /*
     * {@hide}
     * Returns the IMS Registration Status
     */
    public boolean isVolteAvailable() {
        return mPhone.isVolteEnabled();
    }
```
然后观察到这个类里面还有个方法来设置ims registration status的，但是吧其实这个方法没有使用到。
```java
    public void setImsRegistrationState(boolean registered) {
        enforceModifyPermission();
        mPhone.setImsRegistrationState(registered);
    }
```
Phone.java
这个方法只在GsmCdmaPhone.java里面重写了，但是GsmCdmaPhone会有ims服务？
```java
    /**
     * Set IMS registration state
     */
    public void setImsRegistrationState(boolean registered) {
    }
```

## ImsRegistered的最终取值
**fw/opt/telephony**  
ImsPhone.java  
下面便是Ims registered取值的终点，接下来看的赋值（更新）的地方。  
setImsRegistered()正是给mImsRegistered 赋值的地方  
```java
    @Override
    public boolean isImsRegistered() {
        return mImsRegistered;
    }


    public void setImsRegistered(boolean value) {
        mImsRegistered = value;
    }
```
而Volte的取值还需再跟进一步
```java
    @Override
    public boolean isVolteEnabled() {
        return mCT.isVolteEnabled();
    }
```
## VolteEnabled的最终取值
**fw/opt/telephony**  

ImsPhoneCallTracker.java  
来自一个布尔值，其实是一个布尔数组里面的其中一个值。
```java
    public boolean isVolteEnabled() {
        return mImsFeatureEnabled[ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE];
    }
```
## Ims Registered 的更新
巧合的是ims registered的更新也是在这个类里面
```java
    /**
     * Listen to the IMS service state change
     *
     */
    private ImsConnectionStateListener mImsConnectionStateListener =
        new ImsConnectionStateListener() {
        @Override
        public void onImsConnected() {
            if (DBG) log("onImsConnected");
            mPhone.setServiceState(ServiceState.STATE_IN_SERVICE);
            mPhone.setImsRegistered(true);
            mMetrics.writeOnImsConnectionState(mPhone.getPhoneId(),
                    ImsConnectionState.State.CONNECTED, null);
        }


        @Override
        public void onImsDisconnected(ImsReasonInfo imsReasonInfo) {
            if (DBG) log("onImsDisconnected imsReasonInfo=" + imsReasonInfo);
            mPhone.setServiceState(ServiceState.STATE_OUT_OF_SERVICE);
            mPhone.setImsRegistered(false);
            mPhone.processDisconnectReason(imsReasonInfo);
            mMetrics.writeOnImsConnectionState(mPhone.getPhoneId(),
                    ImsConnectionState.State.DISCONNECTED, imsReasonInfo);
        }


        @Override
        public void onImsProgressing() {
            if (DBG) log("onImsProgressing");
            mPhone.setServiceState(ServiceState.STATE_OUT_OF_SERVICE);
            mPhone.setImsRegistered(false);
            mMetrics.writeOnImsConnectionState(mPhone.getPhoneId(),
                    ImsConnectionState.State.PROGRESSING, null);
        }
        ...
    }
```
上面的代码显示，在Ims连接后设置setImsRegistered 为true，在断开后和正在连接的时候都设置为false。


## VolteEnable 的更新 
**fw/opt/telephony**  

ImsPhoneCallTracker.java
Volte的更新也来自ImsConnectionStateListener，更新的是我们取值的那个布尔变量。
除此之外还有两个地方用到了ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE，**但是和这里无关。**

其一：

还是在ImsConnectionStateListener里面，
```java
    /**
     * Listen to the IMS service state change
     *
     */
    private ImsConnectionStateListener mImsConnectionStateListener =
        new ImsConnectionStateListener() {
        ...


        @Override
        public void onFeatureCapabilityChanged(int serviceClass,
                int[] enabledFeatures, int[] disabledFeatures) {
            if (serviceClass == ImsServiceClass.MMTEL) {
                boolean tmpIsVideoCallEnabled = isVideoCallEnabled();
                // Check enabledFeatures to determine capabilities. We ignore disabledFeatures.
                StringBuilder sb;
                if (DBG) {
                    sb = new StringBuilder(120);
                    sb.append("onFeatureCapabilityChanged: ");
                }
                for (int  i = ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE;
                        i <= ImsConfig.FeatureConstants.FEATURE_TYPE_UT_OVER_WIFI &&
                        i < enabledFeatures.length; i++) {
                    if (enabledFeatures[i] == i) {
                        // If the feature is set to its own integer value it is enabled.
                        if (DBG) {
                            sb.append(mImsFeatureStrings[i]);
                            sb.append(":true ");
                        }


                        mImsFeatureEnabled[i] = true;
                    } else if (enabledFeatures[i]
                            == ImsConfig.FeatureConstants.FEATURE_TYPE_UNKNOWN) {
                        // FEATURE_TYPE_UNKNOWN indicates that a feature is disabled.
                        if (DBG) {
                            sb.append(mImsFeatureStrings[i]);
                            sb.append(":false ");
                        }


                        mImsFeatureEnabled[i] = false;
```
这个方法重写自：
ImsConnectionStateListener.java
这个是跟IMS连接紧密相关的
```java
    /**
     * Called when its current IMS connection feature capability changes.
     */
    public void onFeatureCapabilityChanged(int serviceClass,
                int[] enabledFeatures, int[] disabledFeatures) {
        // no-op
    }
```
上面的方法调用自：
**fw/opt/net**  
ImsManager.java
```java
        @Override
        public void registrationFeatureCapabilityChanged(int serviceClass,
                int[] enabledFeatures, int[] disabledFeatures) {
            log("registrationFeatureCapabilityChanged :: serviceClass=" +
                    serviceClass);
            if (mListener != null) {
                mListener.onFeatureCapabilityChanged(serviceClass,
                        enabledFeatures, disabledFeatures);
            }
        }
```

后面两种是在ImsManager.java 这个类中的更新，更新的值保存在新建的ImsConfig对象中，值得注意的是，并没有哪块代码去getFeatureValue。如读者不关心这些可以直接跳到文末总结部分。

其二：

**fw/opt/net**  
ImsManager.java 
可以看到首先创建了一个ImsConfig对象，然后通过setFeatureValue()方法对ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE进行更新。
```java
    private void setLteFeatureValues(boolean turnOn) {
        log("setLteFeatureValues: " + turnOn);
        try {
            ImsConfig config = getConfigInterface();
            if (config != null) {
                config.setFeatureValue(ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE,
                        TelephonyManager.NETWORK_TYPE_LTE, turnOn ? 1 : 0, mImsConfigListener);


                if (isVtEnabledByPlatform(mContext)) {
                    boolean ignoreDataEnabledChanged = getBooleanCarrierConfig(mContext,
                            CarrierConfigManager.KEY_IGNORE_DATA_ENABLED_CHANGED_FOR_VIDEO_CALLS);
                    boolean enableViLte = turnOn && isVtEnabledByUser(mContext) &&
                            (ignoreDataEnabledChanged || isDataEnabled());
                    config.setFeatureValue(ImsConfig.FeatureConstants.FEATURE_TYPE_VIDEO_OVER_LTE,
                            TelephonyManager.NETWORK_TYPE_LTE,
                            enableViLte ? 1 : 0,
                            mImsConfigListener);
                }
            }
        } catch (ImsException e) {
            loge("setLteFeatureValues: exception ", e);
        }
    }
```
查看方法调用得知，是用户在界面上开启enhanced_4g_lte这个开关才更新的，我们推断这次更新只和用户设置有关系，和网络没有关系，但这似乎是不合理的。  
除了更新ImsConfig还更新了Settings 数据库中的存储。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/ims_manager.png)
 
其三：
```java
    /**
     * Update VoLTE config
     * @return whether feature is On
     * @throws ImsException
     */
    private boolean updateVolteFeatureValue() throws ImsException {
        boolean available = isVolteEnabledByPlatform(mContext);//平台支持
        boolean enabled = isEnhanced4gLteModeSettingEnabledByUser(mContext);//开关打开
        boolean isNonTty = isNonTtyOrTtyOnVolteEnabled(mContext);
        boolean isFeatureOn = available && enabled && isNonTty;


        log("updateVolteFeatureValue: available = " + available
                + ", enabled = " + enabled
                + ", nonTTY = " + isNonTty);


        getConfigInterface().setFeatureValue(
                ImsConfig.FeatureConstants.FEATURE_TYPE_VOICE_OVER_LTE,
                TelephonyManager.NETWORK_TYPE_LTE,
                isFeatureOn ?
                        ImsConfig.FeatureValueConstants.ON :
                        ImsConfig.FeatureValueConstants.OFF,
                mImsConfigListener);


        return isFeatureOn;
    }
```
查看方法调用我们推断，这次更新依赖于Ims网络服务。但是这个更新条件的Action是Action to broadcast when ImsService is up。这样看来这里似乎只有在IMS服务建立的时候才会执行。
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/ims-phone.png)
 其中Intent action
```java
    /**
     * Action to broadcast when ImsService is up.
     * Internal use only.
     * @hide
     */
    public static final String ACTION_IMS_SERVICE_UP =
            "com.android.ims.IMS_SERVICE_UP";
```

注意到下面的注释，看来以后是想把方法合并，从一个地方更新
 
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/ims-imanager.png)

整理流程：
![](https://codesimple-blog-images.oss-cn-hangzhou.aliyuncs.com/android/_image/ims-uml.png)
 


# 总结
ImsRegistered的取值来自ImsPhone.java的mImsRegistered，这个值取决于ims网络连接状态。在Telephony中的接口为hide方法，对第三方不可见。
VolteAvailable的取值来自ImsPhoneCallTracker.java的ImsConnectionStateListener，ImsConnectionStateListener监听onFeatureCapabilityChanged来更新（布尔值）。在Telephony中的接口为hide方法，同样对第三方不可见。
以上两者变化的关键方法都在ImsPhoneCallTracker.java的ImsConnectionStateListener里。