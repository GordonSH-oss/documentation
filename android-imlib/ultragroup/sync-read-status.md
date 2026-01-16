---
title: 清除消息未读状态
sidebar_position: 140
---

# 清除消息未读状态

超级群业务可在多个客户端之间同步消息阅读状态。

## 同步消息已读状态

调用同步已读状态接口会同时清除本地与服务端记录的消息的未读状态，同时服务端会将最新状态同步给同一用户账号的其他客户端。

- 如果指定了频道 ID（`channelId`），则标记该频道所有消息为全部已读，并同步其他客户端。
- 如果频道 ID 为空，则标记该超级群会话下所有不属于任何频道的消息为全部已读，并同步其他客户端。

:::tip

 超级群暂不支持按时间戳同步已读状态。调用 `syncUltraGroupReadStatus` 会按指定参数的要求标记全部消息为已读。时间戳参数（`timestamp`）未使用，可传入任意数字。
:::

#### 接口

```java
class OperationCallback {
	void onSuccess()
  void onError(ErrorCode code)
}

public void syncUltraGroupReadStatus(final String targetId, final String channelId, final long timestamp, final IRongCoreCallback.OperationCallback callback)

```

#### 参数说明
|  参数  |   类型    |    说明   |
| :----- | :------- | :--- |
| targetId   | String   | 超级群会话 targetId。   |
| channelId    | String     | 超级群会话频道 channelId。    |
| timestamp  | long    | 该字段无效，可传入任意数字。 |
| callback | OperationCallback|接口回调|


## 监听消息已读时间

您可以通过设置 `setUltraGroupReadTimeListener` 监听器，来监听超级群已读同步的通知：


```java
interface UltraGroupReadTimeListener {
  /**
   超级群已读时间同步

   @param targetId 会话 ID
   @param channelId 频道 ID
   @param readTime 已读时间（服务端将未读数清零的时间）
   */
    void onUltraGroupReadTimeReceived(String targetId, String channelId, long time);
}

public void setUltraGroupReadTimeListener(IRongCoreListener.UltraGroupReadTimeListener listener)

```

<!-- links -->


