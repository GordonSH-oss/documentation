---
title: 获取历史消息
sidebar_position: 100
---

# 获取历史消息

获取历史消息时，您可以选择仅从本地数据中获取、仅从远端获取或者同时从本地与远端获取。
超级群会话没有离线消息，如果想获取用户离线时候的消息需要您根据超级群会话最后一条消息去获取远端的历史消息。

> - 如果您的应用/环境在 2022.10.13 日及以后开通超级群服务，超级群业务中会包含一个 ID 为 `RCDefault` 的默认频道。如果发消息时不指定频道 ID，则该消息会发送到 `RCDefault` 频道中。在获取 `RCDefault` 频道的历史消息时，需要传入该频道 ID。
> - 如果您的应用/环境在 2022.10.13 日前已开通超级群服务，在发送消息时如果不指定频道 ID，则该消息不属于任何频道。获取历史消息时，如果不传入频道 ID，可获取不属于任何频道的消息。融云支持客户调整服务至最新行为。该行为调整将影响客户端、服务端收发消息、获取会话、清除历史消息、禁言等多个功能。如有需要，请提交工单咨询详细方案。

## 获取本地与远端历史消息

您可以通过 [getMessages] 方法先从本地获取历史消息，本地有缺失的情况下会从服务端同步缺失的部分。当本地没有更多消息的时候，会从服务端拉取。

#### 接口

```java
interface IGetMessageCallbackEx{
    void onComplete(List<Message> messageList, long syncTimestamp, boolean hasMoreMsg, IRongCoreEnum.CoreErrorCode errorCode);
    void onFail(IRongCoreEnum.CoreErrorCode errorCode);
}

public void getMessages(final Conversation.ConversationType conversationType, final String targetId, final String channelId, final HistoryMessageOption historyMessageOption, final IRongCoreCallback.IGetMessageCallbackEx callback)
```

#### 参数说明

| 参数             | 类型                                                    | 说明                    |
|:-----------------|:--------------------------------------------------------|:----------------------|
| conversationType | [ConversationType]                                      | 会话类型                |
| targetId         | String                                                  | 会话 ID                 |
| channel          | String                                                  | 超级群频道 ID           |
| historyMsgOption | HistoryMessageOption                                    | 获取历史消息的配置选项。 |
| callback         | IRongCoreCallback.IGetMessageCallback\<List\<Message\>> | 获取历史消息的回调。     |

- **`HistoryMessageOption` 说明**：

    | 参数      | 说明                                                                                                                                                                                                                                              |
    |:----------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | dataTime  | 时间戳，用于控制分页查询消息的边界。默认值为 `0`。                                                                                                                                                                                                   |
    | count     | 要获取的消息数量。如果 SDK \< 5.4.1，范围为 [2-20]；如果 SDK ≧ 5.4.1，范围为 [2-100]；默认值为 `5`。                                                                                                                                                    |
    | pullOrder | 拉取顺序。`DESCEND`：降序，按消息发送时间递减的顺序，获取发送时间早于 `dataTime` 的消息，返回的列表中的消息按发送时间从新到旧排列。`ASCEND`： 升序，按消息发送时间递增的顺序，获取发送时间晚于 `dataTime` 的消息，返回的列表中的消息按发送时间从旧到新排列。 |

## 从本地数据库中获取消息

您可以通过 `getHistoryMessages` 方法分页查询指定会话存储在本地数据库中的历史消息，并获取到异步返回的消息对象列表。列表中的消息按发送时间从新到旧排列。

### 获取指定消息 messageId 前的消息

异步获取会话中，从指定消息之前、指定数量的最新消息实体，返回消息实体 `Message` 对象列表。

#### 接口

```java
public abstract void getHistoryMessages(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final int oldestMessageId,
        final int count,
        final String channelId,
        final IRongCoreCallback.ResultCallback<List<Message>> callback);
```

#### 参数说明

`count` 参数表示返回列表中应包含多少消息。`oldestMessageId` 参数用于控制分页的边界。每次调用 `getHistoryMessages` 方法时，SDK 会以 `oldestMessageId` 参数指向的消息为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `oldestMessageId` 设置为 -1。

