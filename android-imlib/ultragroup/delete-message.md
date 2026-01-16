---
title: 删除消息
sidebar_position: 150
---

# 删除消息

超级群会话消息存储在服务端（免费存储 7 天）和用户设备本地数据库。您可以通过IMLib SDK 删除自己的历史消息，支持仅从本地数据库删除消息、仅从融云服务端删除消息或者本地与服务端同时删除消息。

:::tip

 - 客户端的删除消息的操作均指从当前登录用户的历史消息记录中删除消息，不影响会话中其他用户的历史消息记录。
 - 如果 App 的管理员或者某普通用户希望在该 App 中彻底删除一条消息，例如在所有超级群成员的聊天记录中删除一条消息，应使用客户端或服务端的**撤回消息**功能。消息成功撤回后，原始消息内容会在所有用户的本地与服务端历史消息记录中删除。
:::


| 功能 | 本地/服务端 |API |
|:---|:---|:---|
| [从本地删除全部频道的消息（时间戳）](#local) | 仅从本地删除 | `deleteUltraGroupMessagesForAllChannel` |
| [从本地删除指定频道的消息（时间戳）](#localbychannel) | 仅从本地删除 | `deleteUltraGroupMessages` |
| [从服务端删除指定频道的消息（时间戳）](#all) |  仅从服务端删除  | `deleteRemoteUltraGroupMessages` |
| [从本地和远端删除消息（消息对象）](#by-message-local-remote) | 同时从本地和服务端删除  | `deleteRemoteMessage` |

## 从本地删除全部频道的消息（时间戳） {#local}

删除本地数据库删除所有频道指定时间戳之前的历史消息。需提供 Unix 时间戳，精确到毫秒。单次操作仅针对单个超级群，不支持一次删除多个超级群中的消息。

您可以使用 `deleteUltraGroupMessagesForAllChannel` 方法删除超级群会话全部频道指定时间戳之前的本地历史消息，需提供 Unix 时间戳，精确到毫秒。单次操作仅针对单个超级群，不支持一次删除多个超级群中的消息。
服务端保存的该用户的历史消息记录不受影响。如果该用户从服务端获取历史消息，可能会获取到在本地已删除的消息。

#### 接口原型

```java
ChannelClient.getInstance().deleteUltraGroupMessagesForAllChannel(targetId, timestamp,callback);
```

#### 参数说明

| 参数          | 类型        | 说明                                                                                                                                           |
|:--------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------|
| targetId       | String| 超级群会话的 targetId。    |
| timestamp      | long| 时间戳。删除该时间戳之前的消息，传入 0 表示删除该会话所有本地历史消息。    |
| callback    | IRongCoreCallback.ResultCallback\<Boolean\>      | 删除历史消息结果回调。|

#### 示例代码

```java
String targetId = "会话 Id";
String timestamp = 0;

ChannelClient.getInstance().deleteUltraGroupMessagesForAllChannel(targetId, timestamp,
    new IRongCoreCallback.ResultCallback<Boolean>() {
            /**
             * 成功回调
             */
            @Override
            public void onSuccess(Boolean bool) {

            }
            /**
             * 失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });

```

## 从本地删除指定频道的消息（时间戳） {#localbychannel}

您可以删除超级群会话指定频道指定时间戳之前的本地历史消息，需提供 Unix 时间戳，精确到毫秒。单次操作仅针对单个超级群，不支持一次删除多个超级群中的消息。
服务端保存的该用户的历史消息记录不受影响。如果该用户从服务端获取历史消息，可能会获取到在本地已删除的消息。

#### 接口

```java
ChannelClient.getInstance().deleteUltraGroupMessages(targetId, channelId, timestamp,callback);
```

#### 参数说明

| 参数          | 类型        | 说明                                                                                                                                           |
|:--------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------|
| targetId       | String| 超级群会话的 targetId。    |
| channelId      | String| 超级群频道的 channelId。    |
| recordTime      | long| 时间戳。删除该时间戳之前的消息，传入 0 表示删除该会话所有本地历史消息。    |
| callback    | IRongCoreCallback.ResultCallback\<Boolean\>      | 删除历史消息结果回调。|


#### 示例代码

```java
String targetId = "超级群 ID";
String channelId = "频道 ID";
String recordTime = 0;

ChannelClient.getInstance().deleteUltraGroupMessages(targetId, channelId, timestamp,
    new IRongCoreCallback.ResultCallback<Boolean>() {
            /**
             * 获取成功回调
             */
            @Override
            public void onSuccess(Boolean bool) {

            }
            /**
             * 失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });

```

## 从服务端删除指定频道的消息（时间戳） {#remote}


您可以删除超级群会话指定频道指定时间戳之前的远端历史消息，需提供 Unix 时间戳，精确到毫秒。单次操作仅针对单个超级群，不支持一次删除多个超级群中的消息。
客户端本地保存的该用户的历史消息记录不受影响。

:::tip

 在融云控制台为 App 启用超级群服务后，融云会自动启用历史消息存储（免费存储 7 天）。
:::

#### 接口

```java
ChannelClient.getInstance().deleteRemoteUltraGroupMessages(targetId, channelId, timestamp,callback);
```

#### 参数说明

| 参数          | 类型        | 说明                                                                                                                                           |
|:--------------|:------------|:---------------------------------------------------------------------------------------------------------------------------------------------|
| targetId       | String| 超级群会话的 targetId。    |
| channelId      | String| 超级群频道的 channelId。    |
| recordTime      | long| 时间戳。删除该时间戳之前的消息，传入 0 表示删除该会话所有远端历史消息。    |
| callback    | IRongCoreCallback.ResultCallback\<Boolean\>      | 删除历史消息结果回调。|

#### 示例代码

```java
String targetId = "超级群 ID";
String channelId = "频道 ID";
String timestamp = 0;

ChannelClient.getInstance().deleteRemoteUltraGroupMessages(targetId, channelId, timestamp,
    new IRongCoreCallback.ResultCallback<Boolean>() {
            /**
             * 获取成功回调
             */
            @Override
            public void onSuccess(Boolean bool) {

            }
            /**
             * 失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });

```

## 从本地和远端删除指定频道的消息（消息对象） {#by-message-local-remote}

如果您希望从自己的超级群会话的历史记录中彻底删除一组消息，可以使用 `deleteRemoteMessages` 方法，传入需要被删除的消息对象 [Message] 列表。

删除成功后，您无法从本地数据库获取消息。如果从服务端获取历史消息，也无法获取到已删除的消息。

#### 接口

```java
ChannelClient.getInstance().deleteRemoteMessages(conversationType, targetId, channelId, messages, callback);
```

#### 参数说明

|       参数       |                    类型                     | 说明                                                              |
|:----------------:|:-------------------------------------------:|:----------------------------------------------------------------|
| conversationType |             [ConversationType]              | 会话类型。不支持聊天室。                                            |
|     targetId     |                   String                    | 会话 ID                                                           |
|     channelId    |                   String                    | 超级群频道 ID ID                                                           |
|     messages     |                  Message[]                  | 要删除的消息对象 [Message] 数组。请确保所提供的消息均属于同一会话。 |
|     callback     | IRongCoreCallback.ResultCallback\<Boolean\> | 接口回调                                                          |

```java
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String targetId = "会话 Id";
Message[] messages = {message1, message2};

RongCoreClient.getInstance().deleteRemoteMessages(conversationType, targetId, messages, new IRongCoreCallback.OperationCallback() {
    /**
     * 删除消息成功回调
     */
    @Override
    public void onSuccess() {

    }
    /**
     * 删除消息失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html

