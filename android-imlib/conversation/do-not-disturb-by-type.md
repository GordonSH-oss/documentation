---
title: 按会话类型设置免打扰
sidebar_position: 130
---

# 按会话类型设置免打扰

本文描述如何为指定类型（`conversationType`）的会话设置免打扰级别。

:::tip

 即时通讯客户端 SDK 支持多维度、多级别的免打扰设置。
 - 您可实现从 App Key、指定细分业务（仅超级群）、用户级别多个维度的免打扰功能配置。在融云服务端决定是否触发推送通知时，不同维度的优先级如下：**用户级别设置** > **指定超级群频道的默认配置**（仅超级群支持） > **指定超级群会话的默认配置**（仅超级群支持） > **App Key 级设置**。
 - **用户级别设置**下包含多个细分维度。在融云服务端决定是否触发推送通知时，如存在用户级别配置，不同细分维度的优先级如下：**全局免打扰** > **按频道设置的免打扰** > **按会话设置的免打扰** > **按会话类型设置的免打扰**。详见[免打扰功能概述](./do-not-disturb-about.md)。
:::


## 支持的免打扰级别

免打扰级别提供了针对不同 @ 消息的免打扰控制。从 SDK 5.2.2.1 开始，指定会话类型的免打扰配置支持以下级别：

|   枚举值    |          数值           |           说明           |
| :-------- | :--------------------- |  :----------------------- |
| PUSH_NOTIFICATION_LEVEL_ALL_MESSAGE |`-1` | 所有消息均可进行通知。|
| PUSH_NOTIFICATION_LEVEL_DEFAULT | `0` | 未设置。未设置时均为此初始状态。<br/><br/>**注意**：在此状态下，如果超级群与群频道均为未设置，则认为超级群与频道的默认免打扰级别为全部消息都通知。|
| PUSH_NOTIFICATION_LEVEL_MENTION | `1` | 仅针对 @ 消息进行通知，包括 @指定用户 和 @所有人|
| PUSH_NOTIFICATION_LEVEL_MENTION_USERS| `2` | 仅针对 @ 指定用户消息进行通知，且仅针对被 @ 的指定的用户进行通知。<br/><br/>如：@张三，则张三可以收到推送； @所有人不会触发推送通知。|
| PUSH_NOTIFICATION_LEVEL_MENTION_ALL | `4` | 仅针对 @群全员进行通知，即只接收 @所有人的推送信息。|
| PUSH_NOTIFICATION_LEVEL_BLOCKED| `5` | 不接收通知，即使为 @ 消息也不推送通知。|

低于 5.2.2.1 的 SDK 版本仅支持设置为免打扰状态（不接收推送通知）或提醒状态（接收推送通知）。

## 管理会话类型的免打扰级别

从 SDK 5.2.2.1 版本开始，支持在即时通讯业务中由用户（`userId`）为指定类型的会话（`conversationType`）设置免打扰级别，支持单聊、群聊、超级群会话。

### 设置指定会话类型免打扰级别

:::tip

 该接口在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中，从 5.2.2.1 版本开始支持。
:::


为用户设置指定会话类型（`conversationType`）的免打扰级别，支持单聊、群聊、超级群会话。

#### 接口

```java
ChannelClient.getInstance().setConversationTypeNotificationLevel(conversationType,lev,callback);
```

#### 参数说明
|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
|conversationType|[ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html)|会话类型。不支持聊天室类型，因为聊天室默认就是不接受消息提醒的。|
|lev|PushNotificationLevel | <ul><li>`-1`： 全部消息通知</li><li>`0`： 未设置（用户未设置情况下，默认以群 或者 APP级别的默认设置为准，如未设置则全部消息都通知）</li><li>`1`： 仅针对 @ 消息进行通知</li><li>`2`： 仅针对 @ 指定用户进行通知<p>如：@张三 则张三可以收到推送，@所有人 时不会收到推送。</p></li><li>`4`： 仅针对 @ 群全员进行通知，只接收 @所有人 的推送信息。</li><li>`5`： 不接收通知</li></ul>|
|callback |OperationCallback|回调接口|


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;

ChannelClient.getInstance().setConversationTypeNotificationLevel(conversationType,
        IRongCoreEnum.PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_DEFAULT,new IRongCoreCallback.OperationCallback() {
                @Override
                public void onSuccess() {

                }

                @Override
                public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

               }
        });
```


### 移除指定会话类型的免打扰级别

如需移除指定会话类型的免打扰级别设置，请调用设置接口，并将 `lev` 参数传入 0。

### 查询指定会话类型的免打扰级别

:::tip

 该接口在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中，从 5.2.2.1 版本开始支持。
:::


查询当前用户为指定会话类型（`conversationType`）设置的免打扰级别。

#### 接口
```java
ChannelClient.getInstance().getConversationTypeNotificationLevel(conversationType, callback);
```
#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
|conversationType|[ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html)|会话类型|
|callback |ResultCallback\<IRongCoreEnum.PushNotificationLevel\>|回调接口|


#### 示例代码
```java
ConversationType conversationType = ConversationType.PRIVATE;

ChannelClient.getInstance().getConversationTypeNotificationLevel(conversationType, new IRongCoreCallback.ResultCallback<
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

