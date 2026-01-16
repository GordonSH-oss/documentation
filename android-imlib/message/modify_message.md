---
title: 消息修改
sidebar_position: 140
---

本文主要介绍如何使用 IMLib SDK 的消息修改功能。

:::tip
此功能从 5.26.0 版本开始支持。
:::

## 修改消息简介：

当您通过 IMLib SDK 成功发送消息后，如果发现消息内容有误，可以使用消息修改功能进行修正，修改后的内容会同步到接收者的消息记录中。
消息修改接口支持所有存储类型消息（文本、图片（包含 GIF）、语音（高清、普通）、视频、引用、文件、自定义消息）。

## 消息修改使用场景：

- 私聊场景中：用户消息发送成功后，对已发送的消息内容进行编辑。
- 群聊场景中：用户发送消息成功后，对已发送的消息内容进行编辑，或群组管理员编辑指定消息内容。

### 添加修改消息监听

消息发送方修改消息后，消息接收方可以通过添加消息修改回调接口来监听消息修改事件，并进行相应处理。

消息接收方需要使用 `addMessageModifiedListener` 添加一个监听器，用于监听已接收的消息被修改的事件。当接收到的某条消息被修改时，会通过此监听器回调。

```java
RongCoreClient.getInstance().addMessageModifiedListener(new IRongCoreListener.MessageModifiedListener() {
    @Override
    public void onMessageModified(List<Message> messages) {
        // 接收到的某条消息被修改时，会触发此回调。 
    }

    @Override
    public void onModifiedMessageSyncCompleted() {
        // 离线的消息修改记录同步完成时，会触发此回调。
    }
});
```

### 移除修改消息监听

为了避免内存泄露，请在不需要监听时调用 `removeMessageModifiedListener` 移除监听器。

```java
RongCoreClient.getInstance().removeMessageModifiedListener(listener);
```

## 修改消息

您可以通过调用 `modifyMessageWithParams` 接口修改已发送的消息。

```java
RongCoreClient.getInstance().modifyMessageWithParams(params,callback);
```

#### 参数说明

| 参数         | 类型                                    | 说明                                          |
|:------------|:----------------------------------------|:---------------------------------------------|
| `params`      | `ModifyMessageParams`                     | 消息修改参数对象                            |
| `callback`    | `IRongCoreCallback.ModifyMessageCallback` | 接口回调                                   |

#### ModifyMessageParams 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `messageUId`     | `String`         | 消息唯一 ID。 |
| `messageContent` | `MessageContent` | 修改后的消息体。 |

#### IRongCoreCallback.ModifyMessageCallback 参数说明

调用接口修改消息，会通过 `IRongCoreCallback.ModifyMessageCallback` 的 `onComplete` 方法回调结果。

| 返回值   | 方法                                           |  说明                                  |
|:--------|:----------------------------------------------|----------------------------------------|
| `void`  | `onComplete(errorCode,message)`   | 无论成功失败，均回调本接口。接口调用成功后，`message` 为修改后的消息，如果修改失败，`message` 为原始消息，如果 SDK 未初始化或接口参数校验不通过导致的失败，`message` 为 `NULL`      |


#### 示例代码

```java
        // 假设 originalMessage 是也要修改的原消息
        MessageContent content = originalMessage.getContent();
        // 修改消息体
        if (content instanceof TextMessage) {
            ((TextMessage)content).setContent("修改后的消息体");
        }

        ModifyMessageParams params = new ModifyMessageParams(originalMessage.getUId(),content);
        RongCoreClient.getInstance().modifyMessageWithParams(params, new IRongCoreCallback.ModifyMessageCallback() {
            @Override
            public void onComplete(IRongCoreEnum.CoreErrorCode errorCode, Message message) {
                // 处理结果
            }
        });
```


## 批量查询需要刷新的引用消息

由于引用消息 `ReferenceMessage` 缓存了原文本消息的内容，修改原文本消息不会主动更新引用消息中缓存的消息内容，在显示引用消息时，您可以通过调用 `refreshReferenceMessageWithParams` 接口刷新引用消息，将引用消息中缓存的文本内容刷新成最新的消息内容。

#### 接口

```java
RongCoreClient.getInstance().refreshReferenceMessageWithParams(params,callback);
```

#### 参数说明

| 参数         | 类型                                    | 说明                                          |
|:------------|:----------------------------------------|:---------------------------------------------|
| `params`      | `RefreshReferenceMessageParams`                     | 参数信息                            |
| `callback`    | `IRongCoreCallback.RefreshReferenceMessageCallback` | 接口回调                                   |

#### RefreshReferenceMessageParams 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `conversationIdentifier`     | `ConversationIdentifier`         | 会话标识。 |
| `messageUIds` | `List<String>` | 引用消息唯一 ID 数组，最多 20 个. |

#### IRongCoreCallback.RefreshReferenceMessageCallback 参数说明

调用批量查询需要刷新的引用消息接口，会通过 `IRongCoreCallback.RefreshReferenceMessageCallback` 回调结果。

