---
sidebar_position: 1
title: 聊天室概述
---

# 聊天室概述

[聊天室（Chatroom）](https://www.rongcloud.cn/product/chatroom)提供了一种不设用户上限，支持高并发消息处理的业务形态，可用于直播、社区、游戏、广场交友、兴趣讨论等场景。聊天室业务要点如下：

- App Key 下可创建的聊天室数量没有限制，单个聊天室成员数量没有限制。
- 聊天室具有自动销毁机制，默认情况下所有聊天室会在不活跃（连续时间段内无成员进出且无新消息）达到 1 小时后踢出所有成员并自动销毁，可延长该时间，也可配置为定时自动销毁。详见服务端文档[聊天室销毁机制](/platform-chat-api/chatroom/destroy-about)。
- 聊天室具有离线成员自动退出机制。满足默认预设条件时，融云服务端会踢出聊天室成员，详见[退出聊天室]。
- 聊天室本地消息会在退出聊天室时删除。**IM 旗舰版** 与 **IM 尊享版**客户可选择启用**聊天室消息云端存储**功能，将消息存储在融云服务端。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。
- 聊天室不具备离线消息转推送功能，只有在线的聊天室成员可接收聊天室消息。

## 客户端 UI 框架参考设计

聊天室产品暂不提供聊天室会话专用的 UI 组件。您可以参考以下 UI 框架设计了解聊天室的设计思路。

- 下图**聊天室**标签中为聊天室消息列表。
- 下图**聊天室管理**窗口中展示了聊天室支持的部分能力，如禁言、封禁、白名单等。

![(height=500)](../assets/chatroom-sample-ui.png)

## 服务配置

客户端 SDK 默认支持聊天室，不需要申请开通。

聊天室的部分基础功能与增值服务可以在控制台 IM 服务的[服务购买]和[服务配置]页面进行开通和配置。

## 聊天室功能接口

聊天室会话关系由融云负责建立并保持连接。SDK 提供加入、退出等部分聊天室管理接口。更多聊天室管理功能需要配合使用即时通讯服务端 API。下表描述了融云聊天室主要的功能接口。

| 功能分类                 | 功能描述                                                                                                                                                                                                                                                                                                                                                                                              | 客户端 API                                            | 融云服务端 API                                  |
|:---------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|:---------------------------------------------------|:--------------------------------------------|
| 创建与销毁聊天室         | 手动创建聊天室，或手动销毁聊天室。**注意**：客户端 SDK 无单独的创建聊天室 API。客户端不提供手动销毁聊天室 API。                                                                                                                                                                                                                                                                                            | 不提供该 API                                          | [创建房间]、[销毁房间]                           |
| 加入与退出聊天室         | 加入已存在的聊天室，请确保聊天室 ID 已存在。加入与退出聊天室仅客户端提供 API。**注意**：客户端有废弃接口可支持在聊天室不存在时创建聊天室再加入，但已不推荐使用。                                                                                                                                                                                                                                            | [加入聊天室]、[退出聊天室]                             | 不提供该 API                                    |
| 查询聊天室房间与用户信息 | <ul><li>查询聊天室房间的基础信息，包括聊天室 ID、名称、创建时间。</li><li>查询聊天室成员信息，支持获取聊天室成员用户 ID、加入时间，最多返回 500 个成员信息，支持按加入时间排序。</li></ul>                                                                                                                                                                                                                     | [查询聊天室信息][get-chatroom-info-client]            | [查询房间信息][get-chatroom-info-server]        |
| 聊天室保活               | 添加一个或多个聊天室到聊天室保活列表。在保活列表中的聊天室不会被融云服务端自动销毁。                                                                                                                                                                                                                                                                                                                    | 不提供该 API                                          | [保活房间]                                      |
| 聊天室属性管理           | <p>在指定聊天室中设置自定义属性。比如在语音直播聊天室场景中，利用此功能记录聊天室中各麦位的属性；或在狼人杀等卡牌类游戏场景中记录用户的角色和牌局状态等。</p><p>聊天室属性以 Key-Value 的方式进行存储，支持设置、删除与查询属性，支持批量和强制操作。</p>                                                                                                                                                     | [聊天室属性]                                          | [属性管理（KV）][chatroom-kv]                     |
| 封禁/解封聊天室用户      | 封禁一个或多个聊天室成员。被封禁成员将被踢出指定聊天室，并在封禁时间内不能再进入此聊天室中。                                                                                                                                                                                                                                                                                                             | 不提供该 API                                          | [成员封禁]                                      |
| 聊天室用户白名单         | <p>需在 IM 服务的[服务购买]页面**选择增值服务**时，添加**聊天室消息白名单服务**，开通后可使用。**IM 旗舰版**或**IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。</p><p> 用户被加入某个聊天室的白名单后，在该聊天室消息量较大的情况下，该用户发送的消息不会被丢弃；并且用户也不会被融云服务端自动踢出该聊天室。</p>                         | 不提供该 API                                          | [聊天室白名单服务]                              |
| 发送聊天室消息           | 发送聊天室消息。                                                                                                                                                                                                                                                                                                                                                                                       | [发送消息][send-chatroom-msg-client]                  | [发送聊天室消息][send-chatroom-msg-server]      |
| 撤回聊天室消息           | 撤回聊天室消息。                                                                                                                                                                                                                                                                                                                                                                                       | [撤回消息][recall-msg-client]                         | [消息撤回][recall-msg-server]                   |
| 获取聊天室历史消息       | 获取聊天室历史消息。                                                                                                                                                                                                                                                                                                                                                                                   | [获取聊天室历史消息][get-chatroom-history-msg-client] | [历史消息日志][get-chatroom-history-msg-server] |
| 聊天室低级别消息         | <p>需在 IM 服务的[服务购买]页面**选择增值服务**时，添加**聊天室消息优先级服务**。**IM 旗舰版**或**IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。</p><p>如果消息类型在低级别消息列表中，该类型的消息全部视为低级别消息。当服务器负载高时，高级别的消息优先保留，低级别消息则优先丢弃。默认情况下，所有消息均为高级别消息。</p> | 不提供该 API                                          | [聊天室消息优先级服务]                          |
| 聊天室消息白名单         | <p>需在 IM 服务的[服务购买]页面**选择增值服务**时，添加**聊天室白名单服务**。**IM 旗舰版**或**IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。</p><p>如果消息类型在聊天室消息白名单中，该类型的消息全部受到保护，在聊天室消息量较大的情况下也不会被丢弃。</p>                                                           | 不提供该 API                                          | [聊天室白名单服务]                              |
| 聊天室成员禁言           | 在指定的某个聊天室中，禁言一个或多个成员。聊天室成员被禁言后，可以接收并查看聊天室中用户聊天信息，但不能通过往该聊天室内发送消息。                                                                                                                                                                                                                                                                         | 不提供该 API                                          | [单人禁言]                                      |
| 全体成员禁言             | 设置某一聊天室全部成员禁言，或取消指定聊天室全部成员禁言状态。设置全体群成员禁言后，该聊天室的所有成员均不能通过客户端 SDK 往该群组内发送消息。                                                                                                                                                                                                                                                           | 不提供该 API                                          | [全体禁言]                                      |
| 全体禁言白名单           | 添加一个或多个群成员到聊天室全体成员禁言白名单。聊天室成员被添加到白名单后，即使该聊天室处于全体成员禁言状态，该成员仍可通过客户端 SDK 往该聊天室发送消息。                                                                                                                                                                                                                                               | 不提供该 API                                          | [全体禁言]                                      |
| 全局禁言用户       | <p>需在 IM 服务的[服务购买]页面**选择增值服务**时，添加**聊天室全局禁言服务**。**IM 旗舰版**或**IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。</p><p>添加一个或多个用户到聊天室全局禁言列表中，列表中的用户在应用下的所有聊天室中都无法发送消息。</p>                                                                  | 不提供该 API                                          | [全局禁言]                                      |

## 与群组和超级群的区别

您可以通过以下文档了解业务类型之间的区别及所有功能：

- [即时通讯开发指导 · 业务类型介绍](/guides/realtime-chat/intro-chat#功能特性)
- [IM 尊享版、IM 旗舰版功能对照表](https://help.rongcloud.cn/t/topic/121)

<!-- 链接区域 -->
<!-- Client SDK -->
[加入聊天室]: ./join.md
[退出聊天室]: ./quit.md
[get-chatroom-info-client]: ./get-chatroom-info.md
[send-chatroom-msg-client]: ../message/send.md
[recall-msg-client]: ../message/recall.md
[get-chatroom-history-msg-client]: ./get-history-messages.md
[聊天室属性]: ./chatroomKV.md
<!-- Server API -->
[创建房间]: /platform-chat-api/chatroom/create
[销毁房间]: /platform-chat-api/chatroom/destroy
[保活房间]: /platform-chat-api/chatroom/keep-alive/add-to-keep-alive
[send-chatroom-msg-server]: /platform-chat-api/message/send-chatroom
[recall-msg-server]: /platform-chat-api/message/msgrecall
[get-chatroom-history-msg-server]: /platform-chat-api/message/get-message-history-log
[chatroom-kv]: /platform-chat-api/chatroom/kv-attribute/set-kv-entry
[get-chatroom-info-server]: /platform-chat-api/chatroom/get
[聊天室消息优先级服务]: /platform-chat-api/chatroom/message-priority/add-low-priority-message-type
[聊天室白名单服务]: /platform-chat-api/chatroom/whitelist/whitelist-service-about
[成员封禁]: /platform-chat-api/chatroom/block/block-user
[全体禁言]: /platform-chat-api/chatroom/mute/ban-chatroom
[单人禁言]: /platform-chat-api/chatroom/mute/gag-user
[全局禁言]: /platform-chat-api/chatroom/mute/gag-user-globally
<!-- 控制台 -->
[服务购买]: https://console.rongcloud.cn/agile/im/service/purchase

[服务配置]: https://console.rongcloud.cn/agile/im/service/config