建议获取返回结果中最早一条消息的 ID，并在下一次调用时作为 `oldestMessageId` 传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                             | 说明                                                          |
|:-----------------|:---------------------------------|:------------------------------------------------------------|
| conversationType | [ConversationType]               | 会话类型。                                                     |
| targetId         | String                           | 会话 ID。                                                      |
| oldestMessageId  | long                             | 最后一条消息的 ID。如需查询本地数据库中最新的消息，设置为 `-1`。 |
| count            | int                              | 每页消息的数量。                                               |
| channelId        | String                           | 超级群频道 ID。                                                |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列。                  |

### 获取指定时间戳前后的消息

:::tip

 超级群业务从 5.4.5 开始支持该功能。
:::


获取会话中指定时间戳之前之后、指定数量的最新消息实体，返回消息实体 `Message` 对象列表。

#### 接口

```java
public abstract void getHistoryMessages(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String channelId,
        final long sentTime,
        final int before,
        final int after,
        final IRongCoreCallback.ResultCallback<List<Message>> resultCallback);
```

#### 参数说明

| 参数             | 类型                             | 说明                                           |
|:-----------------|:---------------------------------|:---------------------------------------------|
| conversationType | [ConversationType]               | 会话类型。                                      |
| targetId         | String                           | 会话 ID。                                       |
| channelId        | String                           | 超级群频道 ID。                                 |
| timestamp        | long                             | 以此时间戳为界，获取早于或晚于该时间的历史消息。 |
| before           | int                              | 需要获取的发送时间早于指定时间戳的消息数量。    |
| after            | int                              | 需要获取的发送时间晚于指定时间戳的消息数量。    |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列。   |

### 获取会话中指定消息类型的消息

以下方法从指定消息之前的特定类型的最新消息实体，异步返回消息实体 `Message` 对象列表。

#### 接口

```java
public abstract void getHistoryMessages(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String channelId,
        final String objectName,
        final int oldestMessageId,
        final int count,
        final IRongCoreCallback.ResultCallback<List<Message>> callback);
```

#### 参数说明

`count` 参数表示返回列表中应包含多少消息。`oldestMessageId` 参数用于控制分页的边界。每次调用 `getHistoryMessages` 方法时，SDK 会以 `oldestMessageId` 参数指向的消息为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `oldestMessageId` 设置为 -1。

建议获取返回结果中最早一条消息的 ID，并在下一次调用时作为 `oldestMessageId` 传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                             | 说明                                                          |
|:-----------------|:---------------------------------|:------------------------------------------------------------|
| conversationType | [ConversationType]               | 会话类型。                                                     |
| targetId         | String                           | 会话 ID。                                                      |
| objectName       | String                           | 消息类型标识。内置消息类型的标识可参见[消息类型概述]。          |
| oldestMessageId  | long                             | 最后一条消息的 ID。如需查询本地数据库中最新的消息，设置为 `-1`。 |
| count            | int                              | 每页消息的数量。                                               |
| channelId        | String                           | 超级群频道 ID。                                                |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列。                  |

如果需要获取早于或晚于指定消息的历史消息，可以使用以下带 `direction` 参数的方法：

#### 接口

```java
public abstract void getHistoryMessages(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String channelId,
        final String objectName,
        final int baseMessageId,
        final int count,
        final RongCommonDefine.GetMessageDirection direction,
        final IRongCoreCallback.ResultCallback<List<Message>> callback);
```

#### 参数说明

| 参数             | 类型                                 | 说明                                                                       |
|:-----------------|:-------------------------------------|:-------------------------------------------------------------------------|
| conversationType | [ConversationType]                   | 会话类型。                                                                  |
| targetId         | String                               | 会话 ID。                                                                   |
| channelId        | String                               | 超级群频道 ID。                                                             |
| objectName       | String                               | 消息类型标识。内置消息类型的标识可参见[消息类型概述]。                       |
| baseMessageId    | int                                  | 以此消息 ID 为界，获取早于或晚于该消息的历史消息。                           |
| count            | int                                  | 每页消息的数量。                                                            |
| channelId        | String                               | 超级群频道 ID。                                                             |
| direction        | RongCommonDefine.GetMessageDirection | `FRONT`表示获取早于传入时间戳的消息。`BEHIND` 表示获取晚于传入时间戳的消息。 |
| callback         | ResultCallback\<List\<Message\>>     | 获取历史消息的回调，按照时间顺序从新到旧排列。                               |

