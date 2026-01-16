---
title: 搜索消息
sidebar_position: 80
---

# 搜索消息

您可以通过 IMLib SDK 的相关搜索接口通过关键词、消息类型等条件搜索会话中的符合条件的消息列表，同时支持按时间段搜索。您也可以通过用户 userId 搜索指定会话中符合条件的消息列表。

支持关键字搜索的消息需要实现 `MessageContent` 的 `getSearchableWord` 方法：

- 内置的消息类型中文本消息（[TextMessage])，文件消息（[FileMessage]），和图文消息`RichContentMessage` 类型默认实现了 `MessageContent#getSearchableWord` 方法。
- 自定义消息类型也可以支持关键字搜索，需要您参考文档自行实现。详见[自定义消息类型]。

1. 根据关键字搜索本地存储的全部会话，获取包含关键字的会话列表。
1. 根据搜索会话返回的会话列表数据，调用搜索单个会话的方法，搜索符合条件的消息。

## 搜索符合条件的会话

您可以通过 `searchConversations` 按关键字、消息类型搜索本地存储的符合查找条件的所有会话列表 [SearchConversationResult]。自 5.20.0 版本开始，支持返回超级群会话信息，如果需要超级群功能，请通过[提交工单]开启。

#### 接口

```java
RongIMClient.getInstance().searchConversations(keyword, conversationTypes, messageTypeObjectNames,callback);
```

#### 参数说明

|       参数       |    类型     |        说明        |
|:--------------- |:--------- |:------------------------ |
| keyword |String|搜索的关键字|
| conversationTypes | Conversation.ConversationType[] | 会话类型列表，包含 [ConversationType]。 |
| objectNames | String[] | 消息类型列表，默认仅支持内置类型 `RC:TxtMsg`（文本消息）、`RC:FileMsg`（文件消息）、`RC:ImgTextMsg`（图文消息 ）。|
|callback|RongIMClient.ResultCallback\<List\<SearchConversationResult\>>| 回调。搜索结果为 [SearchConversationResult] 列表。|


#### 示例代码

```java
String keyword = "搜索的关键字";

Conversation.ConversationTypes[] conversationTypes = {ConversationTypes.PRIVATE, ConversationTypes.GROUP};
String[] messageTypeObjectNames = {"RC:TxtMsg"};

RongIMClient.getInstance().searchConversations(keyword, conversationTypes, messageTypeObjectNames,
    new RongIMClient.ResultCallback<List<SearchConversationResult>>() {

    @Override
    public void onSuccess(List<SearchConversationResult> searchConversationResults) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

## 在指定单个会话中搜索

您可以通过不同的方法策略来搜索指定单个会话中符合条件的消息。

### 根据关键字搜索消息 {#bykeyword}

您可以在本地数据库中根据关键字搜索指定会话中的消息，支持搜索指定时间点之前的历史消息记录，回调中返回符合搜索条件的消息列表。

#### 接口

```java
RongIMClient.getInstance().searchMessages(conversationType, targetId, keyword, count, beginTime,callback);
```

#### 参数说明

|       参数       |    类型     |        说明        |
|:--------------- |:--------- |:------------------------ |
|conversationType| [ConversationType]| 会话类型。|
| targetId | String | 会话 ID。 |
| keyword |String| 搜索的关键字。|
| count | int | 每页的数量，每页数量建议最多 100 条。传 `0` 时返回所有搜索到的消息。|
| beginTime | long |查询记录的起始时间。传 `0` 时从最新消息开始搜索。非 `0` 时从该时间往前搜索。|
| callback| RongIMClient.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。 |


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
String keyword = "搜索的关键字";
int count = 20;
long beginTime = 122323344;

RongIMClient.getInstance().searchMessages(conversationType, targetId, keyword, count, beginTime,
    new RongIMClient.ResultCallback<List<Message>>() {

    @Override
    public void onSuccess(List<Message> messages) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

### 根据关键字搜索指定时间段的消息 {#bykeywordandspan}

您可以在指定会话中搜索指定的时间范围内，符合关键字的本地消息，回调中返回符合搜索条件的消息列表。

#### 接口

```java
RongCoreClient.getInstance().searchMessages(conversationType, targetId, keyword, startTime, endTime, offset, limit,callback);
```

#### 参数说明

`limit` 参数控制返回的搜索结果数量，取值范围为 [1-100]。超过 100 默认使用最大值 100。

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
|  conversationType| [ConversationType] | 会话类型|
| targetId | String | 会话 Id |
| keyword | String |搜索的关键字|
| startTime | long |开始时间|
| endTime | long |结束时间|
| offset | int |偏移量|
| limit | int |返回的搜索结果数量，limit 需大于 0。最大值为 100。超过 100 时默认返回 100 条。 |
|callback|IRongCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。|


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
String keyword = "123";
long startTime = 0;
long endTime = 1585815113;
int offset = 0;
int limit = 80;

RongCoreClient.getInstance().searchMessages(conversationType, targetId, keyword, startTime, endTime, offset, limit,
    new IRongCoreCallback.ResultCallback<List<Message>>(){

    @Override
    public void onSuccess(List<Messag> messages) {

    }

    @Override
     public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```

