---
title: 获取频道列表
sidebar_position: 40
---

# 获取频道列表

您有多种方式来获取频道列表，您可以根据实际情况使用客户端 SDK 与融云服务端 API 提供的能力，采取灵活的方式维护超级群的频道列表。

:::tip

 - 如果您的应用/环境在 2022.10.13 日及以后开通超级群服务，超级群业务中会包含一个 ID 为 `RCDefault` 的默认频道。如果发消息时不指定频道 ID，则该消息会发送到 `RCDefault` 频道中。在获取 `RCDefault` 频道的历史消息时，需要传入该频道 ID。
 - 如果您的应用/环境在 2022.10.13 日前已开通超级群服务，在发送消息时如果不指定频道 ID，则该消息不属于任何频道。获取历史消息时，如果不传入频道 ID，可获取不属于任何频道的消息。融云支持客户调整服务至最新行为。该行为调整将影响客户端、服务端收发消息、获取会话、清除历史消息、禁言等多个功能。如有需要，请提交工单咨询详细方案。
:::


## 如何获取超级群全部频道列表

超级群下全部频道的列表可通过融云服务端 API (`/ultragroup/channel/get.json`) 获取。

注意，对单个用户来说，最多可以加入 100 个超级群，在每个超级群中最多可以加入或者创建 50 个频道。

:::tip

  IMLib SDK 会通过频道中收发的消息在本地数据库中生成一个超级群频道列表。该列表仅包含了已在本地产生消息的频道，可能并非该超级群下全部频道的列表。

:::

建议从 App 业务服务端维护超级群的频道列表。App 业务一般需要知晓当前用户的所加入的超级群，以及超级群下的频道列表，才能实现特定的业务功能。假设同一超级群下的不同用户需要展示个性化的频道列表（例如 App 需要在 UI 上展示用户标星的特定频道），则需要为用户保存超级群频道列表。

APP 服务端也可以按照需求将超级群分组（如**超级群概述**中的 UI 框架设计所示）。

## 获取本地指定超级群下的频道列表

IMLib SDK 会根据频道中收发的消息在本地数据库中生成对应频道的会话。您可以从本地数据库获取 IMLib SDK 生成的频道列表。获取到的频道列表按照时间倒序排列。

#### 示例代码

```java
ChannelClient.getInstance()
        .getConversationListForAllChannel(
                Conversation.ConversationType.ULTRA_GROUP,
                groupId,
                new IRongCoreCallback.ResultCallback<List<Conversation>>() {
                    @Override
                    public void onSuccess(List<Conversation> conversations) {
                        if (conversations == null) return;
                        for (Conversation conversation : conversations) {
                            //获取会话的未读消息数
                            int unreadMessageCount = conversation.getUnreadMessageCount();
                            //获取会话的 @ 未读消息数
                            int unreadMentionedCount =
                                    conversation.getUnreadMentionedCount();
                            //获取会话的 @ 我的未读消息数 @since 5.4.5
                            int mentionedMeCount = conversation.getUnreadMentionedMeCount();

                        }
                        //该回调在非 UI 线程返回，如果需要做 UI 操作，请切换至 UI 线程
                    }

                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode e) {}
                });
```

## 获取本地指定超级群特定类型频道列表

:::tip

 SDK 从 5.2.4 开始支持按类型获取频道列表。
:::

IMLib SDK 会根据频道中收发的消息在本地数据库中生成对应频道的会话。您可以从本地数据库获取 IMLib SDK 生成的频道列表。获取到的频道列表按照时间倒序排列：


**频道类型**（`channelType`）支持超级群公有频道或私有频道。

#### 接口

```java
/**
 * 获取超级群频道列表
 *
 * @param targetId 超级群 id
 * @param channelType 频道类型
 * @param callback 结果回调
 */
public abstract void getUltraGroupChannelList(
        String targetId,
        IRongCoreEnum.UltraGroupChannelType channelType,
        IRongCoreCallback.ResultCallback<List<Conversation>> callback);
```

#### 参数说明

    - 频道类型（`channelType`）

        ```java
            /** 超级群频道类型 Added from version 5.2.4 */
            public enum UltraGroupChannelType {
                /** 超级群共有频道 */
                ULTRA_GROUP_CHANNEL_TYPE_PUBLIC,

                /** 超级群私有频道 */
                ULTRA_GROUP_CHANNEL_TYPE_PRIVATE;
            }
        ```

