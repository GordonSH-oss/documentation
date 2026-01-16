---
title: 搜索本地消息
sidebar_position: 110
---

# 搜索本地消息

:::tip
- 搜索消息功能仅支持搜索本地数据库中存储的消息。
- SDK 从 5.3.4 版本开始支持超级群本地消息搜索功能。
- 融云提供搜索超级群历史消息记录的 IM Server API。详见服务端文档[搜索超级群消息]。
:::

您可以通过 IMLib SDK 的相关搜索接口通过关键词、消息类型等条件搜索会话中的符合条件的消息列表，同时支持按时间段搜索。您也可以通过用户 userId 搜索指定会话中符合条件的消息列表。

支持关键字搜索的消息需要实现 `MessageContent` 的 `getSearchableWord` 方法：
- 内置的消息类型中文本消息（[TextMessage]），文件消息（[FileMessage]），和图文消息`RichContentMessage` 类型默认实现了 `MessageContent#getSearchableWord` 方法。
- 自定义消息类型也可以支持关键字搜索，需要您参考文档自行实现。详见[自定义消息类型]。


## 搜索会话

按关键字搜索本地所有会话。返回符合条件的会话列表 [SearchConversationResult]。请注意，超级群业务中单个会话（`conversation`）仅对应单个超级群频道。

#### 接口

```java
ChannelClient.getInstance().searchConversationForAllChannel(keyword, conversationTypes, messageTypeObjectNames,callback);
```

#### 参数说明
如果搜索条件与搜索结果包含多个会话类型，您可以通过返回结果中 [Conversation] 对象的会话类型（`ConversationType`）字段进行分类或筛选。

|       参数       |    类型     |        说明        |
|:--------------- |:--------- |:------------------------ |
| keyword |String|搜索的关键字|
| conversationTypes | Conversation.ConversationType[] | 会话类型列表，包含 [ConversationType]，例如 `ConversationType.ULTRA_GROUP`、`ConversationTypes.PRIVATE`，`ConversationTypes.GROUP`。 |
| objName | String[] | 消息类型列表，默认仅支持内置类型 `RC:TxtMsg`（文本消息）、`RC:FileMsg`（文件消息）、`RC:ImgTextMsg`（图文消息 ）。|
|callback| IRongCoreCallback.ResultCallback\<List\<SearchConversationResult\>>| 回调。搜索结果为 [SearchConversationResult] 列表。请注意，针对超级群，单个会话（`conversation`）对应单个超级群频道。 |


#### 示例代码

```java
String keyword = "搜索的关键字";
Conversation.ConversationTypes[] conversationTypes = {ConversationType.ULTRA_GROUP};
String[] messageTypeObjectNames = {"RC:TxtMsg"};

ChannelClient.getInstance().searchConversationForAllChannel(keyword, conversationTypes, messageTypeObjectNames,
    new IRongCoreCallback.ResultCallback<List<SearchConversationResult>>() {

    @Override
    public void onSuccess(List<SearchConversationResult> searchConversationResults) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

    }
});
```


## 搜索单个频道的消息

当您获取包含关键词的会话列表后，可以搜索指定单个会话中符合条件的消息。在实现超级群消息搜索时，您可以先通过搜索会话获取符合条件的超级群会话列表，再搜索指定频道中符合条件的消息。

搜索单个频道的消息需要 App 同时提供超级群 ID（`targetId`） 与 频道 ID（`channelId`）。

### 根据关键字搜索指定频道的消息 {#bykeyword}

您可以在本地存储中根据关键字搜索指定频道会话中的消息，支持搜索指定时间点之前的历史消息记录。回调中分页返回包含指定关键字的消息列表。

#### 接口

```java
ChannelClient.getInstance().searchMessages(conversationType, targetId, channelId, keyword, count, beginTime,callback);
```

#### 参数说明

