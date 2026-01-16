---
title: 断开连接
sidebar_position: 10
---

# 断开连接

连接融云服务后，在需要用户切换、用户注销的操作时，可通过下面方法断开与融云的 IM 连接，并可根据此方法来设置在用户断开连接后是否接收消息推送。

:::tip
IMLib SDK 在前后台切换或者网络出现异常时会自动重连，保证连接的可靠性。除非 App 逻辑需要登出，否则不需要调用此方法手动断开连接。
:::

:::tip
建议使用 `RongCoreClient` 替换 `RongIMClient`。
:::

## 断开连接（允许推送）

主动断开与融云服务端的 IM 连接，并设置断开连接后允许融云服务端进行远程推送。

### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `isReceivePush` | `Boolean` | 断开连接后，是否允许融云服务端进行远程推送。`true` 表示接收远程推送，`false` 表示不接收远程推送。 |

### 示例代码

```java
RongIMClient.getInstance().disconnect(isReceivePush);
```

或者：

```java
RongIMClient.getInstance().disconnect();
```

如果融云服务端检测到 App 客户端不在线（默认要求全部设备已下线），在接收新消息时，融云服务端会为该用户记录一条离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>，并同时触发融云服务端的推送服务。如您同时集成了融云提供的厂商推送通道，融云服务端会通过推送通道下发一条推送信息到客户端 SDK。该提醒一般以通知形式展示在通知面板，提示用户有离线消息。

## 断开连接（不允许推送）

需要注销登录（登出）或切换 App 用户账号时，推荐调用以下方法主动断开与融云服务端的 IM 连接，并设置断开连接后不允许融云服务端进行远程推送。

### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `isReceivePush` | `Boolean` | 断开连接后，是否允许融云服务端进行远程推送。`true` 表示接收远程推送，`false` 表示不接收远程推送。 |

### 示例代码

```java
RongIMClient.getInstance().disconnect(isReceivePush);
```

或者：

```java
RongIMClient.getInstance().logout();
```

断开连接且不允许推送的情况下，融云服务端仅记录离线消息，但不会为当前设备触发推送服务。如果用户登录了多个设备，则在其他设备中最后一个登录的设备上可正常接收推送。在多设备场景下，App 如需保证设备间消息记录一致，可通过开启**多设备消息同步**实现。详见[多设备消息同步](multiple-client-sync-config)。

## 断开连接（5.34.0 开始支持）

从 5.34.0 版本开始，`RongCoreClient` 仅提供 `disconnect` 和 `logout` 两种方式，其他接口已标记为废弃。

### 断开连接（允许推送）

```java
RongCoreClient.getInstance().disconnect();
```

### 断开连接（不允许推送）

断开与融云服务器的连接，并且不再接收远程推送消息。

```java
RongCoreClient.getInstance().logout(new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 断开连接成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
        // 断开连接失败
    }
});
```
