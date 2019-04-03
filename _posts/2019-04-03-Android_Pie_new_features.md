---
layout: post
title: "Android Pie(9.0) 新特性和适配策略"
subtitle: "Android Pie(9.0) New Features and Adaptation Strategy"
author: "beforenight"
header-img: "img/post-bg-android.jpg"
header-mask: 0.4
tags:
  - Android
  - Pie
  - 适配
---

## Android Pie(9.0) New Features
1. 刘海屏适配
2. 通知功能的变更
3. 隐私权变更
4. 对使用非 SDK 接口的限制 和 适配策略
5. 非Activity-Context启动Activity
6. Apache HTTP 客户端弃用，影响采用非标准 ClassLoader 的应用
7. 前台服务

## Android API Differences Report
> This report details the changes in the core Android framework API between two API Level specifications. It shows additions, modifications, and removals for packages, classes, methods, and fields. The report also includes general statistics that characterize the extent and type of the differences.

> This report is based a comparison of the Android API specifications whose API Level identifiers are given in the upper-right corner of this page. It compares a newer "to" API to an older "from" API, noting all changes relative to the older API. So, for example, API elements marked as removed are no longer present in the "to" API specification.

> To navigate the report, use the "Select a Diffs Index" and "Filter the Index" controls on the left. The report uses text formatting to indicate interface names, links to reference documentation, and links to change description. The statistics are accessible from the "Statistics" link in the upper-right corner.

