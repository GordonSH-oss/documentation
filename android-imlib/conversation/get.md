---
title: 获取会话
sidebar_position: 10
---

# 获取会话

IMLib SDK 会根据用户收发的消息，在本地数据库中生成对应会话，并维护会话列表。应用程序可以获取本地数据库中的会话列表。自 5.20.0 版本开始，SDK 支持在获取会话列表中返回超级群会话信息，以便于开发者根据业务需求对单聊、群聊、超级群会话列表进行混合排序展示；该功能需[提交工单]开启。

## 获取指定单个会话

获取某个会话的详细信息。

#### 接口

```java
RongCoreClient.getInstance().getConversation(conversationType, targetId, callback)
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------|:------------------ |
|conversationType|[ConversationType] |会话类型|
|targetId|String |会话 ID|
|callback|`ResultCallback<Conversation>`|回调接口|

#### 示例代码

```java
String conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";

RongCoreClient.getInstance().getConversation(conversationType, targetId, new IRongCoreCallback.ResultCallback<Conversation>() {

        @Override
        public void onSuccess(Conversation conversation) {
            // 成功并返回会话信息
        }

        @Override
        public void onError(RongIMClient.ErrorCode errorCode) {

        }
});
```


## 批量获取会话信息

除了单个获取会话信息外，IMLib SDK 还支持批量获取会话的详细信息。

#### 接口

```java
RongCoreClient.getInstance().getConversations(conversationIdentifiers, callback)
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------|:------------------ |
|conversationIdentifiers|`List<ConversationIdentifier>`|会话类型|
|callback|`ResultCallback<List<Conversation>>`|回调接口|

> **注意**：
> IMLib SDK 从 5.8.2 版本开始支持批量获取会话信息。
> 支持的会话类型：单聊、群聊、系统。

使用 `getConversations()` 方法查询多个会话的详细信息。
#### 示例代码

```java
// 假设我们有会话标识 conIden1 和 conIden2，它们代表想要查询的会话。

List<ConversationIdentifier> conversationIdentifiers = new ArrayList<>();
conversationIdentifiers.add(ConversationIdentifier.obtain(ConversationType.PRIVATE, "tId1", ""));
conversationIdentifiers.add(ConversationIdentifier.obtain(ConversationType.PRIVATE, "tId2", ""));

RongCoreClient.getInstance().getConversations(conversationIdentifiers, new IRongCoreCallback.ResultCallback<List<Conversation>>() {
                    @Override
                    public void onSuccess(List<Conversation> conversations) {
                        // 成功并返回会话信息
                    }

                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode e) {

                    }
                });

```

## 同步服务端会话信息到本地

:::tip
SDK 从 5.20.0 版本开始提供该接口。您可[提交工单]开通此功能。
:::

针对更换设备登录或卸载重新安装应用的情况下，使用 `getRemoteConversations()` 接口可以将服务端的会话信息同步到本地。该接口只触发拉取操作，会话同步完成后，通过 `setConversationListener()` 设置监听同步结果。当收到 `onConversationPulled()` 回调后，标识远端会话列表已经同步完成。

```java
// 发起拉取远端会话请求。
RongCoreClient.getInstance().getRemoteConversations(new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
    }
});
```

```java
// 注册接口会话同步监听。
RongCoreClient.getInstance().setConversationListener(new IRongCoreListener.ConversationListener() {
    @Override
    public void onConversationSync() {
    }

    @Override
    public void onConversationPulled(int code) {
    }
});
```

## 获取会话列表

会话列表是 SDK 在本地生成和维护的。如果未发生卸载重载或换设备登录，您可以获取本地设备上存储的所有历史消息生成的会话列表。

### 分页获取会话列表

分页获取 SDK 在本地数据库生成的会话列表。返回的会话列表按照时间倒序排列。如果返回结果含有被设置为置顶状态的会话，则置顶会话默认排在最前。

使用 [getConversationListByPage] 方法，时间戳（`startTime`）首次可传 `0`，后续可以使用返回的 [Conversation] 对象的 `sentTime` 或 `operationTime` 属性值为下一次查询的 `startTime`。**注：如果您使用的是 5.6.8 之后的版本，请使用 `operationTime` 字段，不然可能会有数据拉取遗漏的问题。**

:::tip
本接口从 5.20.0 版本支持获取**超级群默认频道**会话，如需要在列表中获取超级群，请升级到 IM 尊享版后，提交工单申请开通。
:::

#### 接口

```java
RongCoreClient.getInstance().getConversationListByPage(callback,timeStamp, count, topPriority, conversationTypes);
```

#### 参数说明

|       参数       |    类型   |        说明         |
|:--------------- |:---------  |:------------------ |
|callback|`ResultCallback<List<Conversation>>`|方法回调。 |
|timeStamp|long | 时间戳，以获取早于这个时间戳的会话列表。首次可传 0，表示从最新开始获取。后续使用真实时间戳。|
|count |int|取回的会话数量。建议此数值不要超过 10 个，当一次性获取的会话数过大时，会导致跨进程通信崩溃，引发获取会话列表失败及通信连接被中断。<br /><br />当实际取回的会话数量小于 `count` 值时，表明已取完数据。|
|topPriority| boolean | 是否优先显示置顶消息。要求 SDK 版本 ≧ 5.6.9。|
|conversationTypes| Array of [ConversationType]|选择要获取的会话类型。可设置多个会话类型。|

#### 示例代码

```java
long timeStamp = 0;
int count = 10;
Conversation.ConversationType[] conversationTypes = {ConversationType.PRIVATE, ConversationType.GROUP};

RongCoreClient.getInstance().getConversationListByPage(new IRongCoreCallback.
        ResultCallback<List<Conversation>>() {

    @Override
    public void onSuccess(List<Conversation> conversations) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
},timeStamp, count, conversationTypes);
```

