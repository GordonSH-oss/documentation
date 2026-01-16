---
title: 插入消息
sidebar_position: 50
---

# 插入消息

IMLib SDK 支持在本地数据库中插入消息。本地插入的消息**不会实际发送给融云服务器和对方**。

:::tip

 - 所有插入消息的 [MessageTag] 必须设置为 `MessageTag.ISPERSISTED`，否则报参数错误异常。详见[了解消息存储策略](../message/introduction.md#了解消息存储策略)。
 - 插入消息的接口仅将消息插入本地数据库，所以当 App 卸载重装或者换端登录时插入的消息不会从远端同步到本地数据库。

:::


## 插入单条发送消息
您可以通过 `insertOutgoingMessage:` 接口在本地数据库插入一条对外发送的消息。以下示例使用 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 下的 `insertOutgoingMessage` 方法。

:::tip
- 如果 sentTime 有问题会影响消息排序，请谨慎使用！
- 此接口不支持聊天室会话类型。
:::

#### 接口

```java
ChannelClient.getInstance().insertOutgoingMessage(conversationType, targetId, "", true, sentStatus, content, sentTime, callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:----- |
| type | [ConversationType] | 会话类型。|
| targetId | String  | 会话 ID。 |
| channelId | String  | 超级群频道 ID。会话类型为单聊、群聊时传空字符串即可。 |
| canIncludeExpansion | boolean  | 是否支持消息扩展。`true` 表示可扩展；`false` 表示不可扩展。 |
| sentStatus | [Message.SentStatus] |发送状态。 |
| content | [MessageContent] |消息内容。 |
| sentTime | long | 消息的发送时间，Unix 时间戳（毫秒）。传 0 会按照本地时间插入。|
| callback | IRongCoreCallback.ResultCallback\<Message\>|回调接口。|

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";
SentStatus sentStatus = SentStatus.SENT;
TextMessage content = TextMessage.obtain("这里是消息内容");
long sentTime = System.currentTimeMillis();

ChannelClient.getInstance().insertOutgoingMessage(conversationType, targetId, "", true, sentStatus, content, sentTime, new IRongCoreCallback.ResultCallback<Message>() {

        /**
         * 成功回调
         * @param message 插入的消息
         */
        @Override
        public void onSuccess(Message message) {

        }

        /**
         * 失败回调
         * @param errorCode 错误码
         */
        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

        }
    });
```

插入本地数据库的消息如需支持消息扩展功能，必须使用 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 下的插入方法，否则无法打开消息的可扩展属性（`canIncludeExpansion`）。本地插入消息时不支持设置消息扩展信息数据。App 可以选择合适的时机为消息设置扩展数据。详见[消息扩展]。


## 插入单条接收消息

您可以通过 `insertIncomingMessage:` 接口在本地数据库被插入一条接收的消息。

#### 接口

```java
RongCoreClient.getInstance().insertIncomingMessage(conversationType, targetId, senderUserId, receivedStatus, content, sentTime, callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------  |:------------------ |
| conversationType| [ConversationType] | 会话类型。|
| targetId | String  | Target ID 用于标识会话，称为目标 ID 或会话 ID。**注意**，因为单聊业务中始终使用对端用户 ID 作为标识本端会话的 Target ID，因此在单聊会话中插入本端接收的消息时，Target ID 始终是单聊对端用户 ID。群聊、超级群的会话 ID 分别为群组 ID、超级群 ID。详见[消息介绍](./introduction.md)中关于 Target ID 的说明。|
| senderUserId|String| 发送方的用户 ID。|
| receivedStatus| [Message.ReceivedStatus] |接收状态。|
| content| [MessageContent] |消息内容。|
| sentTime|long|消息的发送时间，Unix 时间戳（毫秒）。传 0 会按照本地时间插入。|
| callback|IRongCoreCallback.ResultCallback\<Message)\>|回调接口。|

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";
String senderUserId = "模拟发送方的 ID";
ReceivedStatus receivedStatus = new ReceivedStatus(0x1);
TextMessage content = TextMessage.obtain("这是一条插入数据");
long sentTime = System.currentTimeMillis();

RongCoreClient.getInstance().insertIncomingMessage(conversationType, targetId, senderUserId, receivedStatus, content, sentTime, new IRongCoreCallback.ResultCallback<Message>() {
        /**
         * 成功回调
         * @param message 插入的消息
         */
        @Override
        public void onSuccess(Message message) {

        }

        /**
         * 失败回调
         * @param errorCode 错误码
         */
        @Override
        public void onError(RongIMClient.ErrorCode errorCode) {

        }
 });
```


## 插入单条消息

本地会话中插入一条消息，消息不会实际发送给服务器和对方。不支持聊天室会话类型。

#### 接口

```java
ChannelClient.getInstance().insertMessage(message, callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------  |:------------------ |
| message| [Message] | 消息。|
| callback|IRongCoreCallback.ResultCallback\<Message)\>|回调接口。|


#### 示例代码

```java
        Conversation.ConversationType conversationType = Conversation.ConversationType.PRIVATE;
        String targetId = "会话 Id";
        String senderUserId = "模拟发送方的 ID";
        Message.SentStatus sentStatus = Message.SentStatus.SENT;
        Message.ReceivedStatus receivedStatus = new Message.ReceivedStatus(0x1);
        TextMessage content = TextMessage.obtain("这是一条插入数据");
        Message.MessageDirection direction = Message.MessageDirection.SEND;
        long sentTime = System.currentTimeMillis();
        Message message = Message.obtain(targetId, conversationType, content);
        message.setSentTime(sentTime);
        message.setReceivedStatus(receivedStatus);
        message.setSenderUserId(senderUserId);
        message.setSentStatus(sentStatus);
        message.setMessageDirection(direction);
        ChannelClient.getInstance().insertMessage(message, new IRongCoreCallback.ResultCallback<Message>() {
            /**
             * 成功回调
             * @param message 插入的消息
             */
            @Override
            public void onSuccess(Message message) {

            }
            /**
             * 失败回调
             * @param e 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode e) {

            }
        });
```

## 批量插入消息

:::tip

 - 从 5.1.1 版本 IMLib SDK 开始支持批量插入消息功能。
 - 由于批量插入本地数据库失败时会使数据整体插入失败，建议分批插入。单次操作最多插入 500 条消息，建议单次插入 10 条。
 - 该功能不支持聊天室会话类型，不支持超级群会话类型。

:::

`Message` 的下列属性会被入库，其余属性会被抛弃：

- `UId`：消息全局唯一 ID，消息成功收发后会带有由服务端生成的全局唯一 ID。SDK 从 5.3.5 开始支持入库该属性，该字段一般用于迁移数据。
- `conversationType`：会话类型。
- `targetId`：会话 ID。
- `messageDirection`：消息方向。
- `senderUserId`：发送者 ID。
- `receivedStatus`：接收状态；如果消息方向为接收方且未调用 `message.getReceivedStatus().setRead()`，该条消息为未读消息。未读消息数会累加到会话的未读消息数上。
- `sentStatus`：发送状态。
- `content`：消息的内容。
- `sentTime`：消息发送的 Unix 时间戳，单位为毫秒，会影响消息排序。
- `extra`：消息的附加信息

#### 接口

```java
RongCoreClient.getInstance().batchInsertMessage(messages, callback);
```

#### 参数说明


| 参数 | 类型 | 说明 |
|:---|:---|:---|
| messages | `List<Message>`    | 要插入的消息数组。单次操作最多插入 500 条消息，建议单次插入 10 条。 注意，`message` 中必须包含正确有效的 `sentTime`（消息发送的 Unix 时间戳，单位为毫秒），否则无法通过获取历史消息接口从数据库中获取该消息。       |
| callback | IRongCoreCallback.ResultCallback\<Boolean\> |回调接口|


#### 示例代码

```java
RongCoreClient.getInstance().batchInsertMessage(messages, new IRongCoreCallback.ResultCallback<Message>() {
        /**
         * 成功回调
         */
        @Override
        public void onSuccess(Boolean success) {


        }

        /**
         * 失败回调
         * @param errorCode 错误码
         */
        @Override
        public void onError(RongIMClient.ErrorCode errorCode) {

        }
 });
```


批量插入消息提供以下重载方法，设置 `enableCheck` 为 `true` 后，SDK 会在插入时检查当前 UID 是否与数据库中 UID 重复，并移除重复项。注意，App 应自行保证插入的消息数组内部无重复的 UID。

```java
    public abstract void batchInsertMessage(
            final List<Message> messages,
            boolean enableCheck,
            final IRongCoreCallback.ResultCallback<Boolean> callback);
```

<!-- links -->
[消息扩展]: ./expansion.md
<!-- api links -->
[MessageTag]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-message-tag/index.html
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message.SentStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/-sent-status/index.html
[Message.ReceivedStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/-received-status/index.html

