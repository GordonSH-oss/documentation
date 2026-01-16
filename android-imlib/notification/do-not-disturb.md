---
title: 全局免打扰
---

# 全局免打扰

SDK 支持为当前用户设置全局免打扰时段与免打扰级别。

- [setNotificationQuietHoursLevel] 接口会设置一个从任意时间点（`HH:MM:SS`）开始的免打扰时间窗口。在再次设置或删除用户免打扰时间段之前，当次设置的免打扰时间窗口会每日重复生效。例如，App 用户希望设置永久全天免打扰，可设置 `startTime` 为 `00:00:00`，`period` 为 `1439`。
- 单个用户仅支持设置一个时间段，重复设置会覆盖该用户之前设置的时间窗口。
- 如果 SDK 版本 \< 5.2.2，仅支持设置免打扰时段，不支持同时设置免打扰级别。建议您尽快升级到最新稳定版或开发版。

:::tip

 在经 SDK 设置的全局免打扰时段内：

 - 如果客户端处于离线状态，融云服务端将不会进行推送通知。
 - **全局免打扰时段**为用户级别的免打扰设置，且具有最高优先级。在用户设置了**全局免打扰时段**时，均以此设置的免打扰级别为准。
:::

在 App 自行实现本地通知处理时，如果检测到客户端 App 已转至后台运行，可通过 IMLib SDK 提供的全局免打扰接口决定是否弹出本地通知，以实现全局免打扰的效果。

## 设置免打扰时段与级别（SDK ≥ 5.2.2）

从 SDK 5.2.2 开始，为当前用户设置免打扰时间段时，可使用以下免打扰级别：

#### 参数说明

| 枚举值                                              | 数值 | 说明                                                                                                     |
|:----------------------------------------------------|:-----|:-------------------------------------------------------------------------------------------------------|
| PUSH_NOTIFICATION_QUIET_HOURS_LEVEL_DEFAULT         | `0`  | 未设置。如未设置，SDK 会依次查询消息所属群的用户级别免打扰设置及其他非用户级别设置，再判断是否需要推送通知。 |
| PUSH_NOTIFICATION_QUIET_HOURS_LEVEL_MENTION_MESSAGE | `1`  | 仅针对 @ 消息进行通知，包括 @指定用户 和 @所有人的消息。                                                   |
| PUSH_NOTIFICATION_QUIET_HOURS_LEVEL_BLOCKED         | `5`  | 不接收通知，即使为 @ 消息也不推送通知。                                                                    |

### 设置免打扰时段与级别

:::tip

 该接口在 [ChannelClient] 中，从 5.2.2 版本开始支持。
:::

设置消息通知免打扰时间。在免打扰时间内接收到消息时，会根据该接口设置的免打扰级别判断是否需要推送消息通知。

#### 接口

```java
ChannelClient.getInstance().setNotificationQuietHoursLevel(startTime, spanMinutes,callback);
```


#### 参数说明

| 参数        | 类型                                  | 说明                                                                                                                                                                                                                                                                                          |
|:------------|:--------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| startTime   | String                                | 开始时间，精确到秒。格式为 `HH:MM:SS`，例如 `01:31:17`。                                                                                                                                                                                                                                          |
| spanMinutes | int                                   | 免打扰时间窗口大小，单位为分钟。范围为 [1-1439]。                                                                                                                                                                                                                                                |
| lev         | PushNotificationQuietHoursLevel       | <ul><li>`1`： 仅针对 @ 消息进行通知，包括 @指定用户 和 @所有人的消息。如果消息所属会话类型为单聊，则代表不通知。</li><li>`0`： 未设置。如未设置，SDK 会依次查询消息所属群的用户级别免打扰设置及其他非用户级别设置，再判断是否需要推送通知。</li><li>`5`：不接收通知，即使为 @ 消息也不推送通知。</li></ul> |
| callback    | ResultCallback\<List\<Conversation\>> | 回调接口                                                                                                                                                                                                                                                                                      |

#### 示例代码

```java
String startTime = "00:00:00";
int spanMinutes = 1439;

ChannelClient.getInstance().setNotificationQuietHoursLevel(startTime, spanMinutes,
        IRongCoreEnum.PushNotificationQuietHoursLevel.PUSH_NOTIFICATION_QUIET_HOURS_LEVEL_DEFAULT,
        new IRongCoreCallback.OperationCallback() {
                @Override
                public void onSuccess() {

                }

                @Override
                public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

               }
        });

```

