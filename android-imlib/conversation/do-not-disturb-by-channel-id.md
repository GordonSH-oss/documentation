---
title: 按频道设置免打扰
sidebar_position: 120
---

# 按频道设置免打扰

本文描述如何为超级群业务的指定频道（`channelId`）设置免打扰级别。

:::tip

融云即时通讯（IM）客户端 SDK 支持多维度、多级别的免打扰设置。
- 您可实现从 App Key、指定细分业务（仅超级群）、用户级别多个维度的免打扰功能配置。在融云服务端决定是否触发推送通知时，不同维度的优先级如下：**用户级别设置** > **指定超级群频道的默认配置**（仅超级群支持） > **指定超级群会话的默认配置**（仅超级群支持） > **App Key 级设置**。
- **用户级别设置**下包含多个细分维度。在融云服务端决定是否触发推送通知时，如存在用户级别配置，不同细分维度的优先级如下：**全局免打扰** > **按频道设置的免打扰** > **按会话设置的免打扰** > **按会话类型设置的免打扰**。详见[免打扰功能概述](./do-not-disturb-about.md)。

:::

## 支持的免打扰级别

免打扰级别提供了针对不同 @ 消息的免打扰控制。从 SDK 5.2.2 开始，指定频道的免打扰配置支持以下级别：

| 枚举值 | 数值 | 说明 |
|:------|:-----|:-----|
| `PUSH_NOTIFICATION_LEVEL_ALL_MESSAGE` | `-1` | 所有消息均可进行通知。 |
| `PUSH_NOTIFICATION_LEVEL_DEFAULT` | `0` | 未设置。未设置时均为此初始状态。<br/><br/>**注意**：在此状态下，如果超级群与群频道均为未设置，则认为超级群与频道的默认免打扰级别为全部消息都通知。 |
| `PUSH_NOTIFICATION_LEVEL_MENTION` | `1` | 仅针对 @ 消息进行通知，包括 @ 指定用户和 @ 所有人 |
| `PUSH_NOTIFICATION_LEVEL_MENTION_USERS` | `2` | 仅针对 @ 指定用户消息进行通知，且仅针对被 @ 的指定的用户进行通知。<br/><br/>如：@ 张三，则张三可以收到推送；@ 所有人不会触发推送通知。 |
| `PUSH_NOTIFICATION_LEVEL_MENTION_ALL` | `4` | 仅针对 @ 群全员进行通知，即只接收 @ 所有人的推送信息。 |
| `PUSH_NOTIFICATION_LEVEL_BLOCKED` | `5` | 不接收通知，即使为 @ 消息也不推送通知。 |

低于 5.2.2 的 SDK 版本仅支持设置为免打扰状态（不接收推送通知）或提醒状态（接收推送通知）。

## 管理频道的免打扰设置

超级群（UltraGroup）支持在超级群的会话下创建独立的频道（channel），对消息数据（会话、消息、未读数）和群组成员分频道进行聚合。SDK 支持在超级群业务中的用户（`userId`）设置指定群频道（`channelId`）的免打扰级别。

### 设置指定频道免打扰级别（SDK ≥ 5.2.2）

:::tip

该接口在 [`ChannelClient`](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中，从 5.2.2 版本开始支持。

:::

当前用户设置超级群频道中的消息的免打扰级别。

#### 接口

```java
ChannelClient.getInstance().setConversationChannelNotificationLevel(conversationType,targetId,channelId,callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
|:-----|:-----|:-----|
| `conversationType` | [`ConversationType`] | 会话类型。 |
| `targetId` | `String` | 会话 ID |
| `channelId` | `String` | 超级群的会话频道 ID。<ul><li>如果传入频道 ID，则针对该指定频道设置消息免打扰级别。如果不指定频道 ID，则对所有超级群消息生效。</li><li>**注意**：2022.09.01 之前开通超级群业务的客户，如果不指定频道 ID，则默认传""空字符串，即仅针对指定超级群会话（`targetId`）中**不属于任何频道的消息**设置免打扰状态级别。如需修改请提交工单。</li></ul> |
| `lev` | `PushNotificationLevel` | <ul><li>`-1`：全部消息通知</li><li>`0`：未设置（用户未设置情况下，默认以群或者 APP 级别的默认设置为准，如未设置则全部消息都通知）</li><li>`1`：仅针对 @ 消息进行通知</li><li>`2`：仅针对 @ 指定用户进行通知<p>如：@ 张三则张三可以收到推送，@ 所有人时不会收到推送。</p></li><li>`4`：仅针对 @ 群全员进行通知，只接收 @ 所有人的推送信息。</li><li>`5`：不接收通知</li></ul> |
| `callback` | `OperationCallback` | 回调接口。 |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 ID";
String channelId = "频道 ID";

ChannelClient.getInstance().setConversationChannelNotificationLevel(conversationType,targetId,channelId,
        IRongCoreEnum.PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_DEFAULT,new IRongCoreCallback.OperationCallback() {
                @Override
                public void onSuccess() {
                      if (callback != null) {
                          callback.onSuccess(notificationStatus);
                      }
                }

                @Override
                public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
                     if (callback != null) {
                          callback.onError(coreErrorCode);
                     }
               }
        });
```

### 获取指定频道免打扰级别（SDK ≥ 5.2.2）

获取当前用户设置的超级群频道免打扰级别。

#### 接口

```java
ChannelClient.getInstance().getConversationChannelNotificationLevel(conversationType,targetId,channelId,callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
|:-----|:-----|:-----|
| `conversationType` | [ConversationType] | 会话类型 |
| `targetId` | `String` | 会话 ID |
| `channelId` | `String` | 超级群的会话频道 ID，获取会话指定频道的设置。 |
| `callback` | `ResultCallback<IRongCoreEnum.PushNotificationLevel>` | 回调接口 |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 ID";
String channelId = "频道 ID";

ChannelClient.getInstance().getConversationChannelNotificationLevel(conversationType,targetId,channelId,new IRongCoreCallback.ResultCallback<
        IRongCoreEnum.PushNotificationLevel>() {
               @Override
               public void onSuccess(IRongCoreEnum.PushNotificationLevel level) {

               }

               @Override
              public void onError(IRongCoreEnum.CoreErrorCode e) {

              }
        });
```

## 多端同步免打扰状态 {#setlistener}

SDK 提供了会话状态（置顶或免打扰）同步机制，通过设置会话状态同步监听器，当在其它端修改会话状态时，可在本端实时监听到会话状态的改变。详见[多端同步免打扰/置顶](./share-conversation-status-between-clients.md)。

<!--links -->
[`ConversationType`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

