## Unity平台集成指南

### 快速集成

#### 导入SDK
　　在 Unity 编译器中选择 Assets --> Import Package --> Custom Package 找到本地目录下的 `TalkingData_AppAnalytics_SDK.unitypackage` 文件，点击`Open`按钮即可导入成功。

#### 添加权限和依赖的框架
###### Android
　　Android平台中SDK需要获取适当的权限才可以正常工作，开发者需要在 `AndroidManifest.xml` 文件中添加下列所有权限申明。

* `INTERNET` 允许应用联网和发送统计数据的权限。
* `ACCESS_NETWORK_STATE` 允许应用检测网络连接状态，在网络异常状态下避免数据发送，节省流量和电量。
* `READ_PHONE_STATE` 允许应用以只读的方式访问手机设备的信息, 通过获取的信息来定位唯一的玩家。
* `ACCESS_WIFI_STATE` 用来获取设备的 mac 地址。
* `WRITE_EXTERNAL_STORAGE` 用于保存设备信息，以及记录日志。
* `GET_TASKS(可选)` 获取当前应用是否在显示应用，可以更精准的统计用户活跃。
* `ACCESS_FINE_LOCATION(可选)` 用来获取该应用被使用的精确位置信息。
* `ACCESS_COARSE_LOCATION(可选)` 用来获取该应用被使用的粗略位置信息。

###### iOS
　　iOS平台需要添加适当的依赖框架才可以正常工作，开发者需要在编译生成的 `Xcode` 工程中添加下列所有依赖框架。

* `AdSupport.framework` 获取advertisingIdentifier
* `CFNetwork.framework` 发送统计数据
* `CoreTelephony.framework` 获取运营商标识
* `CoreMotion.framework` 支持摇一摇功能
* `Security.framework` 辅助存储设备标识
* `SystemConfiguration.framework` 检测网络状况
* `libc++.tbd` 支持最新的c++11标准
* `libz.tbd` 进行数据压缩

###### WindowsPhone
　　Windows Phone平台中，SDK需要获取适当的权限才可以正常工作，开发者需要在 `WMAppManifest.xml` 文件中添加下列所有权限申明。

* `ID_CAP_IDENTITY_DEVICE` 用于唯一标识一台设备。
* `ID_CAP_IDENTITY_USER` 用于唯一标识一台设备。
* `ID_CAP_LOCATION(可选)` 允许应用访问位置，用于统计用户的区域来源。
* `ID_CAP_NETWORKING` 允许应用联网和发送统计数据的权限。

#### 初始化集成
　　在应用启动后（包含应用刚开启或从后台恢复到前台），调用 `TalkingDataPlugin.SessionStarted`，该接口完成统计模块的初始化和统计 session 的创建，所以越早调用越好。  
　　在应用结束时（包含切出应用和退到后台情况，如点击home、锁屏等按键），调用 `TalkingDataPlugin.SessionStoped`。  
　　可以选择性的在channelId中填入推广渠道的名称，数据报表中则可单独查询到他们的数据。每台设备仅记录首次安装激活的渠道，更替渠道包安装不会重复计量。不同的渠道ID包请重新编译打包。

#### 高级功能
###### 页面统计
　　统计页面的点击次数和停留时间，需要在页面打开和关闭的时候添加对应的API调用。

```
//在进入页面时调用
TalkingDataPlugin.TrackPageBegin("page_name");
//在离开页面时调用
TalkingDataPlugin.TrackPageEnd("page_name");
```
　　注：page_name是自定义的页面名称，注意不要加空格或其他的转义字符。

###### 位置信息统计
　　对于iOS平台，TalkingData默认不统计用户的位置信息，只提供了记录接口，请根据苹果公司的审核原则合理使用用户的位置信息，如有需要，可以将已获取的位置信息提交到TalkingData统计服务器，服务器只保存最近一次提交的位置信息。

```
TalkingDataPlugin.SetLocation(纬度, 经度);
```

###### 使用自定义事件
　　自定义事件用于统计任何您期望去跟踪的数据，同时自定义事件还支持添加一些描述性的属性参数，使用多对Key-Value的方式来进行发送（非必须使用），用来对事件发生时的状况做详尽分析。

　　在应用程序要跟踪的事件处加入下面格式的代码，也就成功的添加了一个简单事件到您的应用程序中了：

```
TalkingDataPlugin.TrackEvent("Event_ID");
```

　　跟踪多个同类型事件，无需定义多个Event ID，可以使用Event ID做为目录名，而使用Label标签来区分这些事件，可按照下面格式添加代码：

```
TalkingDataPlugin.TrackEventWithLabel("Event_ID", "Event_Label");
```

　　为事件添加详尽的描述信息，可以更有效的对事件触发的条件和场景做分析，可按照下面格式添加代码：