> For more information about the Android framework API and SDK, see the [Android Developers site](https://developer.android.com/index.html).

### 刘海屏适配 Display Cutout Support
Android 9 支持最新的全面屏，其中包含为摄像头和扬声器预留空间的屏幕缺口。 通过 **DisplayCutout** 类可确定非功能区域的位置和形状，这些区域不应显示内容。 要确定这些屏幕缺口区域是否存在及其位置，使用 **getDisplayCutout()** 函数。


```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
    View decorView = getWindow().getDecorView();
    WindowInsets rootWindowInsets = decorView.getRootWindowInsets();
    if (rootWindowInsets != null) {
        DisplayCutout cutout = rootWindowInsets.getDisplayCutout();
        List<Rect> boundingRects = cutout.getBoundingRects();
        if (boundingRects != null && boundingRects.size() > 0) {
            String msg;
            for (Rect rect : boundingRects) {
                msg = s+"left-" + rect.left;
                Log.d(TAG, msg);
            }
         }
    }
}
```

用新的窗口布局属性 layoutInDisplayCutoutMode 为设备屏幕缺口周围的内容进行布局。 可以将此属性设为下列值之一：

**LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT**
> The window is allowed to extend into the DisplayCutout area, only if the DisplayCutout is fully contained within a system bar. Otherwise, the window is laid out such that it does not overlap with the DisplayCutout area.

**LAYOUT_IN_DISPLAY_CUTOUT_MODE_SHORT_EDGES**
> The window is always allowed to extend into the DisplayCutout areas on the short edges of the screen. The window will never extend into a DisplayCutout area on the long edges of the screen.屏幕短边有cutout，会延伸过去；若cutout在长边，一定不会延伸过去。

> In this mode, the window extends under cutouts on the short edge of the display in both portrait and landscape, regardless of whether the window is hiding the system bars
On the other hand, should the cutout be on the long edge of the display, a letterbox will be applied such that the window does not extend into the cutout on either long edge在长边有cutout的情况，会排出在外。

**LAYOUT_IN_DISPLAY_CUTOUT_MODE_NEVER**
> The window is never allowed to overlap with the DisplayCutout area.
This should be used with windows that transiently set View.SYSTEM_UI_FLAG_FULLSCREEN or View.SYSTEM_UI_FLAG_HIDE_NAVIGATION to avoid a relayout of the window when the respective flag is set or cleared.


```
WindowManager.LayoutParams lp = getWindow().getAttributes();
lp.layoutInDisplayCutoutMode = WindowManager.LayoutParams.LAYOUT_IN_DISPLAY_CUTOUT_MODE_DEFAULT;
getWindow().setAttributes(lp);
```
### 通知功能的变更 Channel settings, broadcasts, and Do Not Disturb

Android 8.0 引入了通知渠道，允许您为要显示的每种通知类型创建可由用户自定义的渠道。 Android 9 通过下列变更简化通知渠道设置：

- [x] 屏蔽渠道组：现在，用户可以针对某个应用在通知设置中屏蔽整个渠道组。 您可以使用 isBlocked() 函数确定何时屏蔽一个渠道组，从而不会向该组中的渠道发送任何通知。
此外，您的应用可以使用全新的 **getNotificationChannelGroup()** 函数查询当前渠道组设置。
- [x] 全新的广播 Intent 类型：现在，当通知渠道和渠道组的屏蔽状态发生变更时，Android 系统将发送广播 Intent。 拥有已屏蔽的渠道或渠道组的应用可以侦听这些 Intent 并做出相应的回应。 有关这些 Intent 操作和 extra 的更多信息，请参阅 NotificationManager 参考中更新的常量列表。 有关响应广播 Intent 的信息，请参阅广播。
- [x] *NotificationManager.Policy* 有 3 种新的“请勿打扰”优先级类别：
    - **PRIORITY_CATEGORY_ALARMS** 优先处理警报。
    
    - **PRIORITY_CATEGORY_MEDIA** 优先处理媒体源的声音，如媒体和语音导航。
    
    - **PRIORITY_CATEGORY_SYSTEM** 优先处理系统声音。 
- [x] *NotificationManager.Policy* 还有 7 种新的“请勿打扰”常量，可以用来抑制视觉中断：
    - **SUPPRESSED_EFFECT_FULL_SCREEN_INTENT** 防止通知启动全屏 Activity。
    - **SUPPRESSED_EFFECT_LIGHTS** 屏蔽通知灯。
    - **SUPPRESSED_EFFECT_PEEK** 防止通知短暂进入视图（“滑出”）。
    - **SUPPRESSED_EFFECT_STATUS_BAR** 防止通知显示在支持状态栏的设备的状态栏中。
    - **SUPPRESSED_EFFECT_BADGE** 在支持标志的设备上屏蔽标志。 如需了解详细信息，请参阅修改通知标志。
    - **SUPPRESSED_EFFECT_AMBIENT** 在支持微光显示的设备上屏蔽通知。
    - **SUPPRESSED_EFFECT_NOTIFICATION_LIST** 防止通知显示在支持列表视图（如通知栏或锁屏）的设备的列表视图中。

### 隐私权变更
为了增强用户隐私，Android 9 引入了若干行为变更，如限制后台应用访问设备传感器、限制通过 Wi-Fi 扫描检索到的信息，以及与通话、手机状态和 Wi-Fi 扫描相关的新权限规则和权限组。
无论采用哪一种目标 SDK 版本，这些变更都会影响运行于 Android 9 上的所有应用。

### 后台对传感器的访问受限
Android 9 限制后台应用访问用户输入和传感器数据的能力。 如果您的应用在运行 Android 9 设备的后台运行，系统将对您的应用采取以下限制：
您的应用不能访问麦克风或摄像头。
使用连续报告模式的传感器（例如加速度计和陀螺仪）不会接收事件。
使用变化或一次性报告模式的传感器不会接收事件。
如果您的应用需要在运行 Android 9 的设备上检测传感器事件，请使用前台服务。

### 限制访问通话记录
Android 9 引入 CALL_LOG 权限组并将 READ_CALL_LOG、WRITE_CALL_LOG 和 PROCESS_OUTGOING_CALLS 权限移入该组。 在之前的 Android 版本中，这些权限位于 PHONE 权限组。
如果应用需要访问通话记录或者需要处理去电，则您必须向 CALL_LOG 权限组明确请求这些权限。 否则会发生 ==SecurityException==。

### 限制访问电话号码
在未首先获得 READ_CALL_LOG 权限的情况下，除了应用的用例需要的其他权限之外，运行于 Android 9 上的应用无法读取电话号码或手机状态。
与来电和去电关联的电话号码可在手机状态广播（比如来电和去电的手机状态广播）中看到，并可通过 PhoneStateListener 类访问。 但是，如果没有 READ_CALL_LOG 权限，则 PHONE_STATE_CHANGED 广播和 PhoneStateListener <mark>提供的电话号码字段为空</mark>。
要从手机状态中读取电话号码，请根据您的用例更新应用以请求必要的权限：

要通过 PHONE_STATE Intent 操作读取电话号码，同时需要 READ_CALL_LOG 权限和 READ_PHONE_STATE 权限。
要从 onCallStateChanged() 中读取电话号码，只需要 READ_CALL_LOG 权限。 不需要 READ_PHONE_STATE 权限。


### 电话信息现在依赖设备位置设置
如果用户在运行 Android 9 的设备上<mark>停用设备定位</mark>，则以下函数不提供结果：

TelephonyManager.getAllCellInfo()
TelephonyManager.listen()
TelephonyManager.getCellLocation()
TelephonyManager.getNeighboringCellInfo()


### Build.SERIAL 始终设置为 "UNKNOWN" 以保护用户的隐私。
如果您的应用需要访问设备的硬件序列号，您应改为请求 READ_PHONE_STATE权限，然后调用 **getSerial()**。

### 多进程 webview 信息访问限制
在 Android P 中为了提升系统的安全性，用户无法在多进程的 webview 中共享数据目录，该目录下存储的是一些 cookies、Http 缓存和其他一些永久、临时的缓存。当下不少应用会把 webview 放在另一个进程中打开以避免内存泄漏，但是他们 cookies 的设置往往还是在主进程中，所以开发者需要仔细排查自己的应用是否有这么使用，webview 相关运行是否正常等。

### 对使用非 SDK 接口的限制
为帮助确保应用稳定性和兼容性，此平台对某些非 SDK 函数和字段的使用进行了限制；无论您是直接访问这些函数和字段，还是通过反射或 JNI 访问，这些限制均适用。 在 Android 9 中，您的应用可以继续访问这些受限的接口；<mark>该平台通过 toast 和日志条目提醒您注意这些接口</mark>。 如果您的应用显示这样的 toast，则必须寻求受限接口之外的其他实现策略。 如果您认为没有可行的替代策略，您可以提交错误以请求重新考虑此限制。


### 对于非SDK 接口

浅灰名单：仍可以访问的非 SDK 函数/字段。
深灰名单：
对于目标 SDK 低于 API 级别 28 的应用，允许使用深灰名单接口。
对于目标 SDK 为 API 28 或更高级别的应用：行为与黑名单相同。
黑名单：受限，无论目标 SDK 如何
平台将提示接口并不存在。
例如，无论应用何时尝试使用接口，<mark>平台都会引发 NoSuchMethodError/NoSuchFieldException，</mark>即使应用想要了解某个特殊类别的字段/函数名单，平台也不会包含接口。


### 检测是否使用了非SDK接口

[工具veridex](https://android.googlesource.com/platform/prebuilts/runtime/+/master/appcompat)


1. 下载工具，阅读README.txt
2. 打包一个应用 APK，建议使用 release 包，排除一些未使用到的单元测试类或者其他因素的影响，取消混淆，将 APK 放到工具目录下；

3. 执行命令 ./appcompat.sh --dex-file=test.apk，在终端上会输出三个名单每个 API 的详细调用处


### 非 SDK API 的处理
适配的原则是优先黑名单和深灰名单，浅灰名单在官方未有替代 API 之前可以暂时不适配，在 Android P 上运行也不会有任何问题。

- [x] 向google申请
在之前 DP 版本时开发者如果遇到了不得不使用的黑名单或者深灰名单 API，需要向 google 官方及时提出反馈 (反馈url)申请将其移动到浅灰名单中，但是目前正式版本已经发布，未得知该申请通道是否仍有效。
- [x] 针对第三方库调用到了非 SDK API 接口，解决办法当然是直接查询相关资料或者联系库提供方，确认是否有适配 Android P 新版本的 SDK。还有需要提到的一点，就算更换适配完成的第三方 SDK 后，仍然可能会在同一地方扫描出非 SDK API 的调用，这是因为适配工程师只是在调用处加了一个 try-catch 保护逻辑，虽然这样也勉强叫做适配完成，但是还是强烈建议大家使用如下的适配方式：

```
if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.P) {
// Android P or above
} else {
// below Android P
}
```


严格按照上面的适配方案，扫描工具就不会再扫描出此处的非 SDK API 调用，我们也无需每次都去确认所有非 SDK API 调用处都加了保护逻辑。
当然如果第三方库没有适配也没有近期适配的意向，目前有两种方法：
- [ ] 第一种是屏蔽入口；
- [ ] 第二种是反编译 SDK，在关键地方加上适配代码；

### 非Activity-Context启动Activity，现在强制执行 
FLAG_ACTIVITY_NEW_TASK 要求
Apache HTTP 客户端弃用，影响采用非标准 ClassLoader 的应用
将 compileSdkVersion 升级到 28 之后，如果在项目中用到了 Apache HTTP client 的相关类，<mark>就会抛出找不到这些类的错误</mark>。这是因为官方已经<mark>在 Android P 的启动类加载器中将其移除</mark>，如果仍然需要使用 Apache HTTP client.

1. 在 Manifest 文件中加入：


```
<uses-library android:name="org.apache.http.legacy" android:required="false"/>
```



2. 或者也可以直接将 Apache HTTP client 的相关类打包进 APK 中。
3. 如果它们委托给 系统 ClassLoader，则应用在 Android 9 或更高版本上将失败并显示 NoClassDefFoundError，因为 系统 ClassLoader 不再识别这些类。 为防止将来出现类似问题，一般情况下，应用应通过 应用 ClassLoader加载类，而不是直接访问系统 ClassLoader

### 前台服务
针对 Android 9 或更高版本并使用前台服务的应用<mark>必须请求</mark> FOREGROUND_SERVICE 权限。 这是普通权限，因此，系统会自动为请求权限的应用授予此权限。
如果针对 Android 9 或更高版本的应用尝试创建一个前台服务且未请求 FOREGROUND_SERVICE，则系统会引发 SecurityException。





