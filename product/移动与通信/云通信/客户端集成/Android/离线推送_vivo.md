# 消息推送

## 概述

IM 的终端用户需要随时都能够得知最新的消息，而由于移动端设备的性能与电量有限，当 APP 处于后台时，为了避免维持长连接而导致的过多资源消耗，云通信 IM 推荐您使用各厂商提供的系统级推送通道来进行消息通知，系统级推送通道的相比第三方推送拥有更稳定的系统级长连接，可以做到随时接受推送消息，且资源消耗大幅降低。

云通信 IM 目前已经支持了 APNs、小米推送、华为推送、魅族推送、VIVO 推送、OPPO 推送等厂商推送，具体如下：

| 推送通道  | 系统要求   | 条件说明                                                     |
| --------- | ---------- | ------------------------------------------------------------ |
| APNs      | iOS        | iOS 系统推送通道，也是唯一的 iOS 推送通道                    |
| 小米推送  | MIUI       | 使用小米推送 MiPush_SDK_Client_3_6_12.jar                    |
| 华为推送  | EMUI       | 华为移动服务版本 20401300 以上，SDK 版本 push:2.6.3.301      |
| 魅族推送  | Flyme      | 使用魅族推送 push-internal:3.6.+                             |
| vivo 推送 | FuntouchOS | 并非所有 vivo 机型和版本都支持使用 vivo 推送，SDK 版本 vivo_pushsdk_v2.3.1.jar |
| OPPO 推送 | ColorOS    | 并非所有 OPPO 机型和版本都支持使用 OPPO 推送。OPPO目前只有受邀开发者才能集成推送，因此 Demo 暂时没有 OPPO 推送的示例。 |

## 基本流程说明

为 APP 中 IM 功能实现消息推送的过程一般如下：

