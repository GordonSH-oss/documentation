---
title: 发送超级群定向消息
sidebar_position: 90
---

# 发送超级群定向消息


您可以给超级群频道中的指定的一个或多个成员发送普通消息与媒体消息，其他成员不会收到该消息。

:::tip
- 5.6.9 版本开始支持该能力。
- 单条定向消息的接收用户上限 300 个用户。
- 如果定向消息为 @ 消息，不支持 @所有人。
:::

## 发送定向普通消息

您可以使用 [sendDirectionalMessage] 在超级群中发送普通消息给指定频道中的指定用户。不在接收列表的用户不会收到这条消息。

请在消息对象中设置超级群会话类型、超级群 targetId 和 频道 channelId。频道 channelId 为空时默认向 `RCDefault` 频道发送消息。注意，`Message` 中不会保存接收用户的 userId 列表。

#### 接口

```java
RongCoreClient.getInstance().sendDirectionalMessage(message, userIds, pushContent, pushData,callback);
```

#### 参数说明

| 参数        | 类型                   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|:------------|:-----------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]              | 要发送的消息体。必填属性包括会话类型（`conversationType`），会话 ID（`targetId`），消息内容（`content`）。**注意**，群组定向消息要求会话类型为 `Conversation.ConversationType.GROUP`。                                                                                                                                                                                                                                                                                                                                       |
| userIds     | String[]               | 需要接收消息的用户列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| pushContent | String                 | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。<ul><li>如果希望使用融云默认推送通知内容，可以填 `null`。注意自定义消息类型无默认值。</li><li>如果要为该条消息指定个性化的离线推送通知，必须在以上任一处向融云提供 `pushContent` 字段内容。</li><li>如果您自定义的消息类型需要离线推送服务，必须在以上任一处向融云提供 `pushContent` 字段内容，否则用户无法收到离线推送通知。</li></ul> |
| pushData    | String                 | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`</p>。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。                                                                                                                                                                                                                         |
| callback    | [ISendMessageCallback] | 发送消息的回调。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                  |


#### 示例代码

```java
Conversation.ConversationType type = Conversation.ConversationType.GROUP;
String targetId = "123";
TextMessage content = TextMessage.obtain("定向消息文本内容");
Message message = Message.obtain(targetId, conversationType, channelId, messageContent);

String[] userIds = new String[]{"id_01", "id_02"};

RongCoreClient.getInstance().sendDirectionalMessage(message, userIds, pushContent, pushData, new IRongCoreCallback.ISendMessageCallback() {
            @Override
            public void onAttached(Message message) {

            }

            @Override
            public void onSuccess(Message message) {

            }

            @Override
            public void onError(final Message message, IRongCoreEnum.CoreErrorCode errorCode) {

            }
});
```

## 发送定向媒体消息

您可以使用 [sendDirectionalMediaMessage] 在超级群中发送媒体消息给指定频道中的指定用户。不在接收列表的用户不会收到这条消息。

请在消息对象中设置超级群会话类型、超级群 targetId 和 频道 channelId。频道 channelId 为空时默认向 `RCDefault` 频道发送消息。注意，`Message` 中不会保存接收用户的 userId 列表。

#### 接口

```java
RongCoreClient.getInstance().sendDirectionalMediaMessage(message, userIds, null, null, callback);
```

#### 参数说明


| 参数        | 类型                        | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                    |
|:------------|:----------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]                   | 要发送的消息体。必填属性包括会话类型（`conversationType`），会话 ID（`targetId`），消息内容（`content`）。**注意**，超级群定向消息要求会话类型为 `Conversation.ConversationType.GROUP`。                                                                                                                                                                                                                                                                                                                            |
| userIds     | String[]                    | 需要接收消息的用户列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                 |
| pushContent | String                      | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](./chat.md#自定义消息推送通知)。<ul><li>如果希望使用融云默认推送通知内容，可以填 `null`。注意自定义消息类型无默认值。</li><li>如果要为该条消息指定个性化的离线推送通知，必须在以上任一处向融云提供 `pushContent` 字段内容。</li><li>如果您自定义的消息类型需要离线推送服务，必须在以上任一处向融云提供 `pushContent` 字段内容，否则用户无法收到离线推送通知。</li></ul> |
| pushData    | String                      | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`</p>。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](./chat.md#自定义消息推送通知)。                                                                                                                                                                                                                         |
| callback    | [ISendMediaMessageCallback] | 发送媒体消息的回调                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |


#### 示例代码

```java
String targetId = "目标 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;
Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);
Message message = Message.obtain(targetId, conversationType, channelId, mediaMessageContent);

String[] userIds = new String[]{"id_01", "id_02"};

RongCoreClient.getInstance().sendDirectionalMediaMessage(message, userIds, null, null, new IRongCoreCallback.ISendMediaMessageCallback() {
    @Override
    public void onProgress(Message message, int i) {

    }

    @Override
    public void onCanceled(Message message) {

    }

    @Override
    public void onAttached(Message message) {

    }

    @Override
    public void onSuccess(Message message) {

    }

    @Override
    public void onError(final Message message, final IRongCoreEnum.CoreErrorCode errorCode) {

    }
});

```


<!-- 链接区域 -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
<!-- links -->
[用户内容类消息格式]: /platform-chat-api/message-about/objectname
<!-- Javadoc -->
[sendDirectionalMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-directional-message.html
[sendDirectionalMediaMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-directional-media-message.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[ImageMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-image-message/
<!-- callbacks -->
[ISendMessageCallback]: https://doc.rongcloud.cn/apidoc/imlib-android/latest/zh_CN/html/-android--i-m-lib--s-d-k/io.rong.imlib/-i-rong-callback/-i-send-message-callback/index.html
[ISendMediaMessageCallback]: https://doc.rongcloud.cn/apidoc/imlib-android/latest/zh_CN/html/-android--i-m-lib--s-d-k/io.rong.imlib/-i-rong-callback/-i-send-media-message-callback/index.html