|       参数       |    类型     |        说明        |
|:--------------- |:--------- |:------------------------ |
| conversationType| [ConversationType]| 会话类型。超级群会话传入 `ConversationType.ULTRA_GROUP`。 |
| targetId | String | 超级群 ID。 |
| channelId | String | 超级群频道 ID。 |
| keyword |String| 搜索的关键字。|
| count | int | 每页的数量，每页数量建议最多 100 条。传 `0` 时返回所有搜索到的消息。|
| beginTime | long |查询记录的起始时间。传 `0` 时从最新消息开始搜索。非 `0` 时从该时间往前搜索。|
| callback| IRongCoreCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。 |

#### 示例代码

```java
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String targetId = "超级群 ID";
String channelId = "超级群频道 ID";
String keyword = "搜索的关键字";
int count = 20;
long beginTime = 122323344;

ChannelClient.getInstance().searchMessages(conversationType, targetId, channelId, keyword, count, beginTime,
    new IRongCoreCallback.ResultCallback<List<Message>>() {

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

    }
});
```

### 根据关键字搜索指定时间段内指定频道的消息 {#bykeywordandspan}

SDK 您可以在指定超级群指定频道中搜索指定的时间范围内，符合关键字的本地消息，回调中返回符合搜索条件的消息列表。

#### 接口

```java
ChannelClient.getInstance().searchMessages(conversationType, targetId, channelId, keyword, startTime, endTime, offset, limit,callback);
```

#### 参数说明

`limit` 参数控制返回的搜索结果数量，取值范围为 [1-100]。超过 100 默认使用最大值 100。

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
|  conversationType| [ConversationType] | 会话类型。超级群会话传入 `ConversationType.ULTRA_GROUP`。 |
| targetId | String | 超级群 ID |
| channelId | String | 超级群频道 ID。 |
| keyword | String |搜索的关键字|
| startTime | long |开始时间|
| endTime | long |结束时间|
| offset | int |偏移量|
| limit | int |返回的搜索结果数量，limit 需大于 0。最大值为 100。超过 100 时默认返回 100 条。 |
|callback|IRongCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。|

#### 示例代码

```java
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String targetId = "超级群 ID";
String channelId = "超级群频道 ID";
String keyword = "123";
long startTime = 0;
long endTime = 1585815113;
int offset = 0;
int limit = 80;

ChannelClient.getInstance().searchMessages(conversationType, targetId, channelId, keyword, startTime, endTime, offset, limit,
    new IRongCoreCallback.ResultCallback<List<Message>>(){

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
     public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```

### 根据用户 userid 搜索指定频道的消息 {#byuserid}

您可以在指定超级群会话中搜索指定用户 userId 发送的消息，支持搜索指定时间点之前的本地的历史消息记录，回调中返回符合搜索条件的消息列表。

#### 接口

```java
ChannelClient.getInstance().searchMessagesByUser(conversationType, targetId, channelId, userId, count, beginTime,callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------  |:------------------ |
|conversationType| [ConversationType] | 会话类型。超级群会话传入 `ConversationType.ULTRA_GROUP`。 |
| targetId | String  | 超级群 ID。 |
| channelId | String | 超级群频道 ID。 |
| userId | String| 要查询的用户 ID。|
| count | int |返回的搜索结果数量。最大值为 100。超过 100 时默认返回 100 条。|
| beginTime | long |查询记录的起始时间。传 `0` 时从最新消息开始搜索。非 `0` 时从该时间往前搜索。|
|callback|IRongCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。|


#### 示例代码

```java
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String targetId = "超级群 ID";
String channelId = "超级群频道 ID";
String userId = "查询的用户 Id";
int count = 20;
long beginTime = 1585815113;

