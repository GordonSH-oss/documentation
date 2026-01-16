---
title: 消息扩展
sidebar_position: 100
---

# 消息扩展

消息扩展功能可为消息对象（`Message`）增加基于 Key/Value 的状态标识。消息的扩展信息可在发送前、后设置或更新，可用于实现消息评论、礼物领取、订单状态变化等业务需求。

:::tip

 - 消息扩展仅支持单聊、群聊、<!--public-cloud-only start-->超级群<!--public-cloud-only end-->会话类型。不支持<!--public-cloud-only start-->聊天室和<!--public-cloud-only end-->系统会话。
 - 一条消息是否可携带或可设置扩展信息，由发送消息时 `Message` 的可扩展（`canIncludeExpansion`）属性决定，该属性必须在发送前设置，发送后无法修改。
 - 单条消息单次最多可设置 20 个扩展信息 KV 对，总计不可超过 300 个扩展信息 KV 对。在并发情况下如出现设置超过 300 个的情况，超出部分会被丢弃。
 - 如已开通历史消息云存储功能，从服务端获取的历史消息也会携带已设置的扩展信息。
 - 4.x SDK 从 4.0.3 版本开始支持消息扩展功能。
- 每次设置消息扩展将会产生内置通知消息，频繁设置扩展会产生大量消息。
:::


## 实现思路

以**订单状态变化**为例，可通过消息扩展改变消息显示状态。以订单确认为例：

1. 当用户购买指定产品下单后，商家需要向用户发送订单确认信息。可在发送消息时，将消息对象中的 `canIncludeExpansion` 属性设置为可扩展，同时设置用于标识订单状态的 Key 和 Value。例如，在用户未确认前，可用一对 Key/Value 表示该订单状态为「未确认」。
1. 用户点击确认（或其他确认操作）该订单消息后，订单消息状态需要变更为「已确认」。此时，可通过 `updateMessageExpansion` 更新此条消息的扩展信息，标识为已确认状态，同时更改本地显示的消息样式。
1. 发送方通过消息扩展状态监听，获取指定消息的状态变化，根据最新扩展信息显示最新的订单状态。

消息评论、礼物领取可参照以上实现思路：

- **礼物领取**：可通过消息扩展改变消息显示状态实现。例如，向用户发送礼物，默认为未领取状态，用户点击后可设置消息扩展为已领取状态。
- **消息评论**：可通过设置原始消息扩展信息的方式添加评论信息。

## 监听消息扩展数据变更

消息扩展变更的发起方调用 API 更新、删除扩展数据后，SDK 内部将向会话对端发送一条类型标识为 `RC:MsgExMsg` 的消息扩展信令消息。接收方可通过 `MessageExpansionListener` 监听器监听扩展数据变更，并进行相应处理。

建议在初始化 IMLib SDK 后，连接 IM 之前设置监听。

调用 `RongIMClient` 的 `setMessageExpansionListener` 方法设置消息扩展监听器。

#### 接口

```java
RongIMClient.getInstance().setMessageExpansionListener(listener);
```

#### 参数说明

| 参数     | 类型                     | 说明                                                         |
|:---------|:-------------------------|:-----------------------------------------------------------|
| listener | [MessageExpansionListener] | 消息扩展监听器。提供 `onMessageExpansionUpdate()`（消息扩展信息更改时）和 `onMessageExpansionRemove()`（消息扩展信息删除时）方法。 |

## 打开消息的可扩展属性（仅发送前）

构建新消息后，调用 `Message` 的 `setCanIncludeExpansion()` 属性打开或关闭某条消息的可扩展属性。

:::tip

 - 必须在发送消息前设置 `canIncludeExpansion` 属性。
:::

#### 接口

```java
    /**
     * 设置是否可以包含扩展信息
     *
     * @param canIncludeExpansion 是否可以包含扩展信息
     */
    public void setCanIncludeExpansion(boolean canIncludeExpansion) {
        this.canIncludeExpansion = canIncludeExpansion;
    }
```

#### 参数说明

| 参数                | 类型    | 说明                                                                                                                  |
|:--------------------|:--------|:--------------------------------------------------------------------------------------------------------------------|
| canIncludeExpansion | boolean | 该条消息是否可扩展。`true`：该消息允许设置扩展信息。`false`：该消息不允许设置扩展信息。消息发送之后不允许再更改该开关状态。 |

如果 App 先在本地插入消息，再发送本地已插入的消息（例如 App 业务要求在发送前审核消息内容），则必须在插入消息时设置此属性（详见[插入消息]）。本地插入成功后返回的消息不支持再通过 `setCanIncludeExpansion` 修改可扩展属性开关的状态。

#### 示例代码

```java
TextMessage textMessage = TextMessage.obtain("content");
Message message = Message.obtain("userId", Conversation.ConversationType.PRIVATE, textMessage);
message.setCanIncludeExpansion(true);
```

## 设置消息扩展数据（仅发送前）

如果消息已打开可扩展属性，可调用 `Message` 的 `setExpansion` 方法设置扩展数据。该接口仅可在消息发送前调用。

#### 参数说明


