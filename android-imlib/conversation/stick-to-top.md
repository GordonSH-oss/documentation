---
title: 会话置顶
sidebar_position: 90
---

# 会话置顶

会话置顶功能提供以下能力：

- 在会话列表中置顶会话：通过会话 `Conversation` 的置顶 `isTop` 属性控制。
- 在携带同一标签的会话中置顶（需配合使用[会话标签]功能）：通过 [ConversationTagInfo] 类的 `isTop` 属性控制。
- 从 5.20.0 版本开始，支持超级群类型会话置顶，超级群功能需[提交工单]开启。

## 在会话列表中置顶会话

设置指定会话在会话列表中置顶后，SDK 将修改 [Conversation] 的 `isTop` 字段，该状态将会被同步到服务端。融云会在为用户自动同步会话置顶的状态数据。客户端可以主动获取或通过监听器获取到最新数据。

### 设置会话置顶

使用 [setConversationToTop] 设置会话置顶。

#### 接口
```java
RongIMClient.getInstance().setConversationToTop(conversationType, targetId, isTop, callback);
```
#### 参数说明

|       参数       |    类型   | 说明                             |
|:--------------- |:--------- |:-------------------------------|
|conversationType|[ConversationType]  | 会话类型，支持单聊、群聊、系统、超级群会话。            |
|targetId|String | 会话 ID                          |
|isTop|boolean| 是否置顶。`true` 为置顶，`false` 为取消置顶。 |
|callback | ResultCallback\<Boolean\> | 回调接口                           |


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
boolean isTop = true;
boolean needCreate = true;

RongIMClient.getInstance().setConversationToTop(conversationType, targetId, isTop, new
    ResultCallback<Boolean>() {

        @Override
        public void onSuccess(Boolean success) {

        }

        @Override
        public void onError(RongIMClient.ErrorCode ErrorCode) {

        }
    }) ;
```

客户端通过本地消息数据自动生成会话与会话列表，并会在用户登录的多个设备之间同步置顶状态。如果在调用该 API 时，会话尚未生成，或者已被移除，SDK 的处理方式如下：

- 如果 SDK 版本 ≧ 5.6.8，当需要置顶的会话在本地或该用户登录的其他设备上不存在时，SDK 会自动创建会话并置顶。
- 如果 SDK 版本 < 5.6.8，必须使用支持 `needCreate` 参数的重载方法，并设置该参数为 `true`，SDK 才会自动创建会话并置顶。详见 API 参考 [setConversationToTop]。

### 设置会话置顶可选择是否更新会话时间（SDK 版本 ≧ 5.8.2）

使用 [setConversationToTop] 设置会话置顶。

#### 接口
```java
RongCoreClient.getInstance().setConversationToTop(conversationType, targetId, isTop, needCreate, needUpdateTime, callback)
```

#### 参数说明

|       参数       |    类型   | 说明                              |
|:--------------- |:--------- |:--------------------------------|
|conversationType|[ConversationType]  | 会话类型，支持单聊、群聊、系统、超级群会话。             |
|targetId|String | 会话 ID                           |
|isTop|boolean| 是否置顶。`true` 为置顶，`false` 为取消置顶。  |
|needCreate|boolean| 是否创建。`true` 为创建，`false` 为不创建。   |
|needUpdateTime|boolean| 是否更新时间。`true` 为更新，`false` 为不更新。 |
|callback | ResultCallback\<Boolean\> | 回调接口                            |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
boolean isTop = true;
boolean needCreate = true;
boolean needUpdateTime = true;

RongCoreClient.getInstance().setConversationToTop(conversationType, targetId, isTop, needCreate, needUpdateTime, new
    IRongCoreCallback.ResultCallback<Boolean>() {

        @Override
        public void onSuccess(Boolean success) {

        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) {

        }
    }) ;
```

|       参数       |    类型   |        说明         |
|:--------------- |:--------- |:------------------ |
|conversationType|[ConversationType]  |会话类型，支持单聊、群聊、系统会话。|
|targetId|String | 会话 ID |
|isTop|boolean| 是否置顶。`true` 为置顶，`false` 为取消置顶。|
|needCreate|boolean| 是否创建。`true` 为创建，`false` 为不创建。|
|needUpdateTime|boolean| 是否更新时间。`true` 为更新，`false` 为不更新。|
|callback | ResultCallback\<Boolean\> |回调接口|

### 批量设置会话置顶

使用 `setConversationsToTop:isTop:option:completion:` 接口，批量设置会话置顶。

```java
List<ConversationIdentifier> identifiers = new ArrayList<>();
ConversationIdentifier identifier = ConversationIdentifier.obtain(
    Conversation.ConversationType.PRIVATE,
    "targetId",
    ""
);
identifiers.add(identifier);
ConversationTopOption option = new ConversationTopOption(true, true);

RongCoreClient.getInstance().setConversationsToTop(identifiers, true, option, new IRongCoreCallback.ResultCallback<Boolean>() {
    @Override
    public void onSuccess(Boolean aBoolean) {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode code) {
    }
});
```

| 参数                      |      类型      |    说明    |
|:------------------------| :-------- | :--------- |
| conversationIdentifiers | [ConversationIdentifier] | 会话标识列表。   |
| isTop                   | boolean                     | 是否置顶   |
| option                  | ConversationTopOption  | 配置信息，支持设置是否更新时间和是否创建会话   |
| callback                | IRongCoreCallback.ResultCallback  | 设置置顶结果回调 |