### 获取指定多个消息类型的历史消息

获取会话中，从指定消息之前或之后、指定数量的、多个消息类型的最新消息实体，异步返回消息实体 `Message` 对象列表。

#### 接口

```java
public abstract void getHistoryMessages(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String channelId,
        final List<String> objectNames,
        final long timestamp,
        final int count,
        final RongCommonDefine.GetMessageDirection direction,
        final IRongCoreCallback.ResultCallback<List<Message>> callback);
```

#### 参数说明

| 参数             | 类型                                 | 说明                                                                       |
|:-----------------|:-------------------------------------|:-------------------------------------------------------------------------|
| conversationType | [ConversationType]                   | 会话类型。                                                                  |
| targetId         | String                               | 会话 ID。                                                                   |
| channelId        | String                               | 超级群频道 ID。                                                             |
| objectNames      | List\<String\>                       | 消息类型标识。内置消息类型的标识可参见[消息类型概述]。                       |
| timestamp        | long                                 | 以此时间戳为界，获取早于或晚于该时间的历史消息。                             |
| count            | int                                  | 每页消息的数量。                                                            |
| direction        | RongCommonDefine.GetMessageDirection | `FRONT`表示获取早于传入时间戳的消息。`BEHIND` 表示获取晚于传入时间戳的消息。 |
| callback         | ResultCallback\<List\<Message\>>     | 获取历史消息的回调，按照时间顺序从新到旧排列。                               |

超级群还支持通过消息 UID 从本地数据库中提取消息。详见[获取历史消息](../message/get-history.md) 中的**通过消息 UID 获取消息**。

## 获取远端历史消息 {#getmsgremote}

从 5.6.4 开始，IMLib SDK 支持使用 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 的 `getRemoteHistoryMessages` 方法直接查询指定会话存储在**超级群消息云端存储**中的历史消息。

