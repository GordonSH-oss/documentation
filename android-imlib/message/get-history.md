---
title: 获取历史消息
sidebar_position: 30
---

# 获取历史消息

获取历史消息可以仅从本地数据中获取，仅从远端获取，和同时从本地与远端获取。
<!--public-cloud-only start-->
## 开通服务

从远端获取单群聊历史消息是指从融云服务端获取历史消息，该功能要求 App Key 已启用融云提供的**单群聊消息云端存储**服务。您可以在控制台 IM 服务的[服务购买](https://console.rongcloud.cn/agile/im/service/purchase)页面为当前使用的 App Key 开启服务。如果使用生产环境的 App Key，请注意仅 **IM 旗舰版**或 **IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。

**提示**：请注意区分历史消息记录与离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>。融云针对单聊、群聊、系统消息默认提供最多 7 天（可调整）的离线消息缓存服务。客户端上线时 SDK 会自动收取离线期间的消息，无需 App 层调用 API。详见[管理离线消息存储配置](./manage-offline-message-duration.md)。
<!--public-cloud-only end-->
## 从本地数据库中获取消息 {#getmsglocal}

使用 `getHistoryMessages` 方法可分页查询指定会话存储在本地数据库中的历史消息，并返回消息对象列表。列表中的消息按发送时间从新到旧排列。

### 获取会话中所有类型的消息

```java
RongIMClient.getInstance().getHistoryMessages(conversationType, targetId, oldestMessageId, count,callback);
```

`count` 参数表示返回列表中应包含多少消息。`oldestMessageId` 参数用于控制分页的边界。每次调用  `getHistoryMessages` 方法时，SDK 会以 `oldestMessageId` 参数指向的消息为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `oldestMessageId` 设置为 -1。

建议获取返回结果中最早一条消息的 ID，并在下一次调用时作为 `oldestMessageId` 传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                             | 说明                                                          |
|:-----------------|:---------------------------------|:------------------------------------------------------------|
| conversationType | [ConversationType]               | 会话类型                                                      |
| targetId         | String                           | 会话 ID                                                       |
| oldestMessageId  | long                             | 最后一条消息的 ID。如需查询本地数据库中最新的消息，设置为 `-1`。 |
| count            | int                              | 每页消息的数量。                                               |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列                   |

```java
Conversation.ConversationType conversationType = Conversation.ConversationType.PRIVATE;
String targetId = "会话 Id";
int oldestMessageId = -1;
int count = 10;

RongIMClient.getInstance().getHistoryMessages(conversationType, targetId, oldestMessageId, count,
                new RongIMClient.ResultCallback<List<Message>>() {
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
                     public void onError(RongIMClient.ErrorCode e) {
                    }
                });
```

### 获取会话中指定类型的消息 {#getMsgLocal_obj}

```java
RongIMClient.getInstance().getHistoryMessages(conversationType, targetId, objectName, oldestMessageId, count, callback);
```

`count` 参数表示返回列表中应包含多少消息。`oldestMessageId` 参数用于控制分页的边界。每次调用  `getHistoryMessages` 方法时，SDK 会以 `oldestMessageId` 参数指向的消息为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `oldestMessageId` 设置为 -1。`objectName` 参数指定需要获取的消息类型。

建议获取返回结果中最早一条消息的 ID，并在下一次调用时作为 `oldestMessageId` 传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                             | 说明                                                          |
|:-----------------|:---------------------------------|:------------------------------------------------------------|
| conversationType | [ConversationType]               | 会话类型。                                                     |
| targetId         | String                           | 会话 ID。                                                      |
| objectName       | String                           | 消息类型标识。内置消息类型的标识可参见[消息类型概述]。          |
| oldestMessageId  | long                             | 最后一条消息的 ID。如需查询本地数据库中最新的消息，设置为 `-1`。 |
| count            | int                              | 每页消息的数量。                                               |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列。                  |

### 通过消息 UID 获取消息

:::tip

 - SDK 从 5.2.5.2 版本开始支持通过 UID 批量获取消息的接口。仅在 [ChannelClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/index.html) 类中提供。支持的会话类型包括单聊、群聊、聊天室、超级群。
 - 如果 SDK 版本低于 5.2.5.2，可以使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 类中的 `getMessageByUId` 方法，该方法一次仅支持传入一个 UID。
:::


消息的 UID 是由融云服务端生成的全局唯一 ID。消息存入本地数据库后，App 可能需要再次提取特定消息。例如，您的用户希望收藏聊天记录中的部分消息，App 可以先记录消息的 UID，在需要展示时调用 `getBatchLocalMessages`，传入收藏消息的 UID，从本地数据库中提取消息。

```java
ChannelClient.getInstance().getBatchLocalMessages(conversationType, targetId, channelId, messageUIDs, callback);
```

只要持有消息 UID（`messageUIDs`），并且本地数据库中已存有消息，即可以使用该方法从本地数据库提取消息。单次仅可从一个会话（`targetId`）或超级群频道（`channelId`）中提取消息。

在会话类型为**超级群**或**聊天室**时，请注意以下情况：

- 超级群会话默认只同步会话最后一条消息。如果您直接调用该方法（例如您传入了通过融云服务端回调**全量消息路由**获取的 UID），可能 SDK 无法在本地找到对应消息。建议先调用 `getBatchRemoteUltraGroupMessages` 从服务端获取消息。
- 聊天室会话自动在用户退出清空本地消息。如果用户退出聊天室后再调用该接口，则无法取得本地消息。

| 参数             | 类型                                         | 说明                                                                          |
|:-----------------|:---------------------------------------------|:----------------------------------------------------------------------------|
| conversationType | [ConversationType]                           | 会话类型，支持单聊、群聊、聊天室、超级群。                                         |
| targetId         | String                                       | 会话 ID                                                                       |
| channelId        | String                                       | 超级群的频道 ID。非超级群会话类型时，传入空字符串 `""`。                               |
| messageUIDs      | List\<String\>                               | 消息的 UID，即由融云服务端生成的全局唯一 ID。必须传入有效的 UID，最多支持 20 条。 |
| callback         | IRongCoreCallback.IGetMessagesByUIDsCallback | 获取历史消息的回调。获取成功时会返回消息列表，和提取失败的消息的 UID 列表。      |


```java
Conversation.ConversationType conversationType = Conversation.ConversationType.PRIVATE;
String targetId = "会话 Id";
String sampleMessageUID = "C3GC-8VAA-LJQ4-TPPM";
List<String> messageUIDs = new ArrayList<>();
messageUIDs.add(sampleMessageUID);

ChannelClient.getInstance().getBatchLocalMessages(conversationType, targetId, null, messageUIDs,
                new IRongCoreCallback.IGetMessagesByUIDsCallback() {
                    /**
                     * 成功时回调
                     * @param messageList 获取的消息列表
                     * @param mismatchList 失败的消息 UID 列表
                     */
                    @Override
                    public void onSuccess(List<Message> messages, List<String> mismatchList) {

                    }

                    /**
                     * 错误时回调。
                     * @param errorCode 错误码
                     */
                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

                    }
                });

```

## 获取远端历史消息 {#getmsgremote}

使用 `getRemoteHistoryMessages` 方法可直接查询指定会话存储在**单群聊消息云端存储**中的历史消息。

:::tip

- 用户是否可以获取在加入群组之前的群聊历史消息取决于 App 在控制台的设置。您可以在控制台的[功能配置]页面，启用**新用户获取加入群组前历史消息**。启用此选项后，新入群用户可以获取在他们加入群组之前发送的所有群聊消息。如不启用，新入群用户只能看到他们入群后的群聊消息。
- 默认情况下，用户不在群组中不能获取群组中的历史消息。如果您希望用户未在指定群组中时，也可以获取群组历史消息，可以在[融云控制台](https://console.rongcloud.cn/agile/im/service/config#%E5%8D%95%E7%BE%A4%E8%81%8A)，在 **IM 服务**>**功能配置**>**单群聊**>**用户不在群组时是否可以拉取历史消息**，允许不在群组的用户也可以获取该群组的的历史消息。
:::


### 获取会话的远端历史消息

SDK 按照指定条件直接查询并获取**单群聊消息云端存储**中的满足查询条件的历史消息。查询结果与本地数据库对比，排除重复的消息后，返回消息对象列表。返回的消息列表中的消息按发送时间从新到旧排列。

因为默认该接口返回的消息会跟本地消息排重后返回，建议先使用 `getHistoryMessages`，在本地数据库消息全部获取完之后，再获取远端历史消息。否则可能会获取不到指定的部分或全部消息。

```java
RongIMClient.getInstance().getRemoteHistoryMessages(conversationType, targetId, dateTime, count,callback);
```

`count` 参数表示返回列表中应包含多少消息。`dateTime` 参数用于控制分页的边界。每次调用  `getRemoteHistoryMessages` 方法时，SDK 会以 `dateTime` 为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `dateTime` 设置为 0。

建议获取返回结果中最早一条消息的 `sentTime`，并在下一次调用时作为 `dateTime` 的值传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                             | 说明                                                                              |
|:-----------------|:---------------------------------|:--------------------------------------------------------------------------------|
| conversationType | [ConversationType]               | 会话类型                                                                          |
| targetId         | String                           | 会话 Id                                                                           |
| dateTime         | long                             | 时间戳，获取发送时间早于 `dateTime` 的历史消息。传 `0` 表示获取最新 `count` 条消息。 |
| count            | int                              | 要获取的消息数量。如果 SDK \< 5.4.1，范围为 [2-20]；如果 SDK ≧ 5.4.1，范围为 [2-100]。 |
| callback         | ResultCallback\<List\<Message\>> | 获取历史消息的回调，按照时间顺序从新到旧排列。                                      |

```java
Conversation.ConversationType conversationType = Conversation.ConversationType.PRIVATE;
String targetId = "会话 Id";
long dateTime = 0;
int count = 20;

RongIMClient.getInstance().getRemoteHistoryMessages(conversationType, targetId, dateTime, count,
                new RongIMClient.ResultCallback<List<Message>>() {
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
                    public void onError(RongIMClient.ErrorCode e) {

                    }
                });

```

### 自定义获取会话的远端历史消息

通过 `RemoteHistoryMsgOption` 可以自定义 `getRemoteHistoryMessages` 方法获取远端历史消息的行为。SDK 会按照指定条件直接查询**单群聊消息云端存储**中的满足查询条件的历史消息。

```java
RongIMClient.getInstance().getRemoteHistoryMessages(conversationType, targetId, remoteHistoryMsgOption, callback);
```

`remoteHistoryMsgOption` 中包含多个配置项，其中 `count` 与 `dateTime` 参数分别时获取历史消息的数量与分页查询时间戳。`pullOrder` 参数用于控制获取历史消息的时间方向，可选择获取早于或者晚于给定的 `dateTime` 的消息。`includeLocalExistMessage` 参数控制返回消息列表中是否需要包含本地数据库已存在的消息。

| 参数                   | 类型                             | 说明                        |
|:-----------------------|:---------------------------------|:--------------------------|
| conversationType       | [ConversationType]               | 会话类型                    |
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

```java
Conversation.ConversationType conversationType= Conversation.ConversationType.PRIVATE;
String targetId="会话 Id";
RemoteHistoryMsgOption remoteHistoryMsgOption=new RemoteHistoryMsgOption();
remoteHistoryMsgOption.setDataTime(1662542712112L);//2022-09-07 17:25:12:112
remoteHistoryMsgOption.setOrder(HistoryMessageOption.PullOrder.DESCEND);
remoteHistoryMsgOption.setCount(20);

RongIMClient.getInstance().getRemoteHistoryMessages(conversationType, targetId, remoteHistoryMsgOption,
                new RongIMClient.ResultCallback<List<Message>>() {
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
                    public void onError(RongIMClient.ErrorCode e) {

                    }
                });

```

## 获取本地与远端历史消息

:::tip

 该接口在 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 类中提供。**注意**：用户是否可以获取在加入群组之前的群聊历史消息取决于 App 在控制台的设置。您可以在控制台的[功能配置]页面，启用**新用户获取加入群组前历史消息**。启用此选项后，新入群用户可以获取在他们加入群组之前发送的所有群聊消息。如不启用，新入群用户只能看到他们入群后的群聊消息。
:::


`getMessages` 方法与 `getRemoteHistoryMessages` 的区别是 `getMessages` 会先查询指定会话存储本地数据库的消息，当本地消息无法满足查询条件时，再查询在**单群聊消息云端存储**中的历史消息，以返回连续且相邻的消息对象列表。

```java
RongCoreClient.getInstance().getMessages(conversationType, targetId, historyMessageOption, callback);
```

通过 `historyMessageOption` 可以控制 `getMessages` 方法获取历史消息的行为。`count` 参数表示返回列表中应包含多少消息。`dateTime` 参数用于控制分页的边界。每次调用  `getRemoteHistoryMessages` 方法时，SDK 会以 `dateTime` 为界，继续在下一页返回指定数量的消息。如果需要获取会话中最新的 `count` 条消息，可以将 `dateTime` 设置为 0。`pullOrder` 参数用于控制获取历史消息的时间方向，可选择获取早于或者晚于给定的 `dateTime` 的消息。

建议获取返回结果中最早一条消息的 `sentTime`，并在下一次调用时作为 `dateTime` 的值传入，以便遍历整个会话的消息历史记录。

| 参数             | 类型                                                    | 说明                    |
|:-----------------|:--------------------------------------------------------|:----------------------|
| conversationType | [ConversationType]                                      | 会话类型                |
| targetId         | String                                                  | 会话 ID                 |
| historyMsgOption | HistoryMessageOption                                    | 获取历史消息的配置选项。 |
| callback         | IRongCoreCallback.IGetMessageCallback\<List\<Message\>> | 获取历史消息的回调。     |

- **`HistoryMessageOption` 说明**：

    | 参数      | 说明                                                                                                                                                                                                                                              |
    |:----------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | dataTime  | 时间戳，用于控制分页查询消息的边界。默认值为 `0`。                                                                                                                                                                                                   |
    | count     | 要获取的消息数量。如果 SDK \< 5.4.1，范围为 [2-20]；如果 SDK ≧ 5.4.1，范围为 [2-100]；默认值为 `5`。                                                                                                                                                    |
    | pullOrder | 拉取顺序。`DESCEND`：降序，按消息发送时间递减的顺序，获取发送时间早于 `dataTime` 的消息，返回的列表中的消息按发送时间从新到旧排列。`ASCEND`： 升序，按消息发送时间递增的顺序，获取发送时间晚于 `dataTime` 的消息，返回的列表中的消息按发送时间从旧到新排列。 |

`HistoryMessageOption` 默认按消息发送时间升序查询会话中的消息。升序查询消息一般可用于跳转到会话页面指定消息位置后需要查询更新的历史消息的场景。

如需按消息发送时间降序查询会话中的消息，建议获取返回结果中最早一条消息的 `sentTime`，并在下一次调用时作为 `dateTime` 的值传入，以便遍历整个会话的消息历史记录。

```java
Conversation.ConversationType conversationType= Conversation.ConversationType.PRIVATE;
String targetId="会话 Id";
HistoryMessageOption historyMessageOption=new HistoryMessageOption();
historyMessageOption.setDataTime(1662542712112L);//2022-09-07 17:25:12:112
historyMessageOption.setOrder(HistoryMessageOption.PullOrder.ASCEND);
historyMessageOption.setCount(20);
RongCoreClient.getInstance().getMessages(conversationType, targetId, historyMessageOption, new IRongCoreCallback.IGetMessageCallback() {
    @Override
    public void onComplete(List<Message> list, IRongCoreEnum.CoreErrorCode coreErrorCode) {

        Log.e("tag","list---"+list.size());
    }
});
```

<!--links-->
[消息类型概述]: /platform-chat-api/message-about/about-message-types
[功能配置]: https://console.rongcloud.cn/agile/im/service/config
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