### 根据用户 ID 搜索消息 {#byuserid}

您可以在指定会话中搜索指定用户 userId 发送的消息，支持搜索指定时间点之前的本地的历史消息记录，回调中返回符合搜索条件的消息列表。

#### 接口

```java
RongIMClient.getInstance().searchMessagesByUser(conversationType, targetId, userId, count, beginTime,callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------  |:------------------ |
|conversationType| [ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html) | 会话类型|
| targetId | String  | 会话 ID。 |
|userId|String|要查询的用户 ID。|
| count | int |返回的搜索结果数量。最大值为 100。超过 100 时默认返回 100 条。|
| beginTime | long |查询记录的起始时间。传 `0` 时从最新消息开始搜索。非 `0` 时从该时间往前搜索。|
|callback|IRongCallback.ResultCallback\<List\<Message\>>| 回调。搜索结果为 [Message] 列表。|

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
String userId = "查询的用户 Id";
int count = 20;
long beginTime = 1585815113;

RongIMClient.getInstance().searchMessagesByUser(conversationType, targetId, userId, count, beginTime,
    new RongIMClient.ResultCallback<Message>() {

    @Override
    public void onSuccess(List<Messag> messages) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

### 自定义搜索 {#params}

:::tip
自 5.22.0 版本 `RongCoreClient` 开始支持。
:::

您可通过 `SearchMessageParams` 对象自定义搜索条件，并将该对象传入接口 `searchMessagesWithParams()` 进行消息搜索。

#### 参数说明

| 参数      | 类型                                 | 说明         |
| :-------- | :----------------------------------- | :----------- |
| `params`  | `SearchMessageParams`                | 参数对象     |
| `callback`| `IRongCoreCallback.ExResultCallback` | 结果回调     |

- `callback` 参数说明：
    - `messages`：返回的搜索消息列表。
    - `matchCount`：搜索条件匹配到的总数。
    - `code`：错误码。

#### 参数对象说明

参数对象支持灵活组合搜索条件，包括关键字、数量、排序、偏移、时间范围、会话过滤、消息过滤等。

| 参数                | 类型                  | 说明                 |
| :------------------ | :------------------- | :------------------- |
| `keyword`           | `String`             | 关键字               |
| `limit`             | `Int`                | 搜索数量             |
| `offset`            | `Int`                | 搜索偏移量           |
| `order`             | `Order`              | 排序方式             |
| `timeRange`         | `TimeRange`          | 搜索时间范围（可选） |
| `conversationFilter`| `ConversationFilter` | 会话过滤（可选）     |
| `messageFilter`     | `MessageFilter`      | 消息过滤（可选）     |

- `timeRange` 时间范围详细说明：

```java
// 查询时间轴：
// 0--------startTime--------------------endTime---------Now()
// 正序（order == RCOrderAscending）查询：
// 0--------startTime|---->--------------endTime---------Now()
// 倒序（order == RCOrderDescending）查询：
// 0--------startTime--------------<----|endTime---------Now()
// 倒序有位移 (offset) 查询：
// 0--------startTime----------<----|----endTime---------Now()
```

#### 示例代码

```java
SearchMessageParams params = new SearchMessageParams();
params.setKeyword("123");
params.setLimit(10);
params.setOffset(0);
params.setOrder(SearchMessageParams.Order.DESC);

ConversationFilter conversationFilter = new ConversationFilter();
List<Conversation.ConversationType> conversationTypes = new ArrayList<>();
conversationTypes.add(Conversation.ConversationType.PRIVATE);
conversationTypes.add(Conversation.ConversationType.GROUP);

List<String> targetIds = new ArrayList<>();
targetIds.add("targetId2");
targetIds.add("targetId2");

conversationFilter.setConversationTypes(conversationTypes);
conversationFilter.setTargetIds(targetIds);
params.setConversationFilter(conversationFilter);

TimeRange timeRange = new TimeRange();
timeRange.setStartTime(0);
params.setTimeRange(timeRange);

RongCoreClient.getInstance().searchMessagesWithParams(params, new IRongCoreCallback.ExResultCallback<List<Message>, Integer>() {
    @Override
    public void onSuccess(List<Message> messages, Integer matchCount) {
        // 业务处理...
    }
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode error) {
        // 错误处理...
    }
});
```

<!-- links -->
[自定义消息类型]: ./customize.md
<!-- api links -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[FileMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-file-message/
[SearchConversationResult]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-search-conversation-result/index.html