:::tip

 用户是否可以获取在加入超级群之前的群聊历史消息取决于 App 在控制台的设置。默认新入群用户只能看到他们入群后的群聊消息。<!-- 配置方法详见开通超级群服务 TODO: 链接已废弃 (https://help.rongcloud.cn/t/topic/1057) -->。
:::


### 获取会话的远端历史消息

您可以通过 `RemoteHistoryMsgOption` 自定义 `getRemoteHistoryMessages` 方法获取远端历史消息的行为。IMLib SDK 会按照指定条件直接查询**单群聊消息云端存储**中的满足查询条件的历史消息。

#### 参数说明

| 参数                   | 类型                             | 说明                        |
|:-----------------------|:---------------------------------|:--------------------------|
| conversationType       | [ConversationType]               | 会话类型                    |
| channelId              | String                           | 频道 ID                     |
| targetId               | String                           | 会话 ID                     |
| remoteHistoryMsgOption | RemoteHistoryMsgOption           | 获取远端历史消息的配置选项。 |
| callback               | ResultCallback\<List\<Message\>> | 获取历史消息的回调。         |

- **`RemoteHistoryMsgOption` 说明**：

    | 参数                     | 说明                                                                                                                                                                                                                                                           |
    |:-------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | dataTime                 | 时间戳，用于控制分页查询消息的边界。默认值为 `0`。                                                                                                                                                                                                                |
    | count                    | 要获取的消息数量。如果 SDK \< 5.4.1，范围为 [2-20]；如果 SDK ≧ 5.4.1，范围为 [2-100]；默认值为 `5`。                                                                                                                                                                 |
    | pullOrder                | 拉取顺序。`DESCEND`：降序，按消息发送时间递减的顺序，获取发送时间早于 `dataTime` 的消息，返回的列表中的消息按发送时间从新到旧排列。`ASCEND`： 升序，按消息发送时间递增的顺序，获取发送时间晚于 `dataTime` 的消息，返回的列表中的消息按发送时间从旧到新排列。默认值为 `1`。 |
    | includeLocalExistMessage | 是否包含本地数据库中的已有消息。`true`：包含，查询结果不与本地数据库排除重复，直接返回。`false`：不包含，查询结果先与本地数据库排除重复，只返回本地数据库中不存在的消息。默认值为 `false`。                                                                              |

`RemoteHistoryMsgOption` 默认按消息发送时间降序查询会话中的消息，且默认会并将查询结果与本地数据库对比，排除重复的消息后，再返回消息对象列表。在 `includeLocalExistMessage` 设置为 `false` 的情况下，建议 App 层先使用 `getHistoryMessages`，在本地数据库消息全部获取完之后，再获取远端历史消息，否则可能会获取不到指定的部分或全部消息。

如需按消息发送时间升序查询会话中的消息，建议获取返回结果中最新一条消息的 `sentTime`，并在下一次调用时作为 `dateTime` 的值传入，以便遍历整个会话的消息历史记录。升序查询消息一般可用于跳转到会话页面指定消息位置后需要查询更新的历史消息的场景。

### 示例代码

```java
Conversation.ConversationType conversationType= Conversation.ConversationType.ULTRA_GROUP;
String targetId="会话 Id";
String channelId="频道 Id";
RemoteHistoryMsgOption remoteHistoryMsgOption=new RemoteHistoryMsgOption();
remoteHistoryMsgOption.setDataTime(1662542712112L);//2022-09-07 17:25:12:112
remoteHistoryMsgOption.setOrder(HistoryMessageOption.PullOrder.DESCEND);
remoteHistoryMsgOption.setCount(20);

ChannelClient.getInstance().getRemoteHistoryMessages(conversationType, targetId, remoteHistoryMsgOption,
                new IRongCoreCallback.ResultCallback<List<Message>>() {
                    /**
                     * 成功时回调
                     * @param messages 获取的消息列表
                     */
                    @Override
                    public void onSuccess(List<Message> messages) {

                    }

                    /**
                     * 错误时回调。
                     * @param e 错误码
                     */
                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode e) {

                    }
                });

```

### 强制从服务端获取特定批量消息

您可以通过`getBatchRemoteUltraGroupMessages` 接口强制从服务端获取对应的消息，获取成功后会强制更新本地消息。

#### 接口

```java
matchedMsgList 从服务获取的消息列表
notMatchMsgList 非法参数或者从服务没有拿到对应消息

interface IGetBatchRemoteUltraGroupMessageCallback {
    void onSuccess(List<Message> matchedMsgList, List<Message> notMatchedMsgList);

    void onError(IRongCoreEnum.CoreErrorCode errorCode);
}

public void getBatchRemoteUltraGroupMessages(final List<Message> msgList,final IRongCoreCallback.IGetBatchRemoteUltraGroupMessageCallback callback)
```


#### 参数说明
| 参数             | 类型                 | 说明                                                                  |
|:-----------------|:---------------------|:--------------------------------------------------------------------|
| messages         | `List<Message>`             | 消息列表。最多查询 20 条，所有消息必须是超级群类型且为同一个会话，每个消息对象内的 ConversationType,targetId,channelId, messageUid,sentTime 需要为有效值。                                                             |
| callback          | IGetBatchRemoteUltraGroupMessageCallback                 | 查询消息数量失败的回调。                                                   |


## 统计本地历史消息数量

:::tip

 [ChannelClient] 从 5.6.4 开始支持该能力。
:::

您可以获取指定超级群在指定时间范围内的消息数量。该方法支持传入一个频道列表，如果不指定任何频道，则返回全部频道在本地的历史消息数量。

#### 接口

```java
public abstract void getUltraGroupMessageCountByTimeRange(
        String targetId,
        String[] channelIds,
        long startTime,
        long endTime,
        IRongCoreCallback.ResultCallback<Integer> callback);
```

#### 参数说明

| 参数             | 类型                 | 说明                                                                  |
|:-----------------|:---------------------|:--------------------------------------------------------------------|
| targetId         | String             | 超级群会话 targetId                                                               |
| channelIds        | `String[]`           | 超级群频道 channelId 列表                                                          |
| sentTime         | long            | 查询 startTime 之后的消息， startTime >= 0。                            |
| endTime      | long              | 查询 endTime 之前的消息，endTime > startTime。              |
| callback          | ResultCallback \<Integer\>                 | 接口回调                                                   |


<!-- links -->
[消息类型概述]: /platform-chat-api/message-about/about-message-types
<!-- api links  -->
[getMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/get-messages.html
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

 [ChannelClient]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html
 