| 返回值   | 方法                                           |  说明                                  |
|:--------|:----------------------------------------------|----------------------------------------|
| `void`  | `onLocalMessageBlock(msgList)`   | 本地存在的消息回调。                                   |
| `void`  | `onRemoteMessageBlock(msgList)`  | 本地不存在的消息，会自动加载远端消息，并回调结果。          |
| `void`  | `onError(errorCode)`             | 失败回调。                                           |



#### MessageResult 属性说明
| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `messageUId`     | `String`         | 查询的消息 UID。 |
| `message` | `Message` | 查到的消息。 |
| `code` | `IRongCoreEnum.CoreErrorCode` | 错误码。 |

#### 示例代码

```java
        RefreshReferenceMessageParams params = new RefreshReferenceMessageParams(conversationIdentifier,messageUIds);

        RongCoreClient.getInstance().refreshReferenceMessageWithParams(params, new IRongCoreCallback.RefreshReferenceMessageCallback() {
            @Override
            public void onLocalMessageBlock(List<MessageResult> msgList) {
                for (MessageResult messageResult : msgList) {
                    if (messageResult.getCode() == IRongCoreEnum.CoreErrorCode.SUCCESS) {
                        Message  message = messageResult.getMessage();
                        // 刷新 UI
                        // ......
                    }
                }
            }

            @Override
            public void onRemoteMessageBlock(List<MessageResult> msgList) {
                for (MessageResult messageResult : msgList) {
                    if (messageResult.getCode() == IRongCoreEnum.CoreErrorCode.SUCCESS) {
                        Message  message = messageResult.getMessage();
                        // 刷新 UI
                        // ......
                    }
                }
            }

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
                // 错误处理
            }
        });
```

## 设置会话编辑草稿

:::tip
此功能从 5.28.0 版本开始支持。
:::

当用户正在编辑消息时，您可以使用 `saveEditedMessageDraft` 方法保存编辑草稿。这样用户可以在稍后继续编辑该消息，提升用户体验。

#### 接口

```java
RongCoreClient.getInstance().saveEditedMessageDraft(draft, identifier, callback);
```

#### 参数说明

| 参数         | 类型                                    | 说明    |
|:------------|:----------------------------------------|:------|
| `draft`      | `EditedMessageDraft`                     | 草稿信息，包含消息 UID 和草稿内容 |
| `identifier`      | `ConversationIdentifier`                     | 会话标识，指定保存草稿的目标会话 |
| `callback`    | `IRongCoreCallback.OperationCallback` | 接口回调，返回操作结果 |

#### EditedMessageDraft 属性说明

| 属性 | 类型 | 说明      |
| :--- | :--- |:--------|
| `messageUId`     | `String`         | 消息 UID，标识要编辑的消息 |
| `content` | `String` | 草稿内容，用户当前编辑的文本   |

#### 示例代码

```java
EditedMessageDraft draft = new EditedMessageDraft("MSG_UID", "CONTENT");
ConversationIdentifier identifier = ConversationIdentifier.obtain(Conversation.ConversationType.PRIVATE, "Target_ID", "");
RongCoreClient.getInstance().saveEditedMessageDraft(draft, identifier, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode code) {
    }
});
```

## 查询会话编辑草稿

:::tip
此功能从 5.28.0 版本开始支持。
:::

您可以使用 `getEditedMessageDraft` 方法获取指定会话中保存的编辑草稿。当用户重新进入编辑页面时，可以恢复之前保存的草稿内容。

#### 接口

```java
RongCoreClient.getInstance().getEditedMessageDraft(identifier, callback);
```

#### 参数说明

| 参数         | 类型                                    | 说明    |
|:------------|:----------------------------------------|:------|
| `identifier`      | `ConversationIdentifier`                     | 会话标识，指定要查询草稿的目标会话 |
| `callback`    | `IRongCoreCallback.ResultCallback` | 接口回调，返回查询到的草稿信息 |

#### 示例代码

```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(Conversation.ConversationType.PRIVATE, "Target_ID", "");
RongCoreClient.getInstance().getEditedMessageDraft(identifier, new IRongCoreCallback.ResultCallback<EditedMessageDraft>() {
    @Override
    public void onSuccess(EditedMessageDraft draft) {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode code) {
    }
});
```

## 删除会话编辑草稿

:::tip
此功能从 5.28.0 版本开始支持。
:::

当用户完成消息编辑或不再需要草稿时，您可以使用 `clearEditedMessageDraft` 方法删除指定会话的编辑草稿，释放存储空间。

#### 接口

```java
RongCoreClient.getInstance().clearEditedMessageDraft(identifier, callback);
```

#### 参数说明

| 参数         | 类型                                    | 说明    |
|:------------|:----------------------------------------|:------|
| `identifier`      | `ConversationIdentifier`                     | 会话标识，指定要删除草稿的目标会话 |
| `callback`    | `IRongCoreCallback.OperationCallback` | 接口回调，返回删除操作结果 |

#### 示例代码

```java
ConversationIdentifier identifier = ConversationIdentifier.obtain(Conversation.ConversationType.PRIVATE, "Target_ID", "");
RongCoreClient.getInstance().clearEditedMessageDraft(identifier, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode code) {
    }
});
```

<!-- links -->
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