ChannelClient.getInstance().searchMessagesByUser(conversationType, targetId, channelId, userId, count, beginTime,
    new IRongCoreCallback.ResultCallback<Message>() {

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

    }
});
```

## 搜索超级群下多个频道的消息

App 如需搜索指定超级群的消息，并已持有超级群 ID（`targetId`），可以直接使用以下接口搜索本地已接收、已存储的消息。符合搜索条件的消息可能来自多个频道。

### 根据关键字搜索指定超级群本地所有频道的消息

搜索指定超级群存储在本地的消息，支持搜索指定时间点之前的历史消息记录。回调中分页返回包含指定关键字的消息列表。

#### 接口

```java
ChannelClient.getInstance().searchMessageForAllChannel(targetId, conversationType, keyword, count, timestamp,callback);
```

#### 参数说明


|       参数       |    类型     |        说明        |
|:--------------- |:--------- |:------------------------ |
| targetId | String | 超级群 ID。 |
| conversationType | [ConversationType] | 会话类型。超级群会话传入 `ConversationType.ULTRA_GROUP`。 |
| keyword |String| 搜索的关键字。|
| count | int | 每页的数量，每页数量建议最多 100 条。传 `0` 时返回所有搜索到的消息。|
| beginTime | long |查询记录的起始时间。传 `0` 时从最新消息开始搜索。非 `0` 时从该时间往前搜索。|
| callback| IRongCoreCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。 |


#### 示例代码

```java
String targetId = "超级群 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String keyword = "搜索的关键字";
int count = 20;
long timestamp = "122323344";

ChannelClient.getInstance().searchMessageForAllChannel(targetId, conversationType, keyword, count, timestamp,
    new IRongCoreCallback.ResultCallback<List<Message>>() {

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

    }
});
```

### 根据关键字搜索指定超级群指定时间段内本地所有频道的消息

在搜索指定超级群存储在本地的消息时，SDK 支持将关键字搜索的范围限制在指定时间段内。回调中分页返回包含指定关键字和时间段要求的消息列表。

#### 接口

```java
ChannelClient.getInstance().searchMessageByTimestampForAllChannel(conversationType, targetId, keyword, startTime, endTime, offset, limit,callback);
```


#### 参数说明

`limit` 参数控制返回的搜索结果数量，取值范围为 [1-100]。超过 100 默认使用最大值 100。

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
| targetId | String | 超级群 ID |
| conversationType| [ConversationType] | 会话类型。超级群会话传入 `ConversationType.ULTRA_GROUP`。 |
| keyword | String |搜索的关键字|
| startTime | long |开始时间|
| endTime | long |结束时间|
| offset | int |偏移量|
| limit | int |返回的搜索结果数量，limit 需大于 0。最大值为 100。超过 100 时默认返回 100 条。 |
| callback |IRongCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。|


#### 示例代码

```java
String targetId = "超级群 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;
String keyword = "123";
long startTime = 0;
long endTime = 1585815113;
int offset = 0;
int limit = 80;

