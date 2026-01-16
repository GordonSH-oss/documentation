---
sidebar_position: 2
title: 发送群定向消息
---

# 发送群定向消息

可发送普通消息与媒体消息给群组中的指定的一个或多个成员，其他成员不会收到该消息。
<!--public-cloud-only start-->
## 开通服务

使用**发送群组定向消息**功能无需开通服务。注意，**如需将群组定向消息存入服务端历史消息记录**，需要开通以下服务：

- **单群聊消息云端存储**服务。您可前往控制台 IM 服务的 [服务购买](https://console.rongcloud.cn/agile/im/service/purchase#4)页面为当前使用的 App Key 开启服务。**IM 旗舰版**或**IM 尊享版**可开通该服务。具体功能与费用以[融云官方价格说明](https://www.rongcloud.cn/pricing)页面及[计费说明](https://help.rongcloud.cn/t/topic/123)文档为准。
- **群定向消息云存储**服务。您可在[融云控制台](https://console.rongcloud.cn/agile/im/service/config#%E5%85%A8%E5%B1%80%E6%B6%88%E6%81%AF)，开通**群定向消息云存储**。

默认情况下，客户端发送与接收的群定向消息默认都不会存入历史消息服务，因此客户端调用获取历史消息的 API 时，从融云服务端返回的结果中不会包含当前用户发送、接收的群组定向消息。<!--public-cloud-only end-->

## 发送群组定向普通消息

在群组中发送普通消息给群组中的指定用户。不在接收列表的其它用户不会收到这条消息。注意，`Message` 中仅保存群组 ID（Target ID），不会保存接收用户 ID 列表。


#### 接口
```java
RongCoreClient.getInstance().sendDirectionalMessage(message, userIds, pushContent, pushData, callback);
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
Message message = Message.obtain(targetId, conversationType, messageContent);

String[] userIds = new String[]{"id_01", "id_02"};

RongCoreClient.getInstance().sendDirectionalMessage(message, userIds, null, null, new IRongCoreCallback.ISendMessageCallback() {
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

`sendDirectionalMessage` 中直接提供了用于控制推送通知内容（`pushContent`）和推送附加信息（`pushData`）的参数。不过，如果您需要更精细地控制离线推送通知，例如标题、内容、图标、或其他第三方厂商个性化配置，请使用消息推送属性进行配置。

- 如果发送的消息属于 SDK 内置的普通消息类型，例如 [TextMessage]，这两个参数可设置为 `null`。一旦消息触发离线推送通知时，融云服务端会使用各个内置消息类型默认的 `pushContent` 值。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。
- 如果消息类型为自定义消息类型，且需要支持离线推送通知，则必须向融云提供 `pushContent` 字段，否则用户无法收到离线推送通知。
- 如果消息类型为自定义消息类型，但不需要支持远程推送通知（例如通过自定义消息类型实现的 App 业务层操作指令），可将 `pushContent` 字段留空。
- `Message` 的推送属性配置 `MessagePushConfig` 的 `pushContent` 和 `pushData` 会覆盖此处配置，并提供更多配置能力，例如自定义推送通知的标题。详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。

## 发送群组定向媒体消息

在群聊中发送多媒体消息给指定的单个或多个用户。注意，`Message` 中仅保存群组 ID（Target ID），不会保存接收用户 ID 列表。

#### 接口
```java
RongCoreClient.getInstance().sendDirectionalMediaMessage(message, userIds, pushContent, pushData, callback);
```

#### 参数说明

| 参数        | 类型                        | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                             |
|:------------|:----------------------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]                   | 要发送的消息体。必填属性包括会话类型（`conversationType`），会话 ID（`targetId`），消息内容（`content`）。**注意**，群组定向消息要求会话类型为 `Conversation.ConversationType.GROUP`。                                                                                                                                                                                                                                                                                                                                       |
| userIds     | String[]                    | 需要接收消息的用户列表。                                                                                                                                                                                                                                                                                                                                                                                                                                                                                          |
| pushContent | String                      | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。<ul><li>如果希望使用融云默认推送通知内容，可以填 `null`。注意自定义消息类型无默认值。</li><li>如果要为该条消息指定个性化的离线推送通知，必须在以上任一处向融云提供 `pushContent` 字段内容。</li><li>如果您自定义的消息类型需要离线推送服务，必须在以上任一处向融云提供 `pushContent` 字段内容，否则用户无法收到离线推送通知。</li></ul> |
| pushData    | String                      | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`</p>。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。                                                                                                                                                                                                                         |
| callback    | [ISendMediaMessageCallback] | 发送媒体消息的回调                                                                                                                                                                                                                                                                                                                                                                                                                                                                                               |


#### 示例代码
```java
String targetId = "目标 ID";
ConversationType conversationType = ConversationType.PRIVATE;
Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);

Message message = Message.obtain(targetId, conversationType, mediaMessageContent);

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

`sendDirectionalMediaMessage` 方法中直接提供了用于控制推送通知内容（`pushContent`）和推送附加信息（`pushData`）的参数。如果您需要更精细地控制离线推送通知，例如标题、内容、图标、或其他第三方厂商个性化配置，请使用消息推送属性进行配置，详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。

- 如果发送的消息属于 SDK 内置的媒体消息类型，例如 [ImageMessage]，这两个参数可设置为 `null`。一旦消息触发离线推送通知时，融云服务端会使用各个内置消息类型默认的 `pushContent` 值。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。
- 如果是自定义消息类型，且需要离线推送通知，则必须向融云提供 `pushContent` 字段，否则用户无法收到离线推送通知。
- 如果是自定义消息类型，但不需要离线推送通知（例如通过自定义消息类型实现的 App 业务层操作指令），可将 `pushContent` 字段留空。
- `Message` 的推送属性配置 `MessagePushConfig` 的 `pushContent` 和 `pushData` 会覆盖此处配置，并提供更多配置能力，例如自定义推送通知的标题。详见[自定义消息推送通知](../message/send.md#自定义消息推送通知)。


<!-- 链接区域 -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[服务购买]: https://console.rongcloud.cn/agile/im/service/purchase
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