1. 开发者到厂商的平台注册账号，并通过开发者认证后，申请开通推送服务；
2. 创建推送服务，并绑定应用信息，获取推送证书、密码、密钥等信息；
3. 将推送证书及相关信息在 [云通信控制台](https://console.qcloud.com/avc) 上进行填写，云通信服务端会为每个证书生成不同的证书 ID；
4. 将厂商提供的推送 SDK 集成到开发者的项目工程中，并按各厂商的要求进行配置；
5. 集成云通信 IMSDK 到项目后，将证书 ID、设备信息等上报至云通信服务端；
6. 当客户端 APP 在 IM 没有退出登录的情况下，被系统或者用户 kill 时，云通信服务端将通过消息推送进行提醒。

## vivo 推送

vivo 手机使用深度定制 Android 系统，对于第三方 APP 自启动权限管理很严格，默认情况下第三方 APP 都不会在系统的自启动白名单内，APP 在后台时容易被系统 kill，因此推荐在 vivo 设备上集成 vivo 推送，vivo 推送 是 vivo 设备的系统级服务，推送到达率较高。目前，**云通信仅支持 vivo 推送的通知栏消息**。

> 【说明】
>
> - 此指引文档是直接参考 vivo 官方文档所写，若 vivo 推送有变动，以 [vivo 推送官网文档](https://dev.vivo.com.cn/documentCenter/doc/180) 为准。
> - 如果不需要对 vivo 设备做专门的离线推送适配，可以忽略此章节。

### Step 1 - 申请 vivo 推送证书

首先，您需要确保已在 [vivo 开放平台官网](https://dev.vivo.com.cn/home) 进行注册并通过开发者认证，认证过程需要 3天左右，请务必提前阅读一下  [vivo 推送服务说明](https://dev.vivo.com.cn/documentCenter/doc/180) ，以免影响您的接入进度。

在 vivo 开放平台的管理中心，选择**消息推送** - **创建**-**测试推送**，创建好您的魅族推送服务应用。当您创建好 vivo 推送服务应用后，在应用详情中，您可以看到您的应用信息，请记录下您的 **`主包名`**、**`AppID`**、**`AppSecret`**，稍后您需要将这些信息填写到云通信控制台。

![](https://main.qcloudimg.com/raw/bf297b7089f20c43c09d55b624631981.png)

### Step 2 - 托管证书信息到云通信

登录腾讯云 [云通信控制台](https://console.qcloud.com/avc) ，选择您的 IM 应用，进入应用配置，在基础配置中，先在应用平台配置项里**勾选 Android 平台并保存**。

![](https://main.qcloudimg.com/raw/bdbbbce31242bd8a917e3ecec9c3be88.jpg)

然后，在 **Android 推送证书** 中点击**添加证书**，填写您在 Step 1 中记录的 vivo 推送服务应用信息，证书信息填写后 10 分钟内生效。

- **推送平台**：选择 vivo
- **应用包名称**：填写您的 vivo 推送服务应用的 **主包名**
- **APPID**：填写您的 vivo 推送服务应用的 **AppID**
- **AppSecret**：填写您的 vivo 推送服务应用的 **AppSecret**

![](https://main.qcloudimg.com/raw/aec362779569a915d2f2a96d49ecf9bc.png)

正确填写相应信息后，点击 **确定** 保存。您将会看到控制台生成了一个推送证书信息，记录下这个 **`证书 ID`** ，在稍后的 SDK 集成过程中将证书 ID 与设备信息上报给云通信服务端。

> 如果您已有的证书信息有变更，可以选择编辑后进行修改更新。

![](https://main.qcloudimg.com/raw/4eb9be556cce525fc4e1a100846673d2.png)

### Step 3 - 推送 SDK 集成

> 【说明】
>
> - 云通信默认推送的通知标题为 `a new message`。
> - 阅读此小节前，请确保您已经正常集成并使用云通信 IMSDK。
> - 您可以在我们的 demo 里找到 vivo 推送的实现示例，请注意：vivo 推送版本更新时有可能会有功能调整，若您发现本节内容存在差异，烦请您及时查阅 [vivo 推送官网文档](https://dev.vivo.com.cn/documentCenter/doc/155)，并将文档信息差异反馈给我们，我们会及时跟进修改。

#### 3.1 下载 vivo 推送 SDK 并添加引用

集成 vivo 推送前，您需要先到 [vivo 推送运营平台](https://dev.vivo.com.cn/documentCenter/doc/158) 下载 vivo 推送 SDK包，下载完成后，解压，将 vivo 推送 SDK 目录下的相应的 `vivo_pushsdk_xxx.jar` 库文件添加到您项目的 `libs` 目录下，并且在项目中添加引用。

#### 3.2 配置 AndroidManifest.xml 文件

添加 vivo 推送服务需要的配置：

```xml
<!-- ********vivo推送设置start******** -->
<service
         android:name="com.vivo.push.sdk.service.CommandClientService"
         android:exported="true" />
<activity
          android:name="com.vivo.push.sdk.LinkProxyClientActivity"
          android:exported="false"
          android:screenOrientation="portrait"
          android:theme="@android:style/Theme.Translucent.NoTitleBar" />
<meta-data
           android:name="com.vivo.push.api_key"
           android:value="a90685ff-ebad-4df3-a265-3d4bb8e3a389" />
<meta-data
           android:name="com.vivo.push.app_id"
           android:value="11178" />
<!-- ********vivo推送设置end******** -->
<!--这里的 com.vivo.push.app_id ，com.vivo.push.api_key 由 vivo 开放平台生成 -->
```

#### 3.3 自定义一个 BroadcastReceiver 类

为了接收消息，您需要自定义一个继承自 `OpenClientPushMessageReceiver` 类的 BroadcastReceiver，并实现其中的 `onReceiveRegId`，`onNotificationMessageClicked` 方法，然后将此 receiver 注册到 AndroidManifest.xml 中。

以下为 Demo 中的示例代码：

```java
public class VIVOPushMessageReceiverImpl extends OpenClientPushMessageReceiver {
    private static final String TAG = "VIVOPushMessageReceiver";
    @Override
    public void onNotificationMessageClicked(Context context, UPSNotificationMessage upsNotificationMessage) {
        Log.i(TAG, "onNotificationMessageClicked");
    }

    @Override
    public void onReceiveRegId(Context context, String regId) {
        // vivo regId有变化会走这个回调。根据官网文档，获取regId需要在开启推送的回调里面调用PushClient.getInstance(getApplicationContext()).getRegId();参考LoginActivity
        Log.i(TAG, "onReceiveRegId = " + regId);
    }
}
```

将自定义的 BroadcastReceiver 注册到 AndroidManifest.xml：

```xml
<!--这里的 com.tencent.qcloud.uipojo.thirdpush.VIVOPushMessageReceiverImpl 修改成您 APP 中的完整类名-->
<!-- push应用定义消息receiver声明 -->
<receiver android:name="com.tencent.qcloud.uipojo.thirdpush.VIVOPushMessageReceiverImpl">
    <intent-filter>
        <!-- 接收push消息 -->
        <action android:name="com.vivo.pushclient.action.RECEIVE" />
    </intent-filter>
</receiver>
```

#### 3.4 在 APP 中注册 vivo 推送服务

如果您选择启用 vivo 离线推送，需要向 vivo 服务器注册推送服务，通过调用 `PushClient.getInstance(getApplicationContext()).initialize()` 来对 vivo 推送服务进行初始化。

注册成功后，需要在您的 APP 主界面中获取注册结果。其中 `regId` 即为当前设备上当前 APP 的唯一标识，这里需要把 `regId` 记录下来，后面需要上报给云通信服务端使用。

`PushClient.getInstance(getApplicationContext()).initialize()` 可在任意地方调用，为了提高注册成功率，vivo 官方建议在 Application 的 `onCreate` 中调用。

以下为 Demo 中的示例代码：

```java
public class PojoApplication extends Application {

    private static PojoApplication instance;

    @Override
    public void onCreate() {
        super.onCreate();
        //判断是否是在主线程
        if (SessionWrapper.isMainProcess(getApplicationContext())) {
            /**
             * TUIKit的初始化函数
             *
             * @param context  应用的上下文，一般为对应应用的ApplicationContext
             * @param sdkAppID 您在腾讯云注册应用时分配的sdkAppID
             * @param configs  TUIKit的相关配置项，一般使用默认即可，需特殊配置参考API文档
             */
            long current = System.currentTimeMillis();
            TUIKit.init(this, Constants.SDKAPPID, BaseUIKitConfigs.getDefaultConfigs());
            System.out.println(">>>>>>>>>>>>>>>>>>"+(System.currentTimeMillis()-current));
            //添加自定初始化配置
            customConfig();
            System.out.println(">>>>>>>>>>>>>>>>>>"+(System.currentTimeMillis()-current));

            if(IMFunc.isBrandXiaoMi()){
                // 小米离线推送
                MiPushClient.registerPush(this, Constants.XM_PUSH_APPID, Constants.XM_PUSH_APPKEY);
            }
            if(IMFunc.isBrandHuawei()){
                // 华为离线推送
                HMSAgent.init(this);
            }
            if(MzSystemUtils.isBrandMeizu(this)){
                // 魅族离线推送
                PushManager.register(this, Constants.MZ_PUSH_APPID, Constants.MZ_PUSH_APPKEY);
            }
            if(IMFunc.isBrandVivo()){
                // vivo离线推送
                PushClient.getInstance(getApplicationContext()).initialize();
            }
        }
        instance = this;
	}
}
```

在主界面中打开 vivo push服务：

```
if (IMFunc.isBrandVivo()) {
    // vivo离线推送
    PushClient.getInstance(getApplicationContext()).turnOnPush(new IPushActionListener() {
        @Override
        public void onStateChanged(int state) {
            if (state == 0) {
                String regId = PushClient.getInstance(getApplicationContext()).getRegId();
                QLog.i(TAG, "vivopush open vivo push success regId = " + regId);
                ThirdPushTokenMgr.getInstance().setThirdPushToken(regId);
                ThirdPushTokenMgr.getInstance().setPushTokenToTIM();
            } else {
                // 根据vivo推送文档说明，state = 101 表示该vivo机型或者版本不支持vivo推送，链接：https://dev.vivo.com.cn/documentCenter/doc/156
                QLog.i(TAG, "vivopush open vivo push fail state = " + state);
            }
        }
    });
```

### Step 4 - 推送信息上报云通信服务端

若您需要通过 vivo 推送进行 IM 消息的推送通知，必须在 **用户登录成功后** 将您托管到云通信控制台生成的 **证书 ID** 及 vivo 推送服务端生成的 **regId** 上报到云通信服务端，通过 `TIMManager` 中的 `setOfflinePushToken` 方法来进行上报。

> 【注意】
>
> 只有正确上报了 regId 与证书 ID 之后，云通信服务才能将用户与对应的设备信息绑定，从而使用 vivo 推送服务进行推送通知，请务必上报正确信息。

以下为 Demo 中的示例代码：

```java
/**
 * 我们先定义一些常量信息在 Constants.java
 */
/****** vivo离线推送参数start ******/
// 在腾讯云控制台上传第三方推送证书后分配的证书ID
public static final long VIVO_PUSH_BUZID = 6666;
// vivo开放平台分配的应用APPID及APPKEY
public static final String VIVO_PUSH_APPID = "1234512345123451234"; // 见清单文件
public static final String VIVO_PUSH_APPKEY = "12345abcde"; // 见清单文件
/****** vivo离线推送参数end ******/
```

```java
/**
 * 在 ThirdPushTokenMgr.java 中对推送的证书ID及设备信息进行上报操作
 */
public class ThirdPushTokenMgr {
    private static final String TAG = "ThirdPushTokenMgr";

    private String mThirdPushToken;

    private boolean mIsTokenSet = false;
    private boolean mIsLogin = false;

    public static ThirdPushTokenMgr getInstance () {
        return ThirdPushTokenHolder.instance;
    }

    private static class ThirdPushTokenHolder {
        private static final ThirdPushTokenMgr instance = new ThirdPushTokenMgr();
    }

    public void setIsLogin(boolean isLogin){
        mIsLogin = isLogin;
    }

    public String getThirdPushToken() {
        return mThirdPushToken;
    }

    public void setThirdPushToken(String mThirdPushToken) {
        this.mThirdPushToken = mThirdPushToken;  // regId 在此处传值，结合上文自定义 BroadcastReciever 类文档说明
    }

    public void setPushTokenToTIM(){
        if(mIsTokenSet){
            QLog.i(TAG, "setPushTokenToTIM mIsTokenSet true, ignore");
            return;
        }
        String token = ThirdPushTokenMgr.getInstance().getThirdPushToken();
        if(TextUtils.isEmpty(token)){
            QLog.i(TAG, "setPushTokenToTIM third token is empty");
            mIsTokenSet = false;
            return;
        }
        if( !mIsLogin ){
            QLog.i(TAG, "setPushTokenToTIM not login, ignore");
            return;
        }
        TIMOfflinePushToken param = null;
        if(IMFunc.isBrandXiaoMi()){     //判断厂商品牌，根据不同厂商选择不同的推送服务
            param = new TIMOfflinePushToken(Constants.XM_PUSH_BUZID, token);
        }else if(IMFunc.isBrandHuawei()){
            param = new TIMOfflinePushToken(Constants.HW_PUSH_BUZID, token);
        }else if(IMFunc.isBrandMeizu()){
            param = new TIMOfflinePushToken(Constants.MZ_PUSH_BUZID, token);
        }else if(IMFunc.isBrandOppo()){
            param = new TIMOfflinePushToken(Constants.OPPO_PUSH_BUZID, token);
        }else if(IMFunc.isBrandVivo()){
            param = new TIMOfflinePushToken(Constants.VIVO_PUSH_BUZID, token);
        }else{
            return;
        }
        TIMManager.getInstance().setOfflinePushToken(param, new TIMCallBack() {
            @Override
            public void onError(int code, String desc) {
                Log.d(TAG, "setOfflinePushToken err code = " + code);
            }

            @Override
            public void onSuccess() {
                Log.d(TAG, "setOfflinePushToken success");
                mIsTokenSet = true;
            }
        });
    }
}
```

### Step 5 - 离线推送

上报证书 ID 及 regId 成功后，云通信服务端会在该设备上的 IM 用户 logout 之前、APP 被 kill 之后，将消息通过 vivo 推送通知到用户端。

> 【说明】
>
> - vivo 推送只支持部分 vivo 手机，详情见 [vivo 推送常见问题汇总]( https://dev.vivo.com.cn/documentCenter/doc/156)。
> - vivo 推送的到达率并不是 100% 必达。
> - vivo 推送可能会有一些延时，原因与 APP 被 kill 的时机有关，部分情况下与 vivo 推送服务有关。
> - 若 IM 用户已经 logout 或被云通信服务端主动下线，如在其他端登录被踢等情况，则该设备上不会再收到消息推送。

### 常见问题

#### 混淆配置

如果您的应用使用了混淆，为了防止 vivo 离线推送功能异常，您需要 keep 自定义的 BroadcastReceiver，参考添加以下混淆规则，请注意直接使用下面的代码是不行的。

```
# 请将 com.tencent.qcloud.uipojo.thirdpus.VIVOPushMessageReceiverImpl 改成您 APP 中定义的完整类名
#vivo推送
-dontwarn com.vivo.push.**
-keep class com.vivo.push.**{*; }
-keep class com.vivo.vms.**{*; }
-keep class com.tencent.qcloud.uipojo.thirdpus.VIVOPushMessageReceiverImpl{*;}
```

#### 推送提示音配置

目前 vivo 推送不支持自定义的提示音。

#### 推送收不到如何排查

1. 首先，我们需要清楚，任何推送都不是 100% 必达，厂商推送也不例外，因此，若是在快速、连续的推送过程中偶现一至两条推送未通知提醒，通常原因是厂商推送频控的限制引起。
2. 按照推送的流程，确认 vivo 推送证书信息在 [云通信控制台](https://console.qcloud.com/avc) 正确填写；
3. 确认您的项目 [集成 vivo 推送 SDK](#Step&nbsp;3&nbsp;-&nbsp;推送&nbsp;SDK&nbsp;集成) 的配置正确，并正常获取到了 RegID；
4. 确认您已将正确的 [推送信息上报](#Step&nbsp;4&nbsp;-&nbsp;推送信息上报云通信服务端) 至云通信服务端；
5. 在设备中手动 kill 掉 APP，发送若干条消息，确认是否能在一分钟内接收到通知；
6. 若通过上述步骤后仍然接收不到推送，可以将您的问题 `时间点`、`SDKAPPID`、`证书 ID`、`接收推送的 userid` 提交工单处理。