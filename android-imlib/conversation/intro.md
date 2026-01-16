---
title: 会话介绍
sidebar_position: 1
---

# 会话介绍

会话是指 IMLib SDK 根据每条消息的发送方、接收方以及会话类型等信息，自动建立并维护的逻辑关系，是一种抽象概念。

## 会话类型

融云支持多种会话类型，以满足不同业务场景需求。IMLib SDK 通过 `ConversationType` 枚举来表示各类型会话，各枚举值代表的含义参考下表：

| 枚举值                       | 会话类型   |
|------------------------------|--------|
| ConversationType.PRIVATE     | 单聊会话   |
| ConversationType.GROUP       | 群组会话   | <!--public-cloud-only start-->
| ConversationType.ULTRA_GROUP | 超级群会话 |
| ConversationType.CHATROOM    | 聊天室会话 | <!--public-cloud-only end-->
| ConversationType.SYSTEM      | 系统会话   |

`ConversationType` 枚举中还定义了其它会话类型，目前已废弃，不再维护。

### 单聊会话

指两个用户一对一进行聊天，两个用户间可以是好友也可以是陌生人，融云不对用户的关系进行维护管理，会话关系由融云负责建立并保持。

单聊类型会话里存储类型的消息会保存在客户端本地数据库中。

### 群组会话

群组指两个以上用户一起进行聊天，群组成员信息由 App 提供并进行维系，融云只负责将消息传达给群组中的所有用户。每个群最大人数上限为 3000 人，App 内的群组数量没有限制。

群组类型会话里存储类型的消息会保存在客户端本地数据库中。

<!--public-cloud-only start-->
### 超级群会话

超级群会话指无成员上限的多人聊天服务，海量消息并发即时到达，支持消息推送服务。群组成员信息由 App 提供并维系，融云负责将消息传达给群成员。App 内超级群数量没有限制，超级群无人数上限，一个用户可加入 100 个超级群。

超级群类型会话里的消息会保存在客户端本地数据库中，更多内容请参见[超级群概述](../ultragroup/overview.md)。

### 聊天室会话

聊天室成员不设用户上限，海量消息并发即时到达，用户退出聊天室后不会再接收到任何聊天室中的消息，没有推送通知功能。会话关系由融云负责建立并保持连接，通过 SDK 相关接口，可以让用户加入或者退出聊天室。

SDK 不保存聊天室消息，在退出聊天室时会清空此聊天室所有数据，更多内容请参见[聊天室概述](../chatroom/overview.md)。<!--public-cloud-only end-->

### 系统会话

系统会话是指利用系统帐号向用户发送消息从而建立的会话关系，此类型会话可以是通过调用广播接口发送广播来建立，也可以是加好友等单条通知消息而建立的会话。

## 会话实体类

客户端 SDK 中封装的会话实体类是 `Conversation`，所有会话相关的信息都从该实体类中获取。

下表列出了 `Conversation` 中提供的主要属性：

