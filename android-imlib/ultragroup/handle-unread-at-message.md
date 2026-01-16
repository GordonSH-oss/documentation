---
title: 处理未读 @ 消息
sidebar_position: 130
---

# 处理未读 @ 消息

本文档描述了在超级群业务中如何实现获取全部未读 @ 消息、跳转到指定未读 @ 消息等功能。

:::tip

 处理未读 @ 消息的功能依赖融云服务端保存的超级群会话**未读 @ 消息摘要**数据。**@所有人**消息的摘要数据固定存储 7 天（不支持调整）。其他类型 @ 消息的摘要数据与超级群历史消息存储时长一致。
:::


## 获取全部未读 @ 消息

:::tip

 IMLib SDK 版本 ≧ 5.2.5。
:::

IMLib SDK 支持从远端获取指定超级群频道中的全部未读 @ 消息的摘要信息（`MessageDigestInfo`）列表。您可以通过返回的摘要信息，从远端取得未读的 @ 消息。

例如，您希望获取仅展示未读 @ 消息的场景，步骤如下：

1. 获取超级群频道下的未读 @ 消息摘要，最多可获取 50 条（`count` 取值范围为 [1-50]）。成功回调中会返回 `MessageDigestInfo` 的列表。每个 `MessageDigestInfo` 对象包含一条未读消息的摘要信息。使用 `MessageDigestInfo` 中的摘要信息构造 `Message` 列表。每个消息对象需包含从 `MessageDigestInfo` 中获取的  `ConversationType`，`targetId`，`channelId`，`messageUid`，`sentTime`。

#### 示例代码

    ```java
    String targetId = "会话 Id";
    String channelId = "频道 ID";
    long sendTime = 1662542712112L;
    int count = 20;

    ChannelClient.getInstance().getUltraGroupUnreadMentionedDigests(targetId, channelId, sendTime, count,
        new IRongCoreCallback.ResultCallback<List<MessageDigestInfo>>() {
            @Override
            public void onSuccess(List<MessageDigestInfo> messageDigestInfos) {
                List<Message> msgList = new ArrayList<>();
                for (MessageDigestInfo info : messageDigestInfos) {
                    // 可使用 MessageDigestInfo 的属性筛选 @ 消息摘要
                    Message message = new Message();
                    message.setConversationType(info.getConversationType());
                    message.setTargetId(info.getTargetId());
                    message.channelId = info.getChannelId();
                    message.setUId(info.getMessageUid());
                    message.setSentTime(info.getSentTime());
                    // 从 5.6.0 开始，MessageDigestInfo 携带消息类型的唯一标识 ObjectName
                    message.setObjectName(info.getObjectName());
                    msgList.add(message);
                }}

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

            }});
      ```

2. 您可以使用 `getBatchRemoteUltraGroupMessages(msgList, callback)` 方法，传入上一步构建的消息列表，从远端批量提取 @ 消息的完整内容。

    ```java
    ChannelClient.getInstance().getBatchRemoteUltraGroupMessages(msgList,
        new IRongCoreCallback.IGetBatchRemoteUltraGroupMessageCallback(
            @Override
            public void onSuccess(List<Message> matchedMsgList, List<Message> notMatchedMsgList) {}

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode errorCode) {}

        ));
    ```

## 定位到指定未读 @ 消息

:::tip

 要求 SDK 版本 ≧ 5.2.5。
:::


利用获取超级群会话未读 @ 消息摘要列表接口（`getUltraGroupUnreadMentionedDigests`），可以实现从会话列表进入会话页面后定位到特定 @ 消息的位置。

1. 获取超级群频道下的未读 @ 消息摘要，最多可获取 50 条（`count` 取值范围为 [1-50]）。每个 `MessageDigestInfo` 对象包含一条未读 @ 消息的摘要信息。

    ```java
    public void getUltraGroupUnreadMentionedDigests(
            String targetId,
            String channelId,
            long sendTime,
            int count,
            IRongCoreCallback.ResultCallback<List<MessageDigestInfo>> callback);
    ```

1. 构造 `HistoryMessageOption` 对象用于获取历史消息。从 `MessageDigestInfo` 中取出 `sendTime`，作为 `HistoryMessageOption` 对象的 `DataTime`。

    ```java
    HistoryMessageOption option = new HistoryMessageOption();
    option.setOrder(HistoryMessageOption.PullOrder.DESCEND);
    option.setCount(count);
    option.setDataTime(sendTime);
    ```

1. （可选）修正 `HistoryMessageOption` 的 `dataTime`。`getMessages` 的查询结果中不会包含 message.sentTime == dataTime 的消息。如有需要，可根据 `PullOrder` 分别对 `dataTime` 做以下修正，以保证查询结果包含您想要的消息：

    - HistoryMessageOption.PullOrder.DESCEND: dataTime = message.sentTime + 1
    - HistoryMessageOption.PullOrder.ASCEND : dataTime = message.sentTime - 1

1. 使用超级群 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 的获取历史消息接口 `getMessages`，可以获取特定 @ 消息前、后的历史消息。

    ```java
    public void getMessages(
            final Conversation.ConversationType conversationType,
            final String targetId,
            final String channelId,
            final HistoryMessageOption historyMessageOption,
            final IRongCoreCallback.IGetMessageCallbackEx getMessageCallback);
    ```

## 何时清空未读 @ 消息

未读数需要在会话页面清除:

- 会话页面即将消失时
- 处于会话页面, app返回前台时
- 已经加载到最新消息时

清除方法参考[清除消息未读状态]。

:::tip

 当调用 `syncUltraGroupReadStatus` 成功后，客户端本地 `Conversation` 的 `firstUnreadMsgSendTime` 会变为 0。服务端保存的第一条未读时间会变为 0，保存的未读 @ 消息摘要列表也会清零。
:::

[清除消息未读状态]: /android-imlib/ultragroup/sync-read-status

