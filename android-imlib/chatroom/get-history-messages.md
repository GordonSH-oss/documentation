---
title: 获取聊天室历史消息
---

# 获取聊天室历史消息

客户端 SDK 支持获取聊天室历史消息。具体能力如下：

- 通过 [getHistoryMessages] 获取聊天室保存在本地的历史消息。请注意，聊天室业务默认在用户退出聊天室时会清除聊天室保存在本地的历史消息。
- 通过 [getChatroomHistoryMessages] 获取聊天室保存在服务端的历史消息。请注意，该功能依赖[聊天室消息云端存储](https://console.rongcloud.cn/agile/im/service/purchase#19)服务。

## 开通服务

使用 [getChatroomHistoryMessages] 方法要求开通 [聊天室消息云端存储](https://console.rongcloud.cn/agile/im/service/purchase#19)服务。使用前请确认已开通服务。开通后聊天室历史消息保存在云端，默认保存 2 个月。

## 获取聊天室远端历史消息

调用 [getChatroomHistoryMessages] 方法。如果本地数据库存在相同时段内的历史消息，那么调用这个接口会返回一个 size 为 0 的数组。因此建议的调用顺序如下：

1. 先调用 [RongCoreClient] 的 [getHistoryMessages]，传入会话类型，聊天室 ID、最后一条消息的 ID，并指定要获取的消息数量等参数，从本地消息数据库中获取的历史消息。
1. 如果为空的话，再调用 [RongChatRoomClient] 的 [getChatroomHistoryMessages] 方法，传入聊天室 ID、消息的发送时间戳、查询方向，并指定要获取的消息数量等参数，从聊天室消息云端存储中获取历史消息。在查询方向不变的情况下，当次返回的 `syncTime` 的值可以作为下次拉取时的 `recordTime` 传入，方便连续拉取。


#### 接口

```java
RongCoreClient.getInstance().getHistoryMessages(conversationType, targetId, oldestMessageId, count,callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `conversationType` | [ConversationType] | 会话类型 |
| `targetId` | String | 会话 ID |
| `oldestMessageId` | Long | 最后一条消息的 ID。如需查询本地数据库中最新的消息，设置为 `-1`。 |
| `count` | Int | 每页消息的数量。 |
| `callback` | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列 |


#### 示例代码

```java
Conversation.ConversationType conversationType = Conversation.ConversationType.CHATROOM;
int oldestMessageId = -1;
final int count = 10;

final String targetId = "聊天室 ID";
final long recordTime = 0;

RongCoreClient.getInstance().getHistoryMessages(conversationType, targetId, oldestMessageId, count,
                new IRongCoreCallback.ResultCallback<List<Message>>() {

                    @Override
                    public void onSuccess(List<Message> messages) {
                        if (messages == null || messages.isEmpty()) {
                           RongChatRoomClient.getInstance().getChatroomHistoryMessages(targetId, recordTime, count, IRongCoreEnum.TimestampOrder.RC_TIMESTAMP_ASC,
                                new IRongCoreCallback.IChatRoomHistoryMessageCallback() {

                                    @Override
                                    public void onSuccess(List<Message> messages, long syncTime) {

                                    }


                                    @Override
                                    public void onError(IRongCoreEnum.CoreErrorCode code) {

                                    }
                                });

                        }
                    }

                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode e) {

                    }
                });

```

[getChatroomHistoryMessages] 的成功回调中会返回一个 `syncTime`。`syncTime` 的具体意义与 `getChatroomHistoryMessages` 的 `order` 参数取值有关。

- 如果拉取顺序为 `RC_Timestamp_Desc`，表示查询方向为降序，即按消息发送时间递减的顺序，获取发送时间早于 `recordTime` 的消息。`syncTime` 为结果中最早一条消息的时间戳（即最小的时间戳）。
- 如果拉取顺序为 `RC_Timestamp_Asc`，表示查询方向为升序，即按消息发送时间递增的顺序，获取发送时间晚于 `recordTime` 的消息。`syncTime` 为结中最晚消息的时间戳（即最大的时间戳）。

<!-- api links -->
[RongCoreClient]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html
[RongChatRoomClient]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/index.html
[getHistoryMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-history-messages.html
[getChatroomHistoryMessages]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/get-chatroom-history-messages.html
[TimestampOrder]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-enum/-timestamp-order/index.html

