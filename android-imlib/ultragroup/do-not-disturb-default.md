---
title: 设置群/频道默认免打扰
sidebar_position: 210
---

# 设置群/频道默认免打扰

超级群业务支持为指定的群，或群频道设置默认免打扰逻辑。默认免打扰逻辑对所有群成员生效，一般由超级群的管理员进行设置。

如果您希望从 App 服务端控制指定超级群，或指定群频道默认免打扰逻辑，可参考服务端 API 文档[设置超级群/频道默认免打扰](/platform-chat-api/ultragroup/do-not-disturb/set-do-not-disturb)。

**注意事项**

- 在融云服务端判断是否需要推送超级群消息时，指定的超级群，或群频道的默认免打扰配置优先级均低于用户级别配置。如果存在任何用户级别的免打扰配置，则优先以用户级别免打扰配置为准进行判断。

   :::tip

   即时通讯业务免打扰功能的 **用户级别设置** 支持控制指定的单聊会话、群聊会话、超级群会话、超级群频道的免打扰级别，并可设置全局免打扰的时间段与级别。用户级别设置优先级如下：**全局免打扰** > **按频道设置的免打扰** > **按会话设置的免打扰** > **按会话类型设置的免打扰**。详见[超级群免打扰功能概述](./do-not-disturb-about.md)。
   :::


- 为指定的超级群设置的默认免打扰逻辑，自动适用于群下的所有频道。如果针对频道另行设置了默认免打扰逻辑，则以该频道的默认设置为准。

## 支持的免打扰级别

指定超级群或群频道的默认免打扰级别可设置为以下任一级别：

|   枚举值    |          数值           |           说明           |
| :-------- | :--------------------- |  :----------------------- |
| PUSH_NOTIFICATION_LEVEL_ALL_MESSAGE |`-1` | 所有消息均可进行通知。|
| PUSH_NOTIFICATION_LEVEL_DEFAULT | `0` | 未设置。未设置时均为此初始状态。<br/><br/>**注意**：在此状态下，如果超级群与群频道均为未设置，则认为超级群与频道的默认免打扰级别为全部消息都通知。|
| PUSH_NOTIFICATION_LEVEL_MENTION | `1` | 仅针对 @ 消息进行通知，包括 @指定用户 和 @所有人|
| PUSH_NOTIFICATION_LEVEL_MENTION_USERS| `2` | 仅针对 @ 指定用户消息进行通知，且仅针对被 @ 的指定的用户进行通知。<br/><br/>如：@张三，则张三可以收到推送； @所有人不会触发推送通知。|
| PUSH_NOTIFICATION_LEVEL_MENTION_ALL | `4` | 仅针对 @群全员进行通知，即只接收 @所有人的推送信息。|
| PUSH_NOTIFICATION_LEVEL_BLOCKED| `5` | 不接收通知，即使为 @ 消息也不推送通知。|


```java
public enum PushNotificationLevel {
          NONE(-100),
       /** 全部消息通知（接收全部消息通知 -- 显示指定关闭免打扰功能） */
       PUSH_NOTIFICATION_LEVEL_ALL_MESSAGE(-1),

        /** 未设置（向上查询群或者APP级别设置）//存量数据中0表示未设置 */
        PUSH_NOTIFICATION_LEVEL_DEFAULT(0),

        /** 群聊，超级群 @所有人 或者 @成员列表有自己 时通知；单聊代表消息不通知 */
        PUSH_NOTIFICATION_LEVEL_MENTION(1),

        /** 群聊，超级群 @成员列表有自己时通知，@所有人不通知；单聊代表消息不通知 */
        PUSH_NOTIFICATION_LEVEL_MENTION_USERS(2),

        /** 群聊，超级群 @所有人通知，其他情况都不通知；单聊代表消息不通知 */
        PUSH_NOTIFICATION_LEVEL_MENTION_ALL(4),

        /** 消息通知被屏蔽，即不接收消息通知 */
        PUSH_NOTIFICATION_LEVEL_BLOCKED(5);
}
```

## 设置指定超级群的默认免打扰级别

```java
   class OperationCallback {
   onSuccess()
   onError(ErrorCode code)
   }
   public void setUltraGroupConversationDefaultNotificationLevel(
   final String targetId,
   final IRongCoreEnum.PushNotificationLevel level,
   final IRongCoreCallback.OperationCallback callback);
```

## 查询指定超级群的默认免打扰级别

```java
   class ResultCallback {
   onSuccess(PushNotificationLevel level)
   onError(ErrorCode code)
   }

public void getUltraGroupConversationDefaultNotificationLevel(
final String targetId,
final IRongCoreCallback.ResultCallback<IRongCoreEnum.PushNotificationLevel> callback);
```

## 设置指定群频道的默认免打扰级别


```java
class OperationCallback {
onSuccess()
onError(ErrorCode code)
}

public void setUltraGroupConversationChannelDefaultNotificationLevel(
final String targetId,
final String channelId,
final IRongCoreEnum.PushNotificationLevel level,
final IRongCoreCallback.OperationCallback callback);
```

## 查询指定群频道的默认免打扰级别

```java
class ResultCallback {
onSuccess(RCPushNotificationLevel level)
onError(ErrorCode code)
}

public void getUltraGroupConversationChannelDefaultNotificationLevel(
final String targetId,
final String channelId,
final IRongCoreCallback.ResultCallback<IRongCoreEnum.PushNotificationLevel> callback);
```
