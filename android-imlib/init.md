---
sidebar_position: 20
title: 初始化
---

# 初始化

为确保您可以正常连接融云服务器和使用融云即时通讯服务（IM 服务），您须调用 init 方法初始化 IMLib SDK。本文中将详细说明 IMLib SDK 初始化的方法。

首次使用融云的用户，建议您先阅读 [IMLib SDK 快速上手]，以完成开发者账号注册等工作。

## 推送

推送是常见的基础功能。IMLib SDK 已集成融云自有推送，以及多家第三方推送，且会在 SDK 初始化后触发。因此，客户端的推送配置必须在初始化之前提供。详细说明请参见[启用推送]。

## 海外数据中心

- 如果您使用海外数据中心，且使用 5.4.2 及更新版本的开发版（dev）SDK，请注意在初始化配置中传入正确的区域码（[AreaCode](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-init-option/-area-code/index.html#-495745558%2FClasslikes%2F1814687565)）。
- 如果您使用海外数据中心，且使用稳定版（stable） SDK，或使用早于 5.4.2 版本的开发版（dev）SDK，必须在初始化之前修改 IMLib SDK 连接的服务地址为海外数据中心地址。否则 SDK 默认连接中国国内数据中心服务地址。详细说明请参见[配置海外数据中心服务地址]。

## 进程

:::tip

 从 5.3.0 版本开始，IMLib SDK 支持单进程。
:::


SDK 可支持多进程和单进程机制。

如果 SDK 版本 \< 5.3.3 或 ≧ 5.3.5，默认采用多进程机制，初始化之后，应用会启动以下进程：

1. 应用的主进程
2. `<应用包名>:ipc`。此进程是 IM 通信的核心进程，和主进程任务相互隔离。
3. `io.rong.push`：融云默认推送进程。该进程是否启动由推送通道的启用策略决定。详细说明可参考[启用推送]。

如果 SDK 版本 ≧ 5.3.3 且 \< 5.3.5，默认采用单进程机制。

从 5.3.0 版本开始，可以在初始化前通过 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 的 `enableSingleProcess` 接口开启或关闭单进程。请注意，开启单进程后融云会占用主进程的部分内存。

```java
//开启单进程
RongCoreClient.getInstance().enableSingleProcess(true);
```

## 初始化 SDK

:::tip

 以下初始化方法要求 SDK 版本 ≧ 5.4.2。您可前往 [融云官网 SDK 下载页面] 查询最新开发版（Dev）和稳定版（Stable）的版本号。
:::


请在 Application 的 `onCreate()` 方法中初始化 SDK，传入**生产**或**开发**环境的 App Key。

```java
String appKey = "YourAppKey"; // example: bos9p5rlcm2ba
InitOption initOption = new InitOption.Builder().build();

RongCoreClient.init(this, appKey, initOption);
```

初始化配置（`InitOption`）中封装了区域码（[AreaCode](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-init-option/-area-code/index.html#-495745558%2FClasslikes%2F1814687565)）配置。SDK 将通过区域码获取有效的导航服务地址、文件服务地址、数据统计服务地址、和日志服务地址等配置。

- 如果 App Key 属于中国（北京）数据中心，您无需传入任何配置，SDK 会使用默认配置。
- 如果 App Key 属于海外数据中心，则必须传入有效的区域码（[AreaCode](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-init-option/-area-code/index.html#-495745558%2FClasslikes%2F1814687565)）配置。请务必在控制台核验当前 App Key 所属海外数据中心后，找到 [AreaCode](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-init-option/-area-code/index.html#-495745558%2FClasslikes%2F1814687565) 中对应的枚举值进行配置。

例如，使用新加坡数据中心的应用的**生产**或**开发**环境的 App Key：

```java
String appKey = "Singapore_dev_AppKey";
AreaCode areaCode = AreaCode.SG;

InitOption initOption = new InitOption.Builder()
    .setAreaCode(areaCode)
    .build();

RongCoreClient.init(context, appKey, initOption);
```

除区域码外，初始化配置（`InitOption`）中还封装了以下配置：

- 是否开启推送的开关（`enablePush`）：是否整体禁用推送。
- 主进程开关（`isMainProcess`）：是否为主进程。默认情况下由 SDK 判断进程。
- 导航服务地址（`naviServer`）：一般情况下不建议单独配置。SDK 内部默认使用与区域码对应的地址。
- 文件服务地址（`fileServer`）：仅限私有云使用。
- 数据统计服务地址（`statisticServer`）：一般情况下不建议单独配置。SDK 内部默认使用与区域码对应的地址。
<!-- - 云控服务地址（`cloudControlServer`）：自定义云控请求地址。（从 5.24.0 开始支持） -->

```java
String appKey = "Your_AppKey";
AreaCode areaCode = AreaCode.BJ;

InitOption initOption = new InitOption.Builder()
    .setAreaCode(areaCode)
    .enablePush(true)
    .setFileServer("http(s)://fileServer")
    .setNaviServer("http(s)://naviServer")
    .setStatisticServer("http(s)://StatisticServer")
    .build();

RongCoreClient.init(context, appKey, initOption);
```

## 初始化 SDK（旧版）

:::tip

 如果您使用的开发版 SDK 版本号小于 5.4.2，或者稳定版 SDK 版本号小于等于 5.3.8，只能使用以下方式初始化。建议您及时升级 SDK 到最新版本。
:::


请在 Application 的 `onCreate()` 方法中初始化 SDK，传入**生产**或**开发**环境的 App Key。您可以使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 或 `RongIMClient` 的初始化方法。

```java
RongCoreClient.init(context, appKey, enablePush);
```

| 参数       | 类型    | 说明                                                                                                       |
|------------|---------|----------------------------------------------------------------------------------------------------------|
| context    | Context | 应用上下文。                                                                                                |
| appKey     | String  | 应用的**开发**环境或**生产**环境 AppKey，请勿混淆。如果您选择在 `AndroidManifest.xml`设置 appKey，此处可不传。 |
| enablePush | boolean | 是否[启用推送]。默认 `true`，即开启推送功能。请注意，首次设置 enablePush 有效。若您已经将 enablePush 设置为 `true` ，并且已经上报成功推送 token，之后即使设置 enablePush 为 `false`，也不会生效。                                                                 |

仅为保持向后兼容，IMLib SDK 5x 仍支持在 `AndroidManifest.xml` 中配置 App Key。但考虑到 App Key 安全性，融云已不再建议使用这种配置方式。

建议在用户接受隐私协议后，再进行初始化。

```java
/**
 * 应用启动时，判断用户是否已接受隐私协议，如果已接受，正常初始化；否则跳转到隐私授权页面请求用户授权。
 */
public class App extends Application {
    @Override
    public void onCreate() {
        super.onCreate();

        //伪代码，从 sp 里读取用户是否已接受隐私协议
        boolean isPrivacyAccepted = getPrivacyStateFromSp();
        //用户已接受隐私协议，进行初始化
        if (isPrivacyAccepted) {
            String appKey = "融云控制台创建的应用的 AppKey";
            //第一个参数必须传应用上下文
            RongIMClient.init(this.getApplicationContext(), appKey);
        } else {
            //用户未接受隐私协议，跳转到隐私授权页面。
            goToPrivacyActivity();
        }
        ...
    }
}

/**
 * 该类为隐私授权页面，示范如何在用户接受隐私协议后进行 IM 初始化。
 */
public class PrivacyActivity extends Activity implements View.OnClickListener {
    ...
    @Override
    public void onClick(View v) {
        switch (v.getId()) {
            case R.id.accept_privacy:
                //伪代码，保存到 sp
                savePrivacyStateToSp();

                String appKey = "融云控制台创建的应用的 AppKey";
                //第一个参数必须传应用上下文
                RongIMClient.init(this.getApplicationContext(), appKey);
                break;
            default:
                ...
        }
    }
}
```


<!-- links -->
<!-- internal-->
[配置海外数据中心服务地址]: https://help.rongcloud.cn/t/topic/1069
[启用推送]: /android-imlib/push/overview
[IMLib SDK 快速上手]: ./quickstart.md
<!-- external-->
[融云官网 SDK 下载页面]: https://www.rongcloud.cn/devcenter?type=sdk