```
TalkingDataPlugin.TrackEventWithParameters("Event_ID","Event_Label", Dictionary<string, object="">);
```
　　注: 此Dictionary的Value仅支持字符串（string）和数字（number）类型。在Value使用string格式时，报表中将给出事件发生时每种value出现的频次；Value为number时，报表将帮助计算value值的总计值和平均数。

### 推送营销
　　营销推送组件帮助您获得利用数据进行精准推送的能力，结合数据平台提供的各种人群，可以实时编辑发送内容，对任意人群完成推送，并支持实时查阅推送效果数据，不断对比效果，优化营销方法。  
　　营销推送更强大之处在于，您不仅可以选择使用TalkingData提供的推送通道，他还可以与个推、极光等推送平台组合使用，让以往的粗放推送都可达到实时精准化，并实时查阅效果数据。

#### Android
##### 集成TalkingData推送
　　增加以下的调用方法，来让您的应用可以接收推送通知；需注意在获得推送能力的同时，您的应用会在后台中长期运行。

###### 修改AndroidManifest.xml文件
1.添加推送功能所必要的权限。

```
<!-- 允许App开机启动，来接收推送 -->
<uses-permission android:name="android.permission.RECEIVE_BOOT_COMPLETED"></uses-permission>
<!--发送持久广播 -->
<uses-permission android:name="android.permission.BROADCAST_STICKY"></uses-permission>
<!-- 修改全局系统设置-->
<uses-permission android:name="android.permission.WRITE_SETTINGS"></uses-permission>
<!-- 允许振动，在接收推送时提示客户 -->
<uses-permission android:name="android.permission.VIBRATE"></uses-permission>
<!-- 侦测Wifi 变化，以针对不同 Wifi 控制最佳心跳频率，确保push的通道稳定 -->
<uses-permission android:name="android.permission.CHANGE_WIFI_STATE"></uses-permission>
<!-- 此权限用于在接到推送时，可唤醒屏幕，可选择性添加权限 -->
<uses-permission android:name="android.permission.WAKE_LOCK"></uses-permission>
```

2.添加TalkingData所必须的Service。

```
<service android:name="com.apptalkingdata.push.service.PushService" android:process=":push" android:exported="true"></service>
```

3.添加TalkingData所必须的BroadCastReceiver，以支持接收消息。

```
<receiver android:name="com.apptalkingdata.push.service.PushServiceReceiver" android:exported="true">
	<intent-filter>
		<action android:name="android.intent.action.CMD"></action>
		<action android:name="android.talkingdata.action.notification.SHOW"></action>
		<action android:name="android.talkingdata.action.media.MESSAGE"></action>
		<action android:name="android.intent.action.BOOT_COMPLETED"></action>
		<action android:name="android.net.conn.CONNECTIVITY_CHANGE"></action>
		<action android:name="android.intent.action.USER_PRESENT"></action>
	</intent-filter>
</receiver>
<receiver android:name="com.tendcloud.tenddata.TalkingDataAppMessageReceiver" android:enabled="true">
	<intent-filter>
		<action android:name="android.talkingdata.action.media.SILENT"></action>
		<action android:name="android.talkingdata.action.media.TD.TOKEN"></action>
	</intent-filter>
	<intent-filter>
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
</receiver>
```

##### 第三方推送平台与TalkingData组合使用
　　TalkingData支持从平台中划定精准用户群，组合第三方推送平台来直接发送推送和收集实时数据效果；目前支持两家第三方推送平台：个推、极光。如果您已经是三方推送的使用者，这种方式能让您更舒服的实现精准推送并验证效果。

###### 集成第三方推送平台
　　您需要在第三方推送平台中申请账户，并已完成三方推送的对接。 您需要在对接后确认测试通过，三方推送可以接收到推送消息。

###### 添加TalkingData Game Analytics所必须的BroadCastReceiver
1.个推推送

```
<receiver android:name="com.tendcloud.tenddata.TalkingDataAppMessageReceiver" android:enabled="true">
	<intent-filter>
		<!-- 必须添加 -->
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
	<intent-filter>
		<!--注：H0DPYSxUkR9NFoWnvff656要换成开发者自己的appid-->
		<action android:name="com.igexin.sdk.action.H0DPYSxUkR9NFoWnvff656"></action>
	</intent-filter>
</receiver>
```

2.极光推送

```
<receiver android:name="com.tendcloud.tenddata.TalkingDataAppMessageReceiver" android:enabled="true">
	<intent-filter>
		<!-- 必须添加 -->
		<action android:name="com.talkingdata.notification.click"></action>
		<action android:name="com.talkingdata.message.click"></action>
	</intent-filter>
	<intent-filter>
		<!-- 如果使用极光推送，必须添加 -->
		<action android:name="cn.jpush.android.intent.REGISTRATION"></action>
		<action android:name="cn.jpush.android.intent.MESSAGE_RECEIVED"></action>
		<category android:name="您的应用包名"></category>
	</intent-filter>
</receiver>
```