如果希望返回的会话列表严格按照时间倒序排列，请使用带 `topPriority` 参数的重载方法，并将该参数设置为 `false`。该重载方法仅在 5.6.9 及之后版本提供。

```java
long timeStamp = 0;
int count = 10;
boolean topPriority = false;

Conversation.ConversationType[] conversationTypes = {ConversationType.PRIVATE, ConversationType.GROUP};

RongCoreClient.getInstance().getConversationListByPage(new IRongCoreCallback.
        ResultCallback<List<Conversation>>() {

    @Override
    public void onSuccess(List<Conversation> conversations) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

    }
},timeStamp, count, topPriority, conversationTypes);
```


### 获取未读的会话列表

:::tip
本接口从 5.20.0 版本支持获取**超级群默认频道**会话，如需要在列表中获取超级群，请升级到 IM 尊享版后，提交工单申请开通。
:::

#### 接口

```java
RongCoreClient.getInstance().getUnreadConversationList(callback, conversationTypes);
```

使用 `RongCoreClient` 的 [getUnreadConversationList] 方法获取指定类型的含有未读消息的会话列表，支持单聊、群聊、系统会话。获取到的会话列表按照时间倒序排列，置顶会话会排在最前。

#### 参数说明

|      参数       |   类型     |        说明         |
|:--------------- |:---------|:------------------ |
| callback        |`ResultCallback<List<Conversation>>`|回调接口|
|conversationTypes|Array of [ConversationType] |会话类型数组|


#### 示例代码

```java
ConversationType[] conversationTypes = {ConversationType.PRIVATE, ConversationType.GROUP, ConversationType.SYSTEM};

RongCoreClient.getInstance().getUnreadConversationList(new IRongCoreCallback.ResultCallback<List<Conversation>>() {

        @Override
        public void onSuccess(List<Conversation> conversations) {
            // 成功并返回会话信息
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode e) {

        }
}, conversationTypes);
```

### 按会话类型获取会话列表

:::tip
SDK 从 5.24.0 版本开始提供该接口。
:::

使用 `getConversationListByFilters` 接口按会话类型获取会话列表。

#### 接口

```java
List<ConversationTypeFilter> filters = new ArrayList<>();
ConversationTypeFilter filter = new ConversationTypeFilter();
filter.setType(Conversation.ConversationType.PRIVATE);
filter.setChannelId("ChannelId"); // 频道 ID 列表，过滤频道的 ID。有效值包含 "" 或字符串，不允许传 null
filters.add(filter);

ConversationListOption option = new ConversationListOption();
option.setCount(20); // 获取的数量，默认为 20。当实际取回的会话数量小于 count 值时，表明已取完数据。
option.setStartTime(0); // 会话的时间戳，单位：毫秒。获取这个时间戳之前的会话列表，0 表示从最新开始获取。
option.setTopPriority(true); // 查询结果的排序方式，是否置顶优先，传 true 表示置顶会话优先返回，否则结果只以会话时间排序。

 ChannelClient.getInstance().getConversationListByFilters(filters, option, new IRongCoreCallback.ResultCallback<List<Conversation>>() {
    @Override
    public void onSuccess(List<Conversation> conversations) {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
    }
});
```

#### 参数说明

| 参数       | 类型                          | 说明   |
|:---------|:----------------------------|:-----|
| filters  | List                        | 过滤条件 |
| option   | ConversationListOption | 查询条件 |
| calbback | ResultCallback | 回调接口 |


#### 示例代码

```java
ConversationType[] conversationTypes = {ConversationType.PRIVATE, ConversationType.GROUP, ConversationType.SYSTEM};

RongCoreClient.getInstance().getUnreadConversationList(new IRongCoreCallback.ResultCallback<List<Conversation>>() {

        @Override
        public void onSuccess(List<Conversation> conversations) {
            // 成功并返回会话信息
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode e) {

        }
}, conversationTypes);
```


### 卸载重装或换设备登录后的处理方案

如果您的用户卸载重装或换设备登录，可能会发现会话列表为空，或者有部分会话丢失的错觉。

原因如下：

- 在卸载的时候会删除本地数据库，本地没有任何历史消息，导致重新安装后会话列表为空。
- 如果换设备登录，可能本地没有历史消息数据，导致会话列表为空。
- 如果您的 App Key 开启了[多设备消息同步](https://console.rongcloud.cn/agile/im/service/config#%E5%A4%9A%E7%AB%AF) 功能，服务端会同时启用**离线消息补偿**功能。服务端会在 SDK 连接成功后自动同步当天 0 点后的消息，IMLib SDK 接收到服务端补偿的消息后，可生成部分会话和会话列表。与卸载前或换设备前比较，可能会有部分会话丢失的错觉。

如果您希望在卸载重装或换设备登录后，获取到之前的会话列表，可以参考如下方案：

- 申请增加离线消息补偿的天数，最大可修改为 7 天。**注意，设置时间过长，当单用户消息量超大时，可能会因为补偿消息量过大，造成端上处理压力的问题**。<!--public-cloud-only start-->如有需要，请[提交工单]。<!--public-cloud-only end-->
- 在您的服务器中自行维护会话列表，并通过 API 向服务端获取需要展示的历史消息。

[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[getConversationListByPage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-conversation-list-by-page.html
[RongCoreClient]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html
[getUnreadConversationList]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-channel-client/get-unread-conversation-list.html
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Conversation]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/index.html
[获取超级群频道列表]: ../ultragroup/get-channel-list.md
