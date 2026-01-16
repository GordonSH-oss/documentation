---
title: 多端同步免打扰/置顶
sidebar_position: 140
---

# 多端同步免打扰/置顶

IMLib SDK 提供了会话状态（置顶或免打扰）同步机制，通过设置会话状态同步监听器，在其它端修改会话状态时，可在本端实时监听到会话状态的改变。

:::tip
在版本 `5.10.0` 之前，会有未读消息数不准确的情况，建议使用 `5.10.0` 或更高版本。
:::

## 监听器说明

`RongIMClient` 中提供了 `ConversationStatusListener` 监听器。设置监听后，在会话的状态（置顶和免打扰）改变时，会触发以下方法：

```java
public interface ConversationStatusListener {
  void onStatusChanged(ConversationStatus[] conversationStatus);
}
```

`onStatusChanged` 方法返回 `conversationStatus` 的列表，参数如下：

#### 参数说明

| 参数 | 类型 | 描述 |
| :------ | :------ | :------ |
| conversationType | [ConversationType] | 会话类型。 |
| targetId | String | 会话 ID。 |
| isTop | boolean | 会话是否被设置为置顶。 |
| level | PushNotificationLevel | 会话的免打扰级别。SDK 从 5.2.5 版本支持多端同步免打扰级别。具体级别说明详见**免打扰功能概述**。|
| notificationStatus | [ConversationNotificationStatus] | 会话提醒状态。仅支持提醒（`1`）或免打扰（`0`）两种状态。|

## 设置监听器

设置会话状态（置顶和免打扰）多端同步监听器。

#### 示例代码
```java
//同步监听器
RongIMClient.ConversationStatusListener listener = new RongIMClient.ConversationStatusListener() {
  @Override
  public void onStatusChanged(ConversationStatus[] conversationStatus) {
    if (conversationStatus == null) {
      return;
    }
    for (ConversationStatus status : conversationStatus) {
      Conversation.ConversationType conversationType = status.getConversationType(); //获取会话类型
      String targetId = status.getTargetId(); //获取会话 Id
      boolean isTop = status.isTop(); //获取该会话当前状态是否置顶
      IRongCoreEnum.PushNotificationLevel level = status.getNotificationLevel();// 获取PushNotificationLevel（5.2.5新增）
      Conversation.ConversationNotificationStatus notificationStatus = status.getNotifyStatus(); //获取该会话当前的免打扰状态。
    }
  }
};
RongIMClient.getInstance().setConversationStatusListener(listener); //设置监听器
```

<!-- links -->
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[ConversationNotificationStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-notification-status/index.html

