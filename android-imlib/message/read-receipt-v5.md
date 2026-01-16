---
title: 逐条消息已读功能
sidebar_position: 90
---

:::important 重要提示
- 逐条消息已读功能（消息已读 V5）自 SDK 5.20.0 版本起支持。
- 自 2025.11.25 起，新建的 AppKey 且 SDK 版本 ≥ 5.20.0 时，默认开启逐条消息已读功能，[单群聊已读回执功能](/android-imlib/message/read-receipt-1v1-group)将失效。
- 如果您希望使用原来的单群聊已读回执功能，请[提交工单](https://console.rongcloud.cn/agile/formwork/ticket/create)申请切换。
:::

逐条消息已读功能（消息已读 V5）支持对每条接收消息分别设置已读状态，支持单聊和群聊会话类型。

## 发送消息

如需支持已读回执，发送消息时需在 `Message` 对象中设置 `needReceipt` 为 `true`。

### 示例代码
```java
Message message = ...;
message.setNeedReceipt(true); // 设置为 true

RongCoreClient.getInstance().sendMessage(message, null, null, new IRongCoreCallback.ISendMessageCallback() {
    /**
     * 消息发送前回调，回调时消息已存储至数据库。
     * @param message 已存库的消息体。
     */
    @Override
    public void onAttached(Message message) {
    }
    /**
     * 消息发送成功。
     * @param message 发送成功后的消息体。
     */
    @Override
    public void onSuccess(Message message) {
    }
    /**
     * 消息发送失败。
     * @param message   发送失败的消息体。
     * @param errorCode 具体错误码。
     */
    @Override
    public void onError(Message message, IRongCoreEnum.CoreErrorCode coreErrorCode) {
    }
});
```

### 消息对象新增属性

| 参数         | 类型     | 说明                 |
| :----------- | :------- | :------------------- |
| `needReceipt`  | `Boolean`  | 是否支持发送已读回执 |
| `sentReceipt`  | `Boolean`  | 是否已发送已读回执   |

## 发送已读回执

消息接收方在阅读某条消息后，需主动向发送方发送已读回执。调用 `sendReadReceiptResponseV5()` 方法，需传入会话标识（`identifier`）和消息唯一 ID。

| 参数         | 类型                       | 说明                 |
| :----------- | :------------------------- | :------------------- |
| `identifier`   | `ConversationIdentifier`     | 消息所属会话标识     |
| `messageUIds`  | `List`                      | 需发送已读回执的消息 UID 列表 |
| `callback`     | `IRongCoreCallback.OperationCallback` | 结果回调 |

### 示例代码

```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(
    Conversation.ConversationType.GROUP,
    "targetId",
    ""
);
List<String> messageUIds = new ArrayList<>();
messageUIds.add("MessageUID");

RongCoreClient.getInstance().sendReadReceiptResponseV5(identifier, messageUIds, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
    }
});
```

## 接收已读回执回调

消息接收方发送已读回执后，消息发送方会收到回执回调。开发者可注册已读回执监听器 `addMessageReadReceiptV5Listener()`，实现 `onMessageReceiptResponse()` 方法以接收回执信息。

| 参数      | 类型      | 说明                                 |
| :-------- | :-------- | :----------------------------------- |
| responses | List      | 消息已读回执响应信息 `ReadReceiptResponseV5` 列表 |

### 示例代码
```java
RongCoreClient.getInstance().addMessageReadReceiptV5Listener(new IRongCoreListener.MessageReadReceiptV5Listener() {
    @Override
    public void onMessageReceiptResponse(List<ReadReceiptResponseV5> responses) {
    }
});
```

## 获取消息已读回执信息

消息发送方可通过 `getMessageReadReceiptInfoV5()` 接口，传入会话标识（`identifier`）和消息 UID 查询对应消息的已读回执信息。

| 参数         | 类型                       | 说明                 |
| :----------- | :------------------------- | :------------------- |
| `identifier`   | `ConversationIdentifier`     | 消息所属会话标识     |
| `messageUIds`  | `List`                      | 需查询的消息 UID 列表 |
| `callback`     | `IRongCoreCallback.ResultCallback` | 结果回调 |

结果回调中，`infoV5s` 为 `ReadReceiptInfoV5` 列表，`ReadReceiptInfoV5` 包含如下内容：

| 参数         | 类型     | 说明     |
| :----------- | :------- | :------- |
| `messageUId`   | `String`   | 消息 UID |
| `unreadCount`  | `Int`      | 未读数   |
| `readCount`    | `Int`      | 已读数   |
| `totalCount`   | `Int`      | 总人数   |

#### 示例代码
```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(
    Conversation.ConversationType.GROUP,
    "targetId",
    ""
);
List<String> messageUIds = new ArrayList<>();
messageUIds.add("MessageUID");

RongCoreClient.getInstance().getMessageReadReceiptInfoV5(identifier, messageUIds, new IRongCoreCallback.ResultCallback<List<ReadReceiptInfoV5>>() {
    @Override
    public void onSuccess(List<ReadReceiptInfoV5> infoV5s) {
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
    }
});
```

## 批量获取多会话消息已读回执信息

消息发送方可通过 `getMessageReadReceiptInfoV5ByIdentifiers()` 接口，批量查询多会话消息的已读回执信息。

### 参数说明

| 参数         | 类型     | 说明                         |
| :----------- | :------- | :--------------------------- |
| `identifiers`  | `List`     | 消息标识 `MessageIdentifier` 列表 |
| `callback`     | `IRongCoreCallback.ResultCallback` | 结果回调 |

结果回调中，`infoList` 为 `ReadReceiptInfoV5` 列表，内容如下：

| 参数         | 类型     | 说明     |
| :----------- | :------- | :------- |
| `messageUId`   | `String`   | 消息 UID |
| `unreadCount`  | `Int`      | 未读数   |
| `readCount`    | `Int`      | 已读数   |
| `totalCount`   | `Int`      | 总人数   |

#### 示例代码

```java
List<MessageIdentifier> identifiers = new ArrayList<>();
MessageIdentifier identifier = new MessageIdentifier();
ConversationIdentifier conversationIdentifier = ConversationIdentifier.obtain(Conversation.ConversationType.PRIVATE, "targetId", "");
identifier.setIdentifier(conversationIdentifier);
identifier.setMessageUId("messageUId");
identifiers.add(identifier);

RongCoreClient.getInstance().getMessageReadReceiptInfoV5ByIdentifiers(identifiers, new IRongCoreCallback.ResultCallback<List<ReadReceiptInfoV5>>() {
    @Override
    public void onSuccess(List<ReadReceiptInfoV5> infoV5s) {
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
    }
});
```

## 分页获取已读回执用户列表

消息发送方可通过 `getMessagesReadReceiptUsersByPageV5()` 接口，分页获取消息的已读回执用户信息列表。

| 参数         | 类型                               | 说明                 |
| :----------- | :--------------------------------- | :------------------- |
| `identifier`   | `ConversationIdentifier`             | 消息所属会话标识     |
| `messageUId`   | `String`                             | 消息唯一 ID          |
| `option`       | `ReadReceiptUsersOption`             | 分页配置信息         |
| `callback`     | `IRongCoreCallback.ResultCallback`   | 结果回调             |

分页结果 `readReceiptUsersResult` 为 `ReadReceiptUsersResult` 的封装，其内容如下：

| 参数         | 类型     | 说明                             |
| :----------- | :------- | :------------------------------- |
| `users`        | `List`     | 查询到的用户（`ReadReceiptUser`）列表 |
| `pageToken`    | `String`   | 分页标识，若为空表示无下一页     |
| `totalCount`   | `Int`      | 用户总数                         |

### 示例代码

```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(
    Conversation.ConversationType.GROUP,
    "targetId",
    ""
);
ReadReceiptUsersOption option = new ReadReceiptUsersOption();
option.setPageToken("PAGE_TOKEN"); // 首次调用可传空
option.setPageCount(20);
option.setOrder(ReadReceiptUsersOption.Order.DESCEND);
option.setReadStatus(ReadReceiptUsersOption.ReadStatus.UNREAD);

RongCoreClient.getInstance().getMessagesReadReceiptUsersByPageV5(identifier, "messageUId", option, new IRongCoreCallback.ResultCallback<ReadReceiptUsersResult>() {
    @Override
    public void onSuccess(ReadReceiptUsersResult readReceiptUsersResult) {
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
    }
});
```

## 获取指定用户的已读回执信息

消息发送方可通过 `getMessagesReadReceiptByUsersV5()` 接口，获取指定用户的消息已读回执信息。

| 参数         | 类型                     | 说明         |
| :----------- | :----------------------- | :----------- |
| `identifier`   | `ConversationIdentifier`   | 消息所属会话标识 |
| `messageUId`   | `String`                   | 消息唯一 ID   |
| `userIds`      | `List`                     | 用户 ID 列表  |
| `callback`     | `IRongCoreCallback.ResultCallback` | 结果回调 |

结果 `readReceiptUsersResult` 为 `ReadReceiptUsersResult` 的封装，其内容如下：

| 参数         | 类型     | 说明                             |
| :----------- | :------- | :------------------------------- |
| `users`        | `List`     | 查询到的用户（`ReadReceiptUser`）列表 |
| `pageToken`    | `String`   | 分页标识，本接口中不生效，可忽略     |
| `totalCount`   | `Int`      | 用户总数，本接口中不生效，可忽略       |

### 示例代码

```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(
    Conversation.ConversationType.GROUP,
    "targetId",
    ""
);
List<String> userIds = new ArrayList<>();
userIds.add("UserId1");

RongCoreClient.getInstance().getMessagesReadReceiptByUsersV5(identifier, "messageUId", userIds, new IRongCoreCallback.ResultCallback<ReadReceiptUsersResult>() {
    @Override
    public void onSuccess(ReadReceiptUsersResult readReceiptUsersResult) {
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
    }
});
```
