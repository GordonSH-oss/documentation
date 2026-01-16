---
title: 获取未读消息数
sidebar_position: 120
---

# 获取未读消息数

 获取未读消息数

IMLib SDK 支持您获取超级群下的未读消息数，具体可以获取的内容如下：

- 当前用户加入的所有超级群、指定超级群、或指定频道的未读消息数。
- 当前用户加入的所有超级群、指定超级群、或指定频道的未读 @ 消息数。
- 按免打扰级别获取总未读消息数。
- 返回的未读数最大值为 999，如果实际未读数超过 999，接口仍返回 999。

:::tip

 IMLib SDK 仅在 `ChannelClient` 中提供相关接口的调用。
:::

## 获取多个超级群的未读消息数

SDK 支持获取当前用户加入的所有超级群中未读消息数与未读 @ 消息数。

### 批量获取当前用户的超级群的未读消息数

:::tip
 IMLib SDK 从 5.4.6 版本开始支持此功能。
:::

在社群应用场景中，如果您需要实时显示用户所在的多个超级群下所有频道的最新未读消息数据，您可以使用 `getUltraGroupConversationUnreadInfoList` 一次获取最多 20 个超级群下所有频道的未读数据。具体包含：

- 超级群频道的未读消息数
- 超级群频道的未读 @ 消息数
- 超级群频道中仅 @ 当前用户的未读 @ 消息数
- 超级群频道的免打扰级别

#### 接口

```java
ChannelClient.getInstance().getUltraGroupConversationUnreadInfoList(targetIds,callback);
```

#### 参数说明
| 参数             | 类型                 | 说明                                                                  |
|:-----------------|:---------------------|:--------------------------------------------------------------------|
| targetIds         | `String[]`             | 超级群会话 targetId 的列表，最多 20 个。                                                               |
| callback       | ResultCallback\<List\<ConversationUnreadInfo\>\>                 | 接口回调 |


#### 示例代码

```java

ChannelClient.getInstance().getUltraGroupConversationUnreadInfoList(targetIds,

    new IRongCoreCallback.ResultCallback<List<ConversationUnreadInfo>>() {

    @Override
    public void onSuccess(List<ConversationUnreadInfo> ConversationUnreadInfos) {
        if (ConversationUnreadInfos == null) return;
        for (ConversationUnreadInfo unreadInfo : ConversationUnreadInfos) {
            // 获取超级群会话类型
            Conversation.ConversationType type = unreadInfo.getType();
            String targetId = unreadInfo.getTargetId();
            // 获取超级群频道 ID
            String channelId = unreadInfo.getChannelId();
            //获取频道内的未读消息数
            int unreadMessageCount = unreadInfo.getUnreadMessageCount();
            //获取频道内的 @ 未读消息数
            int unreadMentionedCount =
                    unreadInfo.getUnreadMentionedCount();
            //获取频道内仅自己被 @ 的未读消息数（@ 我）
            int mentionedMeCount = unreadInfo.getUnreadMentionedMeCount();
            //获取频道的免打扰级别设置
            IRongCoreEnum.PushNotificationLevel pushNotificationLevel = getPushNotificationLevel();

        }
        //该回调在非 UI 线程返回，如果需要做 UI 操作，请切换至 UI 线程

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
});
```

### 获取当前用户的超级群未读消息数总和

获取当前登录用户加入的超级群所有会话的未读消息数的总和。

#### 接口

```java
class ResultCallback {
  onSuccess(Integer count)
  onError(ErrorCode code)
}

public void getUltraGroupAllUnreadCount(final IRongCoreCallback.ResultCallback<Integer> callback)

```

### 获取当前用户的超级群未读 @ 消息数总和

获取当前登录用户加入的超级群所有会话的未读 @ 消息数的总和。

#### 接口

```java
class ResultCallback {
  onSuccess(Integer count)
  onError(ErrorCode code)
}

public void getUltraGroupAllUnreadMentionedCount(IRongCoreCallback.ResultCallback<Integer> callback) {}
```