ChannelClient.getInstance().searchMessageByTimestampForAllChannel(conversationType, targetId, keyword, startTime, endTime, offset, limit,
    new IRongCoreCallback.ResultCallback<List<Message>>(){

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
     public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```
### 根据关键字搜索指定超级群下多个频道的本地历史消息

:::tip

 SDK 从 5.6.2 版本开始提供该方法。
:::

请确保传入合法的 Channel ID 数组。Channel ID 数组不可为空，ID 数量不可超过 50 个，数组中的 Channel ID 均必须为合法有效的值（大小写英文字母与数字），不支持 Channel ID 为空字符串。支持从最新消息开始搜索，或者搜索早于 `startTime` 的消息。每次搜索最大 100 条，如果还有更多的消息，可以将 `startTime` 设为获取到的消息数组最后一条消息的时间戳，然后继续调用接口。

#### 接口

```java
/**
 * 根据关键字搜索指定超级群下多个频道的本地历史消息
 *
 * <p>注意：如果需要自定义消息也能被搜索到，需要在自定义消息中实现 {@link MessageContent#getSearchableWord()} 方法。
 *
 * @param conversationType 指定的会话类型。
 * @param targetId 指定的会话 id。
 * @param channelIds 消息所属会话的业务标识。(0 < channelIds的数量 <= 50, 不符合则返回 {@link
 *     IRongCoreEnum.CoreErrorCode#RC_INVALID_PARAMETER_CHANNEL_ID})
 * @param keyword 关键词。(0<长度<=1000)
 * @param startTime 查询记录的起始时间， 传 0 时从最新消息开始搜索，从该时间往前搜索。
 * @param limit 返回的搜索结果数量 {@code 0 < limit <= 100}，如果 {@code limit > 100}，则返回 100。
 * @param resultCallback 搜索结果回调。
 * @since 5.6.2
 */
public abstract void searchMessagesForChannels(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String[] channelIds,
        final String keyword,
        final long startTime,
        final int limit,
        final IRongCoreCallback.ResultCallback<List<Message>> resultCallback);

```

### 根据发送者用户 ID 搜索指定超级群下多个频道的本地历史消息

:::tip

 SDK 从 5.6.2 版本开始提供该方法。
:::


请确保传入合法的 Channel ID 数组。Channel ID 数组不可为空，ID 数量不可超过 50 个，数组中的 Channel ID 均必须为合法有效的值（大小写英文字母与数字），不支持 Channel ID 为空字符串。支持从最新消息开始搜索，或者搜索早于 `startTime` 的消息。每次搜索最大 100 条，如果还有更多的消息，可以将 `startTime` 设为获取到的消息数组最后一条消息的时间戳，然后继续调用接口。

#### 接口

```java
/**
 * 根据会话id、会话业务标识、用户ID等搜索指定会话中的消息。
 *
 * <p>注意：如果需要自定义消息也能被搜索到，需要在自定义消息中实现 {@link MessageContent#getSearchableWord()} 方法。
 *
 * @param conversationType 指定的会话类型。
 * @param targetId 指定的会话 id。
 * @param channelIds 消息所属会话的业务标识。(0 < channelIds的数量 <= 50, 不符合则返回 {@link
 *     IRongCoreEnum.CoreErrorCode#RC_INVALID_PARAMETER_CHANNEL_ID})
 * @param userId 用户ID。
 * @param startTime 查询记录的起始时间， 传 0 时从最新消息开始搜索，从该时间往前搜索。
 * @param limit 返回的搜索结果数量 {@code 0 < limit <= 100}，如果 {@code limit > 100}，则返回 100。
 * @param resultCallback 搜索结果回调。
 * @since 5.6.2
 */
public abstract void searchMessagesByUserForChannels(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String[] channelIds,
        final String userId,
        final long startTime,
        final int limit,
        final IRongCoreCallback.ResultCallback<List<Message>> resultCallback);

```

### 根据发送者用户 ID 搜索指定超级群下所有频道的本地历史消息

:::tip

 SDK 从 5.6.2 版本开始提供该方法。
:::


支持从最新消息开始搜索，或者搜索早于 `startTime` 的消息。每次搜索最大 100 条，如果还有更多的消息，可以将 `startTime` 设为获取到的消息数组最后一条消息的时间戳，然后继续调用接口。

#### 接口

```java
/**
 * 根据会话id、用户id等搜索指定会话中的消息。(包含所有的 ChannelId)
 *
 * <p>注意：如果需要自定义消息也能被搜索到，需要在自定义消息中实现 {@link MessageContent#getSearchableWord()} 方法。
 *
 * @param conversationType 指定的会话类型。
 * @param targetId 指定的会话 id。
 * @param userId 用户ID。
 * @param startTime 查询记录的起始时间， 传 0 时从最新消息开始搜索，从该时间往前搜索。
 * @param limit 返回的搜索结果数量 {@code 0 < limit <= 100}，如果 {@code limit > 100}，则返回 100。
 * @param resultCallback 搜索结果回调。
 * @since 5.6.2
 */
public abstract void searchMessagesByUserForAllChannel(
        final Conversation.ConversationType conversationType,
        final String targetId,
        final String userId,
        final long startTime,
        final int limit,
        final IRongCoreCallback.ResultCallback<List<Message>> resultCallback);

```


<!-- links -->
[自定义消息类型]: ../message/customize.md
[搜索超级群消息]: /platform-chat-api/ultragroup/query-history-messages
[获取历史消息]: ./get-history-message.md
<!-- api links -->
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[FileMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-file-message/
[SearchConversationResult]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-search-conversation-result/index.html
[Conversation]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/index.html

