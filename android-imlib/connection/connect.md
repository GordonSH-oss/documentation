---
title: 连接
sidebar_position: 1
---

# 连接

应用客户端成功连接到融云服务器后，才能使用融云即时通讯 SDK 的收发消息功能。

融云服务端在收到客户端发起的连接请求后，会根据连接请求里携带的用户身份验证令牌（Token），判断是否允许用户连接。

## 准备工作

- 通过服务端 API [注册用户（获取 Token）]。融云客户端 SDK 不提供获取 Token 方法。应用程序可以调用自身服务端，从融云服务端获取 Token。
    - 取得 Token 后，客户端可以按需保存 Token，供后续连接时使用。具体保存位置取决于应用程序客户端设计。如果 Token 未失效，就不必再向融云请求 Token。
    - Token 有效期可在控制台进行配置，默认为永久有效。即使重新生成了一个新 Token，未过期的旧 Token 仍然有效。Token 失效后，需要重新获取 Token。如有需要，可以主动调用服务端 API [作废 Token](/platform-chat-api/user/expire)。
- 建议应用程序在连接之前[设置连接状态监听](./monitor-status.md)。
- SDK 已完成初始化。

:::tip

 **请不要在客户端直接调用服务端 API 获取 Token**。获取 Token 需要提供应用的 App Key 和 App Secret。客户端如果保存这些凭证，一旦被反编译，会导致应用的 App Key 和 App Secret 泄露。所以，请务必确保在应用服务端获取 Token。
:::


## 连接聊天服务器

请根据应用的业务需求与设计，自行决定合适的时机（例如登录、注册等），向融云聊天服务器发起连接请求。

请注意以下几点：

- 必须在 SDK 初始化之后，调用 `connect` 方法进行连接，否则可能没有回调。
- SDK 本身有重连机制，在一个应用生命周期内不要多次调用 `connect`，否则可能触发多个回调，导致回调被清除。
- 在应用主进程内调用一次即可。

### 接口（可设置超时）

客户端用户首次连接聊天服务器时，建议您调用带连接超时时间（`timeLimit`）的接口，并设置超时秒数。在网络较差等导致连接超时的情况下，您可以利用接口的超时错误回调，并在 UI 上提醒用户，例如建议客户端用户等待网络正常的时候再次连接。

一旦连接成功，SDK 的重连机制将立即开始生效，并接管所有的重连处理。当因为网络原因断线时，SDK 会不停重连直到连接成功为止，您不需要做额外的连接操作。如果应用主动断开连接，将退出重连机制，详见[断开连接](./disconnect.md)。其他情况请参见[重连机制与重连互踢](./reconnect-kick.md)。

如果后续离线登录，建议将 `timeLimit` 参数设置为 `0`，可以在回调数据库打开（`onDatabaseOpened`）后就进行页面跳转，优先展示本地历史数据。连接逻辑则完全托管给 SDK。

#### 接口

```java
RongIMClient.connect(token, timeLimit, connectCallback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `token` | String | 从服务端获取的 Token。 |
| `timeLimit` | Int | 超时时间（单位：秒）。超时后不再重连。取值 ≦ 0 则将一直连接，直到连接成功或者发生业务错误。 |
| `connectCallback` | ConnectCallback | 连接回调。详见[连接回调方法说明](#connectcallback)。 |

- **`timeLimit` 参数详细说明**：

    - `timeLimit` ≦ 0，IM 将一直连接，直到连接成功或者发生业务错误（如 Token 非法）。
    - `timeLimit` > 0，IM 将最多连接 `timeLimit` 秒。如果在 `timeLimit` 秒内无法连接成功则不再进行重连，通过回调 `onError` 告知连接超时，您需要再自行调用 `connect` 接口。

#### 示例代码

```java
public connectIM(String token) {
  boolean isCachedLogin == getLoginStatusFromSP(context); // 伪代码，从 sharedpreference 里读取该用户是否为免密登录
  int timeLimit = 0;
  if(!isCachedLogin) {
    timeLimit = 5;
  }
  RongIMClient.connect("用户Token", timeLimit, new RongIMClient.ConnectCallback() {
    @Override
    public void onDatabaseOpened(RongIMClient.DatabaseOpenStatus code) {
      if (RongIMClient.DatabaseOpenStatus.DATABASE_OPEN_SUCCESS.equals(code)) {
        // 本地数据库打开，跳转到会话列表页面
      } else {
        // 数据库打开失败，可以弹出 toast 提示。
      }
    }

    @Override
    public void onSuccess(String s) {
      // 连接成功，如果 onDatabaseOpened() 时没有页面跳转，也可在此时进行跳转。
    }

    @Override
    public void onError(RongIMClient.ConnectionErrorCode errorCode) {
      if （errorCode.equals(RongIMClient.ConnectionErrorCode.RC_CONN_TOKEN_EXPIRE)) {
        // 从 APP 服务请求新 token，获取到新 token 后重新 connect()
      } else if (errorCode.equals(RongIMClient.ConnectionErrorCode.RC_CONNECT_TIMEOUT)) {
        // 连接超时，弹出提示，可以引导用户等待网络正常的时候再次点击进行连接
      } else {
        // 其它业务错误码，请根据相应的错误码作出对应处理。
      }
    }
  });
}
```

### 接口（带用户 ID）

:::tip
此接口从开发版 5.20.0 或稳定版 5.7.5 开始支持。
:::

较上面的连接接口增加用户 ID 参数，可在发生连接异常时通过用户 ID 进行问题排查，后续还将支持通过用户 ID 进行导航缓存，相比其他连接接口完成耗时更短，推荐使用此接口。

#### 接口

```java
RongCoreClient.connect(final String token,
                       final int timeLimit,
                       final String userId,
                       final IRongCoreCallback.ConnectCallback connectCallback);