## 获取单个超级群的未读消息数

SDK 支持获取指定超级群或频道中的未读消息数和未读 @ 消息数。

### 获取指定单个超级群的未读消息数

获取当前用户在指定超级群中的未读消息数。

#### 接口

```java
class ResultCallback {
  onSuccess(Integer count)
  onError(ErrorCode code)
}

public void getUltraGroupUnreadCount(String targetId, IRongCoreCallback.ResultCallback<Integer> callback) {}
```

### 获取指定单个超级群的未读 @ 消息数

获取当前用户在指定超级群中的未读 @ 消息数。

#### 接口

```java
class ResultCallback {
  onSuccess(Integer count)
  onError(ErrorCode code)
}

public void getUltraGroupUnreadMentionedCount(final String targetId, final IRongCoreCallback.ResultCallback<Integer> callback)

```

### 获取超级群会话指定频道的未读消息数

获取当前用户在超级群会话指定的频道中的未读消息数。

#### 接口

```java
class ResultCallback {
  onSuccess(Integer count)
  onError(ErrorCode code)
}

public void getUnreadCount(final Conversation.ConversationType conversationType,
            final String targetId,
            final String channelId,
            final IRongCoreCallback.ResultCallback<Integer> callback)

```

### 获取指定频道的未读 @ 消息数

App 直接从 `Conversation` 对象上获取该频道未读 @ 消息数。

- `getUnreadMentionedCount()`：当前频道中 “@所有人”与“@当前用户”的未读消息数之和。
- `getUnreadMentionedMeCount()`：当前频道中 “@当前用户”的未读消息数。要求 SDK 版本 ≧ 5.4.5。

具体示例可参见[获取频道列表](get-channel-list)。

### 按免打扰级别获取超级群的总未读消息数

:::tip

 SDK 从 5.2.5 版本开始支持该接口。仅在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中提供。
:::


获取已设置指定免打扰级别的会话及频道的总未读消息数。SDK 将按照传入的免打扰级别配置查找，再返回该会话下符合设置的频道的总未读消息数。

#### 接口

```java
ChannelClient.getInstance().getUltraGroupUnreadCount(targetId, levels,callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :---: |
| targetId | String   | 会话 ID   |
| levels | `PushNotificationLevel[]` | 免打扰类型数组。详见[免打扰功能概述]。 |
| callback | ResultCallback\<Integer\>  |  回调接口 |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。


#### 示例代码

```java
PushNotificationLevel[] levels = {PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_MENTION}

ChannelClient.getInstance().getUltraGroupUnreadCount(targetId, levels,

    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
});
```


### 按免打扰级别获取超级群的总未读 @ 消息数

:::tip

 SDK 从 5.2.5 版本开始支持该接口。仅在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 中提供。
:::


您可以获取已设置指定免打扰级别的会话及频道的总未读 @ 消息数。IMLib SDK 将按照传入的免打扰级别配置查找，再返回该会话下符合设置的频道的总未读 @ 消息数。

#### 接口

```java
ChannelClient.getInstance().getUltraGroupUnreadMentionedCount(targetId, levels,callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :---: |
| targetId | String   | 会话 ID   |
| levels | `PushNotificationLevel[]` | 免打扰类型数组。详见[免打扰功能概述]。 |
| callback | ResultCallback\<Integer\>  |  回调接口 |

获取成功后，`callback` 中会返回未读消息数（`unReadCount`）。

#### 示例代码

```java
PushNotificationLevel[] levels = {PushNotificationLevel.PUSH_NOTIFICATION_LEVEL_MENTION}

ChannelClient.getInstance().getUltraGroupUnreadMentionedCount(targetId, levels,

    new ResultCallback<Integer>() {

    @Override
    public void onSuccess(Integer unReadCount) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
});
```

如果您希望获取多个类型会话的未读消息总数，可参见[处理会话未读消息数](../conversation/handle-unread-count.md)中介绍的方法。


<!-- links -->
[免打扰功能概述]: ./do-not-disturb-about.md