###### 配置推送key
　　完成以上集成后，您还需要在TalkingData平台中配置推送相关的Key；请进入平台“推送营销”-“推送配置”在android平台配置中添加好这些配置。

#### iOS
##### 在Start方法中注册通知服务
```
void Start () {
	TalkingDataPlugin.SessionStarted("your_app_id", "your_channel_id");
#if UNITY_IPHONE
#if UNITY_5
	UnityEngine.iOS.NotificationServices.RegisterForNotifications(
		UnityEngine.iOS.NotificationType.Alert |
		UnityEngine.iOS.NotificationType.Badge |
		UnityEngine.iOS.NotificationType.Sound);
#else
	NotificationServices.RegisterForRemoteNotificationTypes(
		RemoteNotificationType.Alert |
		RemoteNotificationType.Badge |
		RemoteNotificationType.Sound);
#endif
#endif
	// other code
}
```

##### 在Update方法中调用TalkingData的方法
```
void Update () {
#if UNITY_IPHONE
	TalkingDataPlugin.SetDeviceToken();
	TalkingDataPlugin.HandlePushMessage();
#endif
	// other code
}
```

### 易认证
##### 易认证简介
　　易认证提供稳定的手机短信认证服务，简易集成，即可获得从认证码下发到安全验证的全部能力。

　　这些场景下您可选择使用易认证功能，我们提供多达10000条/日免费认证短信量：  
　　A、服务型的App，要求用户使用手机号来注册的情况。  
　　B、任何的App，可建议用户做手机号绑定，加强联络用户的能力。  
　　C、提供忘记密码功能，利用短信验证来作为临时登录方式。

##### 易认证功能开通
　　您需要首先在统计分析平台中创建好一款产品（或是以前已在使用统计的产品），找到易认证功能标签，点击立即开通，即可为这款产品开通易认证功能。

　　在功能成功开通后，您会获得集成所必需的标识码：  
　　app ID : 与统计分析的app ID相同，用来唯一标识您的该款App。  
　　secret ID：客户端集成时需要传入的安全码，用于避免他人仿冒您的身份来发送短信。  
　　server security：我们提供了最高安全级别的服务器端校验短信认证结果的方式，此标识码用于服务器方式的身份安全验证。

##### 易认证初始化
　　启用易认证功能，您需要主动调用易认证初始化方法，这与统计分析初始化不同，需要单独调用。

```
void Start () {
	TalkingDataSMSPlugin.Init("your_app_id", "your_secret_id");
}
```

##### 定义易认证的回调方法
```
void OnApplySucc(string requestId) {
	Debug.Log ("OnApplySucc:" + requestId);
}

void OnApplyFailed(int errorCode, string errorMessage) {
	Debug.Log ("OnApplyFailed:" + errorCode + " " + errorMessage);
}

void OnVerifySucc(string requestId) {
	Debug.Log ("OnVerifySucc:" + requestId);
}

void OnVerifyFailed(int errorCode, string errorMessage) {
	Debug.Log ("OnVerifyFailed:" + errorCode + " " + errorMessage);
}
```

##### 发送短信认证码调用方法
　　从您的应用向我们的SDK发起请求，申请发送一条认证码短信。成功受理的发送请求，SDK会通知服务器，经由运营商通道直至短信下发。请注意以下限制，对单个App：

* 每个手机号，60S内不可多次请求短信  
* 每个手机号，10分钟内最多请求3条认证短信  
* 每个手机号，24小时内最多请求5条认证短信  
* 每个设备，24小时内最多请求5条认证短信

```
// 在需要发送短信认证码时调用此接口
TalkingDataSMSPlugin.ApplyAuthCode("country_code", "mobile", this.OnApplySucc, this.OnApplyFailed);
// 在需要重新发送短信认证码时调用此接口
TalkingDataSMSPlugin.ReapplyAuthCode("country_code", "mobile", "request_id", this.OnApplySucc, this.OnApplyFailed);
```

##### 认证码校验调用方法
　　从您的应用向我们的SDK发起校验请求，我们会快速验证手机号与认证码是否匹配，并将结果返回给您的应用。

　　认证码的有效期为30分钟，如果超过了验证时间还没有成功验证，将会失效。

```
TalkingDataSMSPlugin.VerifyAuthCode ("country_code", "mobile", "auth_code", this.OnVerifySucc, this.OnVerifyFailed);
```

##### requestId特别说明
　　您在进行集成时应该会注意到，我们的每次成功请求和成功验证时，都会返回一个requestId，这个Id有很多妙用值得您的注意。

　　在请求发送短信认证码时，携带requestId能很好的处理“重发”的问题，重发状况下用户不会收到多条不一样的认证码，就不会出现校验时不知道该填写哪条认证码的苦恼。  
　　校验成功时，我们返回requestId，用于帮助您区分一个手机号上同时并发的多次短信验证。这样多种不同安全级别的场景下的短信认证都可以同时进行，切不会搞乱，而引发安全问题。
