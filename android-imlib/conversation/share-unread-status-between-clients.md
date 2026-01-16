---
title: 多端同步阅读状态
sidebar_position: 150
---

# 多端同步阅读状态

在即时通讯业务中，同一用户账号可能在多个设备上登录。仅在开通**多设备消息同步**服务后，融云会在多个设备之间同步消息数据，但设备上的会话中消息的已读/未读状态仅存储在本地。因此，应用程序可能希望将用户当前登录设备上指定会话的已读/未读状态同步给其他终端。

IMLib SDK 设计了多端同步单聊和群聊会话消息未读状态的机制。在一端主动调用同步消息未读状态接口 `syncConversationReadStatus`，其他端可通过会话状态同步监听器监听到会话中消息未读状态的最新数据。

:::tip

 - 如果 SDK 版本 ≦ 5.6.2，不支持多端同步**系统会话**的阅读状态。
 - 超级群业务采用不同多端消息未读状态同步机制。详见超级群文档[清除消息未读状态](../ultragroup/sync-read-status.md)。
:::


## 主动同步消息未读状态

多端登录时，通知其它终端同步某个会话的消息未读状态。

#### 接口

```java
RongIMClient.getInstance().syncConversationReadStatus(conversationType, targetId, timestamp, callback);
```

#### 参数说明

|       参数       |    类型    |        说明         |
|:--------------- |:---------  |:------------------ |
|conversationType| [ConversationType] | 会话类型|
|targetId|String |会话 ID|
| timestamp |String|该会话中已读的最后一条消息的发送时间戳|
|callback | RongIMClient.OperationCallback |回调接口|

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
String timestamp = "12222222";

RongIMClient.getInstance().syncConversationReadStatus(conversationType, targetId, timestamp, new RongIMClient.OperationCallback() {
    @Override
    public void onSuccess() {
    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {
    }
});
```

## 监听消息未读状态同步数据

`RongIMClient` 中提供了 `SyncConversationReadStatusListener` 监听器。客户端设置该监听器后，才能接收来自其他同步的阅读状态数据。

#### 接口

```java
RongIMClient.getInstance().setSyncConversationReadStatusListener(listener);
```

IMLib SDK 在接收到阅读状态同步数据后，会触发监听器的 `onSyncConversationReadStatus` 方法。SDK 会将指定会话中早于等于 `syncConversationReadStatus` 传入时间戳的消息均置为已读：

```java
onSyncConversationReadStatus(Conversation.ConversationType type, String targetId)
```
#### 参数说明

|       参数       |    类型    |        说明         |
|:--------------- |:---------  |:------------------ |
|conversationType| [ConversationType] | 会话类型|
|targetId|String |会话 ID|

<!-- links -->
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

