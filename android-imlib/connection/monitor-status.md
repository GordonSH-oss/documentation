---
title: 连接状态
sidebar_position: 50
---

# 连接状态

IMLib SDK 为 App 提供了 IM 连接状态监听器 `ConnectionStatusListener`。通过监听 IM 连接状态的变化，您可以进行不同业务处理，或在页面上给出提示。

## 连接状态监听器说明 {#status}

`ConnectionStatusListener` 接口定义如下：

```java
  public interface ConnectionStatusListener {
        void onChanged(ConnectionStatus status);
  }
```

当连接状态发生变化时，IMLib SDK 会通过 `onChanged()` 方法，将当前的连接状态回调给您。各状态的具体说明请参考下表。

| 状态名称 | 状态值 | 说明 |
| :--- | :--- | :--- |
| `NETWORK_UNAVAILABLE` | -1 | 网络不可用。 |
| `CONNECTED` | 0 | 连接成功。 |
| `CONNECTING` | 1 | 连接中。 |
| `UNCONNECTED` | 2 | 未连接状态，即应用没有调用过连接方法。 |
| `KICKED_OFFLINE_BY_OTHER_CLIENT` | 3 | 用户账号在其它设备登录，此设备被踢下线。 |
| `TOKEN_INCORRECT` | 4 | Token 过期时触发此状态。 |
| `CONN_USER_BLOCKED` | 6 | 用户被控制台封禁。 |
| `SIGN_OUT` | 12 | 用户主动断开连接的状态，详见[断开连接](./disconnect.md)。 |
| `SUSPEND` | 13 | 连接暂时挂起（多是由于网络问题导致），SDK 会在合适时机进行自动重连。 |
| `TIMEOUT` | 14 | 连接超时，SDK 将停止连接，用户需要做超时处理，再自行调用[连接接口](./connect.md)进行连接。 |

## 添加或移除连接状态监听器 {#addlistener}

SDK 支持设置多个监听器。建议在应用生命周期内设置。

为了避免内存泄露，请在不需要监听时，将设置的监听器移除。

#### 示例代码
```java
// 添加连接状态监听器 since 5.1.6
RongCoreClient.addConnectionStatusListener(listener);
// 移除连接状态监听器 since 5.1.6
RongCoreClient.removeConnectionStatusListener(listener);
```

## 获取当前连接状态

您可以通过 `getCurrentConnectionStatus` 方法，主动获取当前 IM 服务连接状态。


:::tip

获取连接状态时，如 SDK 刚好开始重连，可能会出现获取到的还是已连接的状态。
:::
#### 示例代码

```java
RongCoreClient.getInstance().getCurrentConnectionStatus()
```

