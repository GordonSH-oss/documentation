---
title: 监听频道状态变更
sidebar_position: 50
---

# 监听频道状态变更

客户端可设置监听，在超级群频道发生类型变更（仅适用于私有频道）、成员变更、频道删除等情况时收到取对应通知。

- 由于频道类型、用户是否在私有频道成员列表等差异，通知用户的范围会有差异。
- 客户端 SDK 接收到频道删除（解散）、私有频道成员列表变更通知后，会根据具体变化清理本地数据。

## 设置频道状态变化通知

1. 在 SDK 初始化后，连接 IM 成功前调用 `io.rong.imlib.ChannelClient#setUltraGroupChannelListener` 方法, 添加频道变更监听:

    ```java
    ChannelClient.getInstance().setUltraGroupChannelListener();
    ```

1. 实现 `IRongCoreListener.UltraGroupChannelListener` 接口，包括私有频道成员变更通知，频道类型变更通知，频道解散通知。

    ```java
    /** 私有频道时，私有频道成员列表用户被踢通知。 此时私有频道成员列表内用户才能收到此事件。公有频道时，不会触发此回调 */
    void ultraGroupChannelUserDidKicked(List<UltraGroupChannelUserKickedInfo> infoList);

    /** 频道属性改变时有频道时的回调。私有变公有时，私有频道成员列表内会收到此事件。公有变私有时，所有成员都能收到此事件 */
    void ultraGroupChannelTypeDidChanged(List<UltraGroupChannelChangeTypeInfo> infoList);

    /** 频道被解散通知 公有频道时，删除频道通知频道中所有人。私有频道时，删除频道通知私有频道成员列表中所有人。*/
    void ultraGroupChannelDidDisbanded(List<UltraGroupChannelDisbandedInfo> infoList);
    ```


## 频道类型变更的通知

超级群的频道按类型区分为公有频道与私有频道。超级群频道类型变更仅可通过 App 服务端调用服务端 API 实现。客户端 SDK 不提供接口。

频道类型发生变更时，SDK 通过以下方法通知 App：

#### 接口

```java
void ultraGroupChannelTypeDidChanged(List<UltraGroupChannelChangeTypeInfo> infoList);
```

#### 参数说明

- 参数说明

    | 返回值   | 返回类型                        | 说明         |
    |:---------|:--------------------------------|:-----------|
    | infoList | UltraGroupChannelChangeTypeInfo | 频道变更信息 |

- 频道变更类型说明

    | 枚举值                                                        | 说明                                                      |
    |:--------------------------------------------------------------|:--------------------------------------------------------|
    | ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PUBLIC_TO_PRIVATE             | 超级群公有频道变成了私有频道                              |
    | ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PRIVATE_TO_PUBLIC             | 超级群私有频道变成了公有频道                              |
    | ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PUBLIC_TO_PRIVATE_USER_NOT_IN | 超级群公有频道变成了私有频道，但是当前用户不在该私有频道中 |

:::tip
- **公有频道变私有频道**：所有用户都会收到通知。但根据用户是否在私有频道成员列表中，收到的通知有差异：

    - 在私有频道成员列表内的用户，收到的变更类型是 `ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PUBLIC_TO_PRIVATE`。
    - 不在私有频道成员列表的用户，收到的变更类型是 `ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PUBLIC_TO_PRIVATE_USER_NOT_IN`。

    如有需要，App 可在公有频道变私有频道前，提前指定用户加入私有频道成员列表。

- **私有频道变公有频道**：仅在私有频道成员列表的用户会收到通知，变更类型为 `ULTRA_GROUP_CHANNEL_CHANGE_TYPE_PRIVATE_TO_PUBLIC`。
:::


## 频道已删除（解散）的通知

超级群频道删除（解散）仅可通过 App 服务端调用服务端 API 实现。客户端 SDK 不提供接口。

删除频道时，SDK 通过以下方法通知 App：

#### 接口

```java
void ultraGroupChannelDidDisbanded(List<UltraGroupChannelDisbandedInfo> infoList);
```

#### 参数说明

- 参数说明

    | 返回值   | 返回类型                       | 说明         |
    |:---------|:-------------------------------|:-----------|
    | infoList | UltraGroupChannelDisbandedInfo | 频道变更信息 |

### 频道已删除的通知范围

- **删除公有频道的通知**：所有用户会收到通知。
- **删除私有频道的通知**：仅在私有频道成员列表的用户会收到通知。

### 如何清理本地数据

IMLib SDK 收到通知后会清理删除用户本地会话，但会保留本地会话中的消息。

## 私有频道成员列表变更的通知

超级群私有频道成员列表仅可通过服务端 API 变更。客户端 SDK 不提供接口。

私有频道成员列表发生变化时，SDK 通过以下方法通知 App：

#### 接口

```java
void ultraGroupChannelUserDidKicked(List<UltraGroupChannelUserKickedInfo> infoList);
```

#### 参数说明

- 参数说明：

    | 返回值   | 返回类型                        | 说明         |
    |:---------|:--------------------------------|:-----------|
    | infoList | UltraGroupChannelUserKickedInfo | 频道变更信息 |

### 私有频道成员列表变更的通知范围

- 如果频道类型为私有频道，将用户从私有频道成员列表移除时，仅通知被移除的用户
- 如果频道类型为公有频道，将用户从私有频道成员名单移除时，不发送通知。注意，该列表在该公有频道变更类型为私有频道时才会生效。

### 如何清理本地数据

- **当前登录用户被移出私有频道成员列表**
    - （SDK \< 5.4.0）：客户端 SDK 收到通知后会从会话列表中删除本地会话，但会保留本地会话的消息。
    - （SDK ≧ 5.4.0）：客户端 SDK 收到通知后会不从会话列表中删除本地会话，App 可以自行决定是否需要删除会话及会话中的消息。
- **其他情况**：如果当被移出列表的用户非本端登录用户，客户端 SDK 收到通知后不做任何处理。