### 监听置顶状态同步 {#setlistener}

SDK 提供了会话状态（置顶状态数据和免打扰状态数据）同步机制。设置会话状态同步监听器后，如果会话状态改变，可在本端收到通知。

会话的置顶和免打扰状态数据同步后，触发 `ConversationStatusListener` 的 `onStatusChanged` 方法。详细说明可参见[多端同步免打扰/置顶](./share-conversation-status-between-clients.md)。



:::tip

 `ConversationStatusListener` 从 5.24.0 版本开始增加会话状态同步完成回调，会话状态同步完成后，触发 `onConversationStatusSync` 方法回调。
:::

#### 示例代码

```java
public interface ConversationStatusListener {
  // 会话状态（置顶和免打扰）多端同步监听
  void onStatusChanged(ConversationStatus[] conversationStatus);
  // 会话状态同步完成
  void onConversationStatusSync(IRongCoreEnum.CoreErrorCode code);
}
```

### 获取会话置顶状态

:::tip

 SDK 从 5.1.5 版本开始支持该功能。
:::


主动获取指定会话的置顶状态。

#### 接口

```java
RongCoreClient.getInstance().getConversationTopStatus(targetId, conversationType, callback)
```
#### 参数说明
| 参数             | 类型                             | 说明     |
| ---------------- | -------------------------------- | -------- |
| targetId         | String                           | 会话 ID  |
| conversationType | [ConversationType]                 | 会话类型 |
| callback         | IRongCoreCallback.ResultCallback | 回调结果 |

### 获取置顶会话列表

主动获取指定会话类型的所有置顶会话。

#### 参数说明
|       参数       |    类型   |        说明         |
|:--------------- |:--------- |:------------------ |
| callback |ResultCallback\<List\<Conversation>> |回调接口|
| conversationTypes |Conversation.ConversationType... |会话类型数组|

#### 示例代码

```java
Conversation.ConversationType[] conversationTypes = {Conversation.ConversationType.PRIVATE};

RongIMClient.getInstance().getTopConversationList(new ResultCallback<List<Conversation>>() {
            @Override
            public void onSuccess(List<Conversation> conversations) {

            }

            @Override
            public void onError(ErrorCode e) {

            }
        }, conversationTypes);
```



## 在携带标签的所有会话中置顶会话

:::tip

 该功能相关接口仅在 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 中提供。在标签标记的所有会话中置顶是通过修改 `ConversationTagInfo.isTop` 字段实现的，不影响 [Conversation] 的 `isTop` 字段。
:::


如果 App 实现了[会话标签]功能，App 用户可能会使用同一个标签标记多个会话，并且需要将其中一个会话置顶。SDK 在 [ConversationTagInfo] 中提供 `isTop` 属性，用于控制会话是否需要在携带同样标签的会话中置顶。

### 在标签下置顶会话

在携带指定标签的所有会话中设置指定会话置顶。例如，在所有添加了「培训班」标签的会话中将与「Tom」的私聊会话置顶。


#### 接口

```java
RongCoreClient.getInstance().setConversationToTopInTag(tagId, conversationIdentifier, isTop,callback) 
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :-----|:-----|:----|
| tagId | String | 标签 ID |
| conversationIdentifier | [ConversationIdentifier] | 会话标识，需要指定会话类型（[ConversationType]）和 Target ID。 |
|isTop | boolean | 是否置顶|
| callback | OperationCallback | 操作回调 |


#### 示例代码
```java
String targetId = "useridoftom";
ConversationIdentifier conversationIdentifier = new ConversationIdentifier(Conversation.ConversationType.PRIVATE, targetId);
String tagId = "peixunban";

RongCoreClient.getInstance().setConversationToTopInTag(tagId, conversationIdentifier, isTop,
    new IRongCoreCallback.OperationCallback() {
            /**
             * 成功回调
             */
            @Override
            public void onSuccess() {

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



### 获取会话在标签下的置顶状态

查询指定会话是否在携带同一标签的所有会话中置顶。获取成功后会返回是否已置顶。

#### 示例代码

```java
RongCoreClient.getInstance().getConversationTopStatusInTag(conversationIdentifier, tagId,
    new IRongCoreCallback.ResultCallback<Boolean>() {
            /**
             * 获取成功回调
             */
            @Override
            public void onSuccess(Boolean bool) {

            }
            /**
             * 获取失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });
```

### 是否开启同步空置顶会话

> 此功能在 5.10.0 版本起开始支持。

在您卸载并重新安装应用或更换设备登录时，融云允许您决定是否希望保留会话列表中置顶的会话，包括那些尚未包含任何消息的空会话。默认情况下，这些置顶状态将被保留。

您可以在 SDK 初始化过程中选择是否启用同步置顶会话的功能。通过调用 `InitOption` 类的 `enableSyncEmptyTopConversation` 方法，并传入 `true` 或 `false` 作为参数，来启用或禁用此功能。默认情况下，此功能是关闭的。

#### 示例代码

```java
InitOption initOption = new InitOption.Builder()
    .enableSyncEmptyTopConversation(true)
    .build();
RongCoreClient.init(context, appKey, initOption);
```

<!-- links -->
[会话标签]: ./tag.md
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[Conversation]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/index.html
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[ConversationIdentifier]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation-identifier/index.html
[ConversationTagInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation-tag-info/index.html
[setConversationToTop]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/set-conversation-to-top.html

