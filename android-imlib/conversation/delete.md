---
title: 删除会话
sidebar_position: 40
---

# 删除会话

如需从会话列表中删除一个会话或多个会话，可以通过 SDK 删除会话功能实现。客户端的会话列表是根据本地消息生成的，删除会话操作指的是删除本地会话。

## 删除指定会话

调用 `removeConversation` 可实现软删除的效果。该接口不会删除会话内的消息，仅将该会话项目从 SDK 的会话列表中移除。成功删除会话后，App 可以刷新 UI，不再向用户展示该会话项目。

#### 接口

```java
RongIMClient.getInstance().removeConversation(conversationType, targetId, ResultCallback<Boolean>)
```

#### 参数说明

| 参数 | 类型 | 说明 |
|:-----|:-----|:-----|
| `conversationType` | [`ConversationType`] | 会话类型 |
| `targetId` | `String` | 会话 ID |
| `callback` | `ResultCallback<Boolean>` | 回调接口 |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";

RongIMClient.getInstance().removeConversation(conversationType, targetId, new  ResultCallback<Boolean>() {

    @Override
    public void onSuccess(Boolean success) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

如果会话内再来一条消息，该会话会重新出现在会话列表中，App 用户可查看会话内的历史消息和最新消息。

SDK 未提供同时删除指定会话项目和会话历史消息的接口。如需同时删除会话内的消息，您需在删除指定会话时同时调用删除消息的接口。详见[删除消息]。

## 按会话类型删除会话

如需清空某一类型的所有会话，例如清空所有群聊会话。SDK 支持按指定会话类型清空所有会话及会话内的本地历史消息，一次支持清空多个类型的会话。

#### 接口

```java
RongIMClient.getInstance().clearConversations(ResultCallback, mConversationTypes);
```

#### 参数说明

| 参数 | 类型 | 说明 |
|:-----|:-----|:-----|
| `callback` | `ResultCallback` | 移除会话是否成功的回调。 |
| `conversationTypes` | [`ConversationType`] | 需要清空的会话类型列表。不支持超级群。 |

#### 示例代码

```java
Conversation.ConversationType[] mConversationTypes = {
            Conversation.ConversationType.PRIVATE,
            Conversation.ConversationType.GROUP
};

RongIMClient.getInstance().clearConversations(new RongIMClient.ResultCallback() {

    @Override
    public void onSuccess(Object object) {

    }

    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }, mConversationTypes);
```

该接口仅支持清空会话在当前用户设备本地数据库内的消息。如果 App 已经开通**单群聊消息云端存储**服务，服务端保存的该用户的历史消息记录不受影响。如果该用户从服务端获取历史消息，会获取到在本地已删除的消息。如果需要同时删除服务端的会话历史消息，您可以在删除会话后同时调用可删除远端历史消息的接口。详见[删除消息]。


## 批量删除会话

根据会话对象批量在会话列表中删除指定会话（包括本地和云端会话），支持会话类型：单聊、群聊、系统、超级群、公众号。

#### 接口

```java
ChannelClient.getInstance().batchDeleteConversations(params, callback);
```

#### 参数说明

| 参数 | 类型                                | 说明        |
|:-----|:----------------------------------|:----------|
| `params` | `ConversationBatchDeletionParams` | 批量删除会话参数。 |
| `callback` | `OperationCallback`               | 结果回调。     |

#### ConversationBatchDeletionParams 属性说明

| 参数 | 类型                                | 说明               |
|:-----|:----------------------------------|:-----------------|
| `identifiers` | `List<ConversationIdentifier>` | 会话标识列表，最大数量为 20。 |
| `deleteRemotely` | `boolean`               | 是否删除远端会话。            |
| `deleteMessages` | `boolean`               | 是否删除该会话下的所有消息。            |



#### 示例代码

```java
List<ConversationIdentifier> identifiers = new ArrayList<>();
identifiers.add(ConversationIdentifier.obtain(Conversation.ConversationType.PRIVATE, "TargetId", ""));
identifiers.add(ConversationIdentifier.obtain(Conversation.ConversationType.GROUP, "TargetId", ""));

ConversationBatchDeletionParams params = new ConversationBatchDeletionParams();
params.setDeleteMessages(true);
params.setDeleteRemotely(false);
params.setIdentifiers(identifiers);

ChannelClient.getInstance().batchDeleteConversations(params, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
    }
});
```

## 删除全部会话

SDK 内部没有清除全部会话的方法，如有需要可通过以下任一方式实现：
<!--public-cloud-only start-->
- 如果应用涉及超级群业务时，您可以使用[按会话类型删除会话](#按会话类型删除会话)，传入所有会话类型。这种方式会清除本地消息。
- 从 5.20.0 版本开始，支持清除超级群类型会话，超级群功能需[提交工单]开启。 <!--public-cloud-only end-->
- 先获取会话列表，循环删除指定会话。

<!-- links -->
[删除消息]: ../message/delete.md
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[`ConversationType`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

