---
title: 处理会话未读消息数
sidebar_position: 20
---

# 处理会话未读消息数

您可以使用 IMLib SDK 提供的接口直接获取会话中的未读消息数。具体能力如下：

- 获取所有会话（不含聊天室）中的未读消息总数（`getTotalUnreadCount`）
- 获取指定会话中的总未读消息数，或指定会话中指定消息类型的总未读消息数，或按会话类型获取总未读消息总数，或指定会话类型是否包含免打扰消息的总未读消息数（`getUnreadCount`）

在用户使用您的 App 时，UI 上未读计数可能需要发生变化，此时您可以清除会话中的未读数（`clearMessagesUnreadStatus`）。

## 获取所有会话总未读消息数

您可以使用 `getTotalUnreadCount` 获取所有类型会话（不含聊天室）中未读消息的总数。
#### 接口

```java
RongIMClient.getInstance().getTotalUnreadCount(callback);
```
获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

#### 参数说明
| 参数             | 类型                      | 说明                            |
|:-----------------|:--------------------------|:------------------------------|
| callback         | ResultCallback\<Integer\> | 回调接口，返回未读消息数（`unReadCount`）                     |


#### 示例代码
```java
RongIMClient.getInstance().getTotalUnreadCount(new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode ErrorCode) {

    }
});
```

## 获取指定会话的总未读消息数

获取指定会话中的未读消息总数。

#### 接口
```java
RongIMClient.getInstance().getUnreadCount(conversationType, targetId, callback);
```
#### 参数说明

| 参数             | 类型                      | 说明                            |
|:-----------------|:--------------------------|:------------------------------|
| conversationType | [ConversationType]        | 会话类型。不适用于聊天室、超级群。 |
| targetId         | String                    | 会话 ID                         |
| callback         | ResultCallback\<Integer\> | 回调接口                        |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

#### 示例代码
```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";

RongIMClient.getInstance().getUnreadCount(conversationType, targetId,
    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode ErrorCode) {

    }
});
```

## 获取指定会话中指定消息类型的总消息未读数

:::tip

 该方法在 SDK 5.1.5 版本引入。该方法仅在 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 中提供。
:::


获取指定会话内指定的某一个或多个消息类型的未读数。

#### 接口
```java
RongCoreClient.getInstance().getUnreadCount(targetId, conversationType, objectNames, callback)
```
#### 参数说明
| 参数             | 类型                             | 说明                                                    |
|------------------|----------------------------------|-------------------------------------------------------|
| targetId         | String                           | 会话 ID。                                                |
| conversationType | ConversationType                 | 会话类型。不适用于聊天室。                          |
| objectNames      | String[]                         | 消息类型标识数组。内置消息类型的标识可参见[消息类型概述]。 |
| callback         | IRongCoreCallback.ResultCallback | 回调结果。                                               |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

## 按会话类型获取总未读消息数

:::tip
- 从 5.20.0 版本开始，支持超级群类型会话未读数，超级群功能需[提交工单]开启。
:::

获取多个指定会话类型的未读数。
#### 接口
```java
RongIMClient.getInstance().getUnreadCount(conversationTypes, containBlocked, callback);
```
#### 参数说明

| 参数              | 类型                      | 说明                                                       |
|:------------------|:--------------------------|:---------------------------------------------------------|
| conversationTypes | [ConversationType] []     | 会话类型数组。不适用于聊天室、超级群。                        |
| containBlocked    | boolean                   | 是否包含消息免打扰的未读消息数。`true`：包含。`false`：不包含。 |
| callback          | ResultCallback\<Integer\> | 回调接口                                                   |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

#### 示例代码

```java
ConversationTypes[] conversationTypes = {ConversationTypes.PRIVATE, ConversationTypes.GROUP};
boolean containBlocked = true;

RongIMClient.getInstance().getUnreadCount(conversationTypes, containBlocked,
    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode ErrorCode) {

    }
});
```

## 按会话免打扰级别获取总未读消息数

:::tip

 SDK 从 5.2.5 版本开始支持该接口。仅在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中提供。
:::


获取已设置指定免打扰级别的会话的总未读消息数。SDK 将按照传入的免打扰级别配置查找会话，再返回这些所有会话的总未读消息数。
#### 接口

```java
ChannelClient.getInstance().getUnreadCount(conversationTypes, levels, callback)
```

#### 参数说明

| 参数              | 类型                       |                 必填                 |
|:------------------|:---------------------------|:----------------------------------:|
| conversationTypes | [ConversationType] []      |     会话类型数组。不适用于聊天室。     |
| levels            | [PushNotificationLevel] [] | 免打扰类型数组。详见[免打扰功能概述]。 |
| callback          | ResultCallback\<Integer\>  |               回调接口               |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

#### 示例代码

```java
ConversationTypes[] conversationTypes = {ConversationTypes.PRIVATE, ConversationTypes.GROUP};
PushNotificationLevel[] levels = {PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_MENTION}

ChannelClient.getInstance().getUnreadCount(conversationTypes, levels,

    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
});
```

## 按会话免打扰级别获取总未读 @ 消息数

:::tip

 SDK 从 5.2.5 版本开始支持该接口。仅在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中提供。
:::


获取已设置指定免打扰级别的会话的总未读 @ 消息数。SDK 将按照传入的免打扰级别配置查找会话，再返回这些所有会话的总未读 @ 消息数。
#### 接口
```java
ChannelClient.getInstance().getUnreadMentionedCount(conversationTypes, levels, callback)
```
#### 参数说明

| 参数              | 类型                       |                 必填                 |
|:------------------|:---------------------------|:----------------------------------:|
| conversationTypes | [ConversationType] []      |     会话类型数组。不适用于聊天室。     |
| levels            | [PushNotificationLevel] [] | 免打扰类型数组。详见[免打扰功能概述]。 |
| callback          | ResultCallback\<Integer\>  |               回调接口               |

获取成功后，`callback` 中会返回未读 @ 消息数（`unReadCount`）。
#### 示例代码
```java
ConversationTypes[] conversationTypes = {ConversationTypes.PRIVATE, ConversationTypes.GROUP};
PushNotificationLevel[] levels = {PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_MENTION}

ChannelClient.getInstance().getUnreadMentionedCount(conversationTypes, levels,

    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
});
```

## 清除指定会话未读数

按时间戳清除指定会话的未读数。SDK 会将该时间戳之前的消息的未读状态全部清除。
#### 接口
```java
RongIMClient.getInstance().clearMessagesUnreadStatus(conversationType, targetId,  timestamp, callback);
```

#### 参数说明

| 参数             | 类型               | 说明                                 |
|:-----------------|:-------------------|:-----------------------------------|
| conversationType | [ConversationType] | 会话类型。不适用于聊天室、超级群。      |
| targetId         | String             | 会话 ID                              |
| timestamp        | long               | 时间戳。此时间戳之前的未读消息都清除。 |
| callback         | OperationCallback  | 回调接口                             |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
long timestamp = 1585811571;

RongIMClient.getInstance().clearMessagesUnreadStatus(conversationType, targetId, timestamp, new OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

<!-- links -->
[消息类型概述]: /platform-chat-api/message-about/about-message-types
[免打扰功能概述]: ./do-not-disturb-about.md
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