### 设置免打扰时段与级别（支持时区）

:::tip

[ChannelClient] 接口从 5.14.0 版本开始支持。
:::

通过免打扰配置( `NotificationQuietHoursSetting` )设置消息通知免打扰时间，可以设置时区。在免打扰时间内接收到消息时，会根据该接口设置的免打扰级别判断是否需要推送消息通知。

#### 接口

```java
ChannelClient.getInstance().setNotificationQuietHoursLevel(setting, callback);
```

#### 免打扰 `NotificationQuietHoursSetting` 参数介绍

| 参数        | 类型                                  | 说明                                                                                                                                                                                                                                                                                          |
|:------------|:--------------------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| startTime   | String                                | 开始时间，精确到秒。格式为 `HH:MM:SS`，例如 `01:31:17`。                                                                                                                                                                                                                                          |
| spanMinutes | int                                   | 免打扰时间窗口大小，单位为分钟。范围为 [1-1439]。                                                                                                                                                                                                                                                |
| lev         | PushNotificationQuietHoursLevel       | <ul><li>`1`： 仅针对 @ 消息进行通知，包括 @指定用户 和 @所有人的消息。如果消息所属会话类型为单聊，则代表不通知。</li><li>`0`： 未设置。如未设置，SDK 会依次查询消息所属群的用户级别免打扰设置及其他非用户级别设置，再判断是否需要推送通知。</li><li>`5`：不接收通知，即使为 @ 消息也不推送通知。</li></ul> |
| timeZone    | String   | 国内数据中心默认  Asia/Shanghai ，海外默认 UTC。                                                                                                                                                                                                                                                                                     |

#### 示例代码

```java
NotificationQuietHoursSetting setting = new NotificationQuietHoursSetting();
setting.setStartTime("00:00:00");
setting.setSpanMinutes(1439);
setting.setTimeZone("Asia/Shanghai");
setting.setLevel(IRongCoreEnum.PushNotificationQuietHoursLevel.PUSH_NOTIFICATION_QUIET_HOURS_LEVEL_DEFAULT);

ChannelClient.getInstance().setNotificationQuietHoursLevel(setting, new IRongCoreCallback.OperationCallback() {
                @Override
                public void onSuccess() {

                }

                @Override
                public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

               }
        });

```

### 移除免打扰时段与级别

:::tip

 该接口在 [ChannelClient] 中，从 5.2.2 版本开始支持。
:::


您可以调用以下方法将免打扰时间段设置移除。

#### 接口

```java
ChannelClient.getInstance().removeNotificationQuietHours(callback);
```

### 获取免打扰时段与级别

:::tip

 该接口在 [ChannelClient] 中，从 5.2.2 版本开始支持。
:::


您可以通过以下方法获取免打扰时间段设置。在免打扰时间内接收到消息时，会根据当前免打扰级别判断是否需要推送消息通知。

#### 接口

```java
ChannelClient.getInstance().getNotificationQuietHoursLevel(callback);
```

#### 参数说明

| 参数     | 类型                                  | 说明                                                                     |
|:---------|:--------------------------------------|:-----------------------------------------------------------------------|
| callback | [GetNotificationQuietHoursCallbackEx] | 获取免打扰时间的回调。获取成功时会返回 `startTime`、`spanMinutes`、`level`。 |

[GetNotificationQuietHoursCallbackEx]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-get-notification-quiet-hours-callback-ex/index.html


### 获取免打扰时段与级别（支持时区）

:::tip

[ChannelClient] 接口从 5.14.0 版本开始支持。
:::

您可以通过以下方法获取免打扰时间段设置，`NotificationQuietHoursSetting` 中可以查看时区。在免打扰时间内接收到消息时，会根据当前免打扰级别判断是否需要推送消息通知。

#### 接口

```java
ChannelClient.getInstance().getNotificationQuietHoursLevel(callback);
```
#### 参数说明

| 参数     | 类型                                | 说明                                                                     |
|:---------|:----------------------------------|:-----------------------------------------------------------------------|
| callback | [ResultCallback]                  | 获取免打扰时间的回调，获取成功时会返回 `startTime`、`spanMinutes`、`level`、`timeZone`。 |


[setNotificationQuietHoursLevel]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/set-notification-quiet-hours-level.html?query=public%20abstract%20void%C2%A0setNotificationQuietHoursLevel(NotificationQuietHoursSetting%C2%A0setting,%20IRongCoreCallback.OperationCallback%C2%A0callback)
[ChannelClient]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html
