---
title: 监听会话同步状态
sidebar_position: 70
---

# 监听会话同步状态

在 IMlib SDK 每一次连接 IM 服务成功时，IMLib SDK 都会自动从服务端拉取超级群会话列表与消息。

您可以通过设置超级群会话同步监听器，获取超级群会话列表与会话最后一条消息同步完成的通知，从而进行不同业务处理，例如刷新 UI 界面。


## 设置超级群会话同步监听器

:::tip

 从 SDK 5.2.2 版本开始支持该回调接口。
:::

在 IMLib SDK 初始化后，连接 IM 成功前调用 `setUltraGroupConversationListener` 方法，添加监听器，建议在应用生命周期内设置。

```java
// 超级群会话监听器
interface UltraGroupConversationListener {
   // 超级群会话列表与会话最后一条消息同步完成
   void ultraGroupConversationListDidSync();
}

public void setUltraGroupConversationListener(IRongCoreListener.UltraGroupConversationListener listener) {
}
```