| 属性名                            | 类型                                | 描述                                                                                                                                                                                                                                                                                                      |
|-----------------------------------|-------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| targetId                          | String                              | 会话 ID（或目标 ID），用于标识会话对端。<ul><li>单聊时，会话 ID 直接使用对方的用户 ID。 </li><li>在群组、聊天室、超级群中，为对应的群组、聊天室、超级群 ID。</li><li>系统会话中，为开发者指定的系统账号 ID。</li></ul>                                                                                                  |
| channelId                         | String                              | 消息所属会话的业务标识，仅适用于超级群。                                                                                                                                                                                                                                                                    |
| portraitUrl                       | String                              | 会话中头像的 URL。                                                                                                                                                                                                                                                                                         |
| unreadMessageCount                | String                              | 会话中未读消息数。                                                                                                                                                                                                                                                                                         |
| isTop                             | boolean                             | 会话是否置顶。                                                                                                                                                                                                                                                                                             |
| isTopForTag                       | boolean                             | 当前会话在此 tag 下的置顶状态态。<br />1. 仅在通过 tag 获取会话时有效。                                                                                                                                                                                                                                                                                             |
| conversationTitle                 | String                              | 会话标题。                                                                                                                                                                                                                                                                                                 |
| conversationType                  | [ConversationType]                  | 会话类型，参考上文详细描述。                                                                                                                                                                                                                                                                                |
| latestMessage                     | [MessageContent]                    | 会话中在客户端本地存储的最后一条消息的消息内容。关于消息的存储属性请参考[消息介绍](../message/introduction.md)中关于 MessageTag 的说明。                                                                                                                                                                  |
| latestMessageId                   | int                                 | 会话中最后一条在客户端本地存储的消息的 ID。                                                                                                                                                                                                                                                                |
| draft                             | String                              | 会话里保存的草稿信息，参考[草稿详细说明](./draft.md)。                                                                                                                                                                                                                                                      |
| operationTime                      | long                                | 获取该会话的操作时间（Unix时间戳、毫秒），用于分页获取会话列表时传入的时间戳。<br />1. 初始值与 sentTime 相同，置顶等操作会更新此时间戳。                                                                                                                                                                                  |
| receivedTime                      | long                                | 会话中最后一条消息的接收时间。<br />1. 返回值为 Unix 时间戳，单位毫秒。<br />2. 接收时间为消息到达接收端时客户端的本地时间。                                                                                                                                                                                  |
| sentTime                          | long                                | 会话中最后一条消息的发送时间，为 Unix 时间戳，单位毫秒。<br />1. 当会话里最后一条消息为发送成功或者接收到的消息时，返回该消息到达融云服务器的时间。<br />2. 当会话里最后一条消息为发送失败的消息时，返回此条消息的本地发送时间。<br />3. 当会话有草稿信息，且草稿保存时间大于最后一条消息时间时，返回草稿保存时间。 |
| receivedStatus                    | [Message.ReceivedStatus]            | 会话中最后一条消息的接收状态。                                                                                                                                                                                                                                                                             |
| sentStatus                        | [Message.SentStatus]                | 会话中最后一条消息的发送状态。                                                                                                                                                                                                                                                                             |
| objectName                        | String                              | 会话中最后一条消息的类型名，与消息内容体对应。预定义消息类型的 objectName 参见[消息类型概述]。自定义消息类型的 objectName 为您自行指定的值。                                                                                                                                                                  |
| senderUserId                      | String                              | 会话中最后一条消息发送者 ID。                                                                                                                                                                                                                                                                              |
| senderUserName                    | String                              | 发送者名称。该字段仅供 SDK 内部使用。                                                                                                                                                                                                                                                                       |
| notificationStatus                | [ConversationNotificationStatus]    | 会话的免打扰状态。                                                                                                                                                                                                                                                                                         |
| mentionedCount                    | int                                 | 本会话里自己被 @ 的消息数量。                                                                                                                                                                                                                                                                              |
| mentionedMeCount                    | int                                 | 超级群会话里自己被 @ 的消息数量。                                                                                                                                                                                                                                                                              |
| latestMessageDirection            | [Message.MessageDirection]          | 会话中最后一条消息的方向，分为发送和接收。                                                                                                                                                                                                                                                                  |
| latestMessageExtra <!-- iOS 无--> | String                              | 会话中最后一条消息的附加信息。                                                                                                                                                                                                                                                                             |
| latestMessageUId                  | String                              | 会话中最后一条消息的唯一 ID。<br />1. 只有发送成功的消息才有唯一 ID。<br />2. 在同一个 Appkey 下全局唯一。                                                                                                                                                                                                   |
| latestMessageReadReceiptInfo      | [ReadReceiptInfo]                   | 会话中最后一条消息的阅读回执状态，仅适用于群聊。                                                                                                                                                                                                                                                            |
| latestMessageConfig               | [MessageConfig]                     | 会话中最后一条消息的配置信息。                                                                                                                                                                                                                                                                             |
| latestCanIncludeExpansion         | boolean                             | 会话中最后一条消息是否允许消息扩展。<br />1. 该属性在消息发送时确定，发送之后不能再做修改；<br />2. 扩展信息只支持单聊、群组和超级群，其它会话类型不能设置扩展信息。                                                                                                                                            |
| latestExpansion                   | `Map<String, String>`               | 会话中最后一条消息的扩展信息。详情请参考[消息扩展](/android-imlib/message/expansion)。                                                                                                                                                                                     |
| pushNotificationLevel             | int                                 | 会话免打扰级别，详见[免打扰功能概述](do-not-disturb-about)。                                                                                                                                                                                                                                                |
| channelType                       | IRongCoreEnum.UltraGroupChannelType | 从 5.2.4 开始，支持超级群频道类型，包含公有频道和私有频道。                                                                                                                                                                                                                                                  |
| firstUnreadMsgSendTime            | long                                | 从 5.2.5 版本开始，支持会话中第一条未读消息的时间戳属性，仅对超级群生效。                                                                                                                                                                                                                                    |

<!-- links -->
[消息类型概述]: /platform-chat-api/message-about/about-message-types
<!-- api links-->
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message.MessageDirection]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/-message-direction/index.html
[Message.SentStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/-sent-status/index.html
[Message.ReceivedStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/-received-status/index.html
[ReadReceiptInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-read-receipt-info/index.html
[ConversationNotificationStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-notification-status/index.html
[MessageConfig]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-config/index.html