```

:::tip

 开发者须确保此用户 ID 匹配 Token 对应的用户 ID，否则可能造成问题排查干扰。
:::

### 接口（无超时设置）

如果客户端用户已成功登录过您的应用，且连接过融云聊天服务器，后续离线登录时建议使用此接口。

此时因用户已有历史数据，并不需要强依赖于连接成功。可以在回调数据库打开（`onDatabaseOpened`）后就进行页面跳转，优先展示本地历史数据。连接逻辑完全托管给 SDK 即可。

:::tip

 当网络较差时，有可能长时间不会回调。
:::


调用此接口后，SDK 的重连机制将立即开始生效，并接管所有的重连处理。SDK 会不停重连直到连接成功为止，您不需要做额外的连接操作。如果应用主动断开连接，将退出重连机制，详见[断开连接](./disconnect.md)。其他情况请参见[重连机制与重连互踢](./reconnect-kick.md)。

#### 接口

```java
RongIMClient.connect(token, connectCallback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `token` | String | 从服务端获取的用户身份令牌（Token）。 |
| `connectCallback` | ConnectCallback | 连接回调。详见下文[连接回调方法说明](#connectcallback)。 |

### 连接回调方法说明 {#connectcallback}

连接回调 `connectCallback` 提供了以下三个回调方法：

- `onDatabaseOpened(DatabaseOpenStatus code)`

  本地数据库打开状态回调。当回调 `DATABASE_OPEN_SUCCESS` 时，说明本地数据库打开，此时可以拉取本地历史会话及消息，适用于离线登录场景。

- `onSuccess(String userId)`

  连接成功的回调，返回当前连接的用户 ID。

- `onError(ConnectionErrorCode errorCode)`

  连接失败并返回对应的连接错误码，您需要参考[连接状态码](#connectstatus)进行不同业务处理。

  常见错误如下：

  - SDK 没有初始化即调用 `connect()` 方法。

  - 应用客户端与应用服务器的 App Key 不一致，导致 `TOKEN_INCORRECT` 错误。

    融云的每个应用都提供用于隔离生产和开发环境的两套独立 App Key / Secret。由测试环境切换到生产环境时，很容易发生应用客户端与应用服务端 App Key 不一致的问题。

  - Token 已过期。参见[获取 Token]。

## 连接状态码 {#connectstatus}

下表列出了连接相关的状态码。更多状态码，详见[状态码](/android-imlib/code)。

| 名称 | 值 | 说明 |
| :--- | :--- | :--- |
| `IPC_DISCONNECT` | -2 | IPC 进程意外终止。<br/>可能原因：<br/>1. 手机系统策略，导致 IPC 进程被回收或被解绑，应用层调用融云 IM 接口时会触发此问题，SDK 会做好自动重连，应用不需要额外处理。<br/>2. 找不到对应 CPU 架构的 `libRongIMLib.so` 或 `libsqlite.so`，请确保集成了对应架构的 so。 |
| `RC_CONN_ID_REJECT` | 31002 | App Key 错误，请检查您使用的 App Key 是否正确。 |
| `RC_CONN_TOKEN_INCORRECT` | 31004 | Token 无效。<br/>请检查客户端初始化使用的 App Key 和您服务器获取 Token 时使用的 App Key 是否一致。|
| `RC_CONN_NOT_AUTHRORIZED` | 31005 | App 校验未通过（开通了 App 校验功能，但是校验未通过）。 |
| `RC_CONN_APP_BLOCKED_OR_DELETED` | 31008 | 应用被封禁或已删除。请检查您使用的 App Key 是否被封禁或已删除。 |
| `RC_CONN_USER_BLOCKED` | 31009 | 用户被封禁。请检查您使用的 Token 是否正确，以及对应的 `UserId` 是否被封禁。 |
| `RC_CONN_TOKEN_EXPIRE` | 31020 | Token 过期。<br/>您在控制台设置的 Token 过期了，需要请求您的服务器重新获取 Token 并用新的 Token 建立连接。|
| `RC_CONN_PROXY_UNAVAILABLE` | 31028 | 代理服务不可访问。 |
| `RC_CLIENT_NOT_INIT` | 33001 | SDK 没有初始化。在使用 SDK 任何功能之前，必须先初始化。 |
| `RC_CONNECTION_EXIST` | 34001 | 连接已存在，或正在重连中。 |
| `RC_CONNECT_TIMEOUT` | 34006 | SDK 内部连接超时。调用可设置超时的连接接口，并设置有效的 `timeLimit` 值时会出现该错误。<br/>SDK 不会继续重连，需要应用调用 `connect` 接口进行连接。|
| `UNKNOWN` | -1 | 未知错误。|

<!-- 链接区域 -->
[注册用户（获取 Token）]: /platform-chat-api/user/register
[获取 Token]: /platform-chat-api/user/register#获取-token