| 参数         | 类型                      | 说明                                                                                                                                                                                                                                                                          |
|:-------------|:--------------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| expansionMap | HashMap\<String, String\> | 消息扩展信息。单次最大设置扩展信息键值对 20 对。单条消息可设置最多 300 个扩展信息。<ul><li>Key 支持大小写英文字母、数字、 特殊字符`+ = - _` 的组合方式，不支持汉字。最大 32 个字符。</li><li>（SDK \< 5.2.0）Value 最大 64 个字符</li><li>（SDK ≧ 5.2.0）Value 最大 4096 个字符</li></ul> |

#### 示例代码

```java
/** 为消息设置扩展数据 */
HashMap expansionKey = new HashMap();
expansionKey.put("key", "value");
message.setExpansion(expansionKey);

/** 发送消息 */
RongIMClient.getInstance().sendMessage(message, null, null, new IRongCallback.ISendMessageCallback() {
    @Override
    public void onAttached(io.rong.imlib.model.Message message) {

    }

    @Override
    public void onSuccess(io.rong.imlib.model.Message message) {

    }

    @Override
    public void onError(io.rong.imlib.model.Message message, RongIMClient.ErrorCode errorCode) {

    }
});
```

上例中发送的消息会携带 `Expansion` 属性中的扩展数据。消息发送成功后，IMLib SDK 会将消息及扩展数据存入本地数据库。

如果发送的是本地数据库中已存在的消息（详见[插入消息]），请注意：

- （SDK ≧ 5.3.4）发送成功后 SDK 会刷新本地数据库中消息的扩展数据。
- （SDK \< 5.3.4）发送成功后 SDK 无法刷新本地数据库中消息的扩展数据。建议 App 先发送消息，再通过调用 `updateMessageExpansion` 更新本地与远端的扩展信息。对端可通过监听收到扩展数据更新。

## 更新扩展数据

您可以通过 `updateMessageExpansion()` 方法更新消息扩展信息。仅支持已打开可扩展属性的消息。更新消息扩展信息后，更新发起者应在成功回调里处理 UI 数据刷新，会话对端可通过消息扩展监听器收到更新的数据。

:::tip

 - 每次更新（或删除）消息扩展时，SDK 内部将向会话对端发送一条类型标识为 `RC:MsgExMsg` 的消息扩展信令消息。因此，频繁更新消息扩展会导致产生大量消息。
 - 每个终端在设置扩展信息时，如未达到上限都可以进行设置；在并发情况下，会出现设置超过 300 的情况，超出部分会被丢弃。
 - 如您的业务场景需要在应用服务端处理，也可以通过应用服务端来[设置单群聊扩展](/platform-chat-api/message/set-expansion)。
:::

#### 接口

```java
RongIMClient.getInstance().updateMessageExpansion(expansionMap,messageUId,callback);
```

#### 参数说明

| 参数       | 类型              | 说明                                                                                                                                                                                                                                    |
|:-----------|:------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| expansion  | Map               | 要更新的消息扩展信息键值对，类型是 HashMap。<ul><li>Key 支持大小写英文字母、数字、 特殊字符`+ = - _` 的组合方式，不支持汉字。最大 32 个字符。</li><li>（SDK \< 5.2.0）Value 最大 64 个字符</li><li>（SDK ≧ 5.2.0）Value 最大 4096 个字符</li></ul> |
| messageUId | String            | 消息唯一 Id，通过 `Message` 对象的 `getUId()`方法获取。                                                                                                                                                                                   |
| callback   | OperationCallback | 更新扩展消息回调                                                                                                                                                                                                                        |

#### 示例代码

```java
RongIMClient.getInstance()
        .updateMessageExpansion(
                expansionMap,
                messageUId,
                new RongIMClient.OperationCallback() {
                    @Override
                    public void onSuccess() {
                     // 更新发起者在这里处理更新扩展后的 UI 数据刷新

                    }

                    @Override
                    public void onError(RongIMClient.ErrorCode errorCode) {
                        Toast.makeText(
                                        getApplicationContext(),
                                        "设置失败，ErrorCode : " + errorCode.getValue(),
                                        Toast.LENGTH_LONG)
                                .show();
                    }
                });
```

## 删除扩展数据

您可以在消息发送后调用 `RongIMClient` 的 `removeMessageExpansion` 方法删除消息扩展信息中特定的键值对，仅支持删除已打开可扩展属性的消息中的扩展信息。删除消息扩展信息后，您应在成功回调里处理删除后的 UI 数据刷新，会话对端可通过消息扩展监听器收到通知。

#### 接口

```java
RongIMClient.getInstance().removeMessageExpansion(keyArray, messageUId, callback);
```

#### 参数说明

| 参数       | 类型              | 说明                                                   |
|:-----------|:------------------|:-----------------------------------------------------|
| keyArray   | List              | 消息扩展信息中待删除的 key 的列表，类型是 ArrayList。    |
| messageUId | String            | 消息唯一 Id，通过 `Message` 对象的 `getUId()` 方法获取。 |
| callback   | OperationCallback | 删除扩展消息的回调                                     |


<!-- links -->
[插入消息]: ./insert.md
[MessageExpansionListener]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-listener/-message-expansion-listener/index.html

