---
sidebar_position: 20
title: 用户级别配置
---

本文档旨在说明用户级别配置相关的接口。

## 用户级别配置事件回调

:::tip
从 5.24.0 版本开始支持。
:::

### 监听用户级别配同步事件

在用户连接成功后，SDK 会拉取当前用户的配置信息，拉取成功后，通过 `UserSettingsListener` 监听中的 `onUserSettingsSync` 回调通知 App。开发者需要通过 `RongCoreClient` 中的 `addUserSettingsListener: 接口注册监听。

#### 示例代码

1. 注册用户级别配置事件监听

```java
// 添加用户配置事件监听
RongCoreClient.getInstance().addUserSettingsListener(listener)

// 移除用户配置事件监听
RongCoreClient.getInstance().removeUserSettingsListener(listener)
```

2. 实现用户级别配置同步回调方法

```java
    /**
     * 用户级别事件代理。
     *
     * <p>可以用 CoreClient 的 getAppSettings 接口，查询相关配置 isUserSettingsEnabled。
     *
     */
    interface UserSettingsListener {
        /**
         * 用户级别的所有配置信息同步完成回调。
         *
         * <p>在连接成功后，SDK 会同步当前用户的所有配置，同步完成后回调该方法。
         *
         * @param code 错误码
         */
        void onUserSettingsSync(IRongCoreEnum.CoreErrorCode code);
    }
```