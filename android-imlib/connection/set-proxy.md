---
title: Socks 5 代理
unlisted: true
sidebar_position: 1000
---

# Socks 5 代理

客户端 SDK 支持设置 Socks5 代理。设置成功后，SDK 即可通过代理网络访连接融云服务器。

:::tip

 - 客户端 SDK 从 5.3.0 版本开始支持设置代理。目前仅支持 Socks5 代理。
 - 建议在 SDK 初始化 `init` 之前配置代理。
:::


## 设置代理

SDK 的代理设置保存在内存中，每次启动应用后需重新设置代理。

1. 创建 Socks5 代理配置。

    #### 示例代码
    ```java
    RCIMProxy proxy = new RCIMProxy(
            "10.10.10.10",
            1010,
            "userName",
            "password");

    ```

    | 参数 | 类型 | 必填 | 说明 |
    | :--- | :--- | :--- | :--- |
    | `host` | String | 是 | 代理服务器地址。|
    | `port` | Int | 是 | 端口。 |
    | `userName` | String | 否 | 代理服务器用户名。仅在 `userName` 和 `password` 均为有效值时才会进行认证。 |
    | `password` | String | 否 | 代理服务器密码。仅在 `userName` 和 `password` 均为有效值时才会进行认证。 |

2. （推荐）因 SDK 内部不会自动检测 `proxy` 的有效性，请在设置代理前测试代理服务是否可用。

    #### 示例代码
    ```java
    RongIMClient.getInstance()
            .testProxy(
                    proxy,
                    "www.example.com",
                    new RongIMClient.Callback() {
                        @Override
                        public void onSuccess() {
                            // 连接成功，代理可用
                        }

                        @Override
                        public void onError(RongIMClient.ErrorCode errorCode) {
                            // 失败，代理不可用或 testHost 返回异常
                        }
                    });

    ```

    | 参数 | 类型 | 说明 |
    | :--- | :--- | :--- |
    | `proxy` | RCIMProxy | 您的代理服务器。必须为有效值，否则返回失败。|
    | `testHost` | String | 测试地址，用于测试代理的连通性。如果您的 App 业务服务器需要通过代理访问，建议使用 App 服务器地址。 |
    | `callback` | Callback | 测试结果的回调。 |

3. 在测试代理返回成功后进行代理设置。如果代理 IP 或端口错误，SDK 将无法正常与融云建立连接。

    #### 示例代码
    ```java
    // 设置代理
    RongIMClient.getInstance().setProxy(proxy);
    ```

    `setProxy` 方法直接使用您配置的代理信息。SDK 在设置代理时不会自动检测 `proxy` 的有效性。

    :::tip

    - 重复多次设置代理时，仅最后设置的代理生效。
    - 请传入正确的代理配置。如果代理 IP 或端口错误，`setProxy` 方法会返回 `true`，直接使用参数错误的代理，导致 SDK 无法与融云正常建立连接。
    - 在 SDK 连接过程中、连接成功后无法设置代理，此时调用 `setProxy` 方法返回 `false`。
    :::


## 取消代理

SDK 的代理设置只保存在内存中，杀死 App 后代理就失效了。

如果 SDK 未与融云建立连接，直接将代理设置为空可取消代理设置。

#### 示例代码
```java
// 取消代理设置
RongIMClient.getInstance().setProxy(null);
```

:::tip

 如果需要在 SDK 建立连接后取消并重设代理，必须先调用 `disconnect` 断开连接，重设代理后再进行连接。
:::


## 获取当前代理

获取当前设置的代理。

#### 示例代码
```java
RongIMClient.getInstance().getCurrentProxy();
```

## 状态码

| 状态码 | 说明 |
| :--- | :--- |
| 31028 | 配置了 SDK 通过代理连接融云，但代理地址不可用。 |
| 34238 | 非法的代理配置。请检查代理是否为空或者是否传入了非法参数。 |
| 34239 | 传入的代理测试服务非法。  |
| 34240 | 代理地址或 `testHost` 地址无法连通。 |
