---
title: 输入状态
sidebar_position: 190
---

# 输入状态

IMLib SDK 可以向超级群中发送当前用户输入状态。超级群内收到通知的用户可以在 UI 展示 “ xxx 正在输入”。

:::tip

 为保证最佳体验，建议在仅在人数小于 10,000 的超级群中使用该功能。
:::


## 发送输入状态

您可以在当前用户输入文本时调用 `sendUltraGroupTypingStatus`，发送当前用户输入状态。

#### 接口

```java
enum UltraGroupTypingStatus{
	//正在输入文本
	UltraGroupTypingStatusText = 0
}

class OperationCallback {
	onSuccess()
  onError(ErrorCode code)
}
public void sendUltraGroupTypingStatus(final String targetId, final String channelId, final IRongCoreEnum.UltraGroupTypingStatus typingStatus, final IRongCoreCallback.OperationCallback callback)

```
#### 参数说明
|     参数     |   类型   |                        说明                        |
|:------------:|:--------:|:------------------------------------------------:|
|   targetId   | String  |         超级群会话 targetId。          |
|  channelId  | String |                  超级群频道 channelId。        |
|  status  | UltraGroupTypingStatus |                  输入状态类型。                 |
| callback |  IRongCoreCallback.OperationCallback   |                     接口回调                     |


## 监听输入状态

为了减少服务端和客户端的压力，服务端会将一段时间内（例如 5 秒）用户输入事件汇总之后统一批量下发，所以回调以数组形式提供。应用程序收到通知时可以在 UI 展示 “ xxx 正在输入”。

```java
class UltraGroupTypingStatusInfo {
  String targetId
  String channelId
  String userId
  UltraGroupTypingStatus status
  long timestamp //服务端收到用户操作的上行时间.
}

interface UltraGroupTypingStatusListener {
      void onUltraGroupTypingStatusChanged(List<UltraGroupTypingStatusInfo> infoList);
  }

public void setUltraGroupTypingStatusListener(IRongCoreListener.UltraGroupTypingStatusListener listener)
```

