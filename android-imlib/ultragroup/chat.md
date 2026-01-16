---
title: 收发消息
sidebar_position: 80
---

# 收发消息

本文介绍了如何使用 IMLib SDK 接收跟发送超级群消息，超级群收发消息需要使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 下的方法。

## 前置条件

建议先阅读[超级群概述](./overview.md)和[超级群私有频道概述](./private-channel-about.md)，了解在 App 业务中**如何使用频道**和超级群频道功能特性。

- 通过服务端 API [创建超级群]
- 通过服务端 API [创建频道]，或使用默认频道 channelId `RCDefault`
- 通过服务端 API 将发送方用户[加入超级群]
- 如不确定发送方是否在超级群中，请通过服务端 API [查询用户是否为群成员]
- 如向超级群私有频道中发送消息，请确认已通过服务端 API [添加私有频道成员]

:::tip

 当用户在当前频道聊天页面发送与接收消息时，需要同时检查超级群 targetId 和频道 channelId。如果超级群 targetId 和频道 channelId 和当前频道聊天页面对应，才能在当前频道页面进行展示处理，否则就不处理。如果消息出现在其他聊天页面，一般是因为超级群 ID 或者频道 ID 发生错误。
:::

## 接收消息

超级群会话消息的接收方法跟单群聊会话的消息接口方法一致，具体如何使用请参考[接收消息]。

:::tip
超级群会话没有离线消息，如果想获取用户离线时候的消息需要您根据超级群会话最后一条消息去获取历史消息。
:::


## 构造消息对象

在发送消息前，需要构造 [Message] 对象，消息的 `conversationType` 字段必须填写超级群业务的会话类型 `ConversationType.ULTRA_GROUP`。消息的 `targetId` 字段表示超级群 ID，`channelId` 表示超级群频道 ID。

IMLib SDK 定义的 [Message] 属性中可包含两大类消息内容：普通消息内容和媒体消息内容。

普通消息内容父类是 `MessageContent` 的子类，例如文本消息（[TextMessage]）。

```java
String targetId = "超级群 ID";
ConversationType conversationType = Conversation.ConversationType.ULTRA_GROUP;
String channelId = "超级群频道 ID";

TextMessage messageContent = TextMessage.obtain("测试超级群");

Message message = Message.obtain(targetId, conversationType, channelId, messageContent);
```

媒体消息内容指 `MediaMessageContent` 的子类，例如图片消息（[ImageMessage]）、GIF 消息（[GIFMessage]）等。

```java
String targetId = "超级群 ID";
ConversationType conversationType = Conversation.ConversationType.ULTRA_GROUP;
String channelId = "超级群频道 ID";

Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);

Message message = Message.obtain(targetId, conversationType, channelId, mediaMessageContent);
```

:::tip

 - 发送普通消息请使用 `sendMessage` 方法，发送媒体消息请使用 `sendMediaMessage` 方法。
 - 客户端 SDK 发送消息存在频率限制，每秒最多只能发送 5 条消息。
 - 如果您的应用/环境在 2022.10.13 日及以后开通超级群服务，超级群业务中会包含一个 ID 为 `RCDefault` 的默认频道。如果发消息时不指定频道 ID，则该消息会发送到 `RCDefault` 频道中。
 - 如果您的应用/环境在 2022.10.13 日前已开通超级群服务，在发送消息时如果不指定频道 ID，则该消息不属于任何频道。融云支持客户调整服务至最新行为。该行为调整将影响客户端、服务端收发消息、获取会话、清除历史消息、禁言等多个功能。如有需要，请提交工单咨询详细方案。
:::


## 发送普通消息

普通消息指文本消息、引用消息等不涉及媒体文件上传的消息。普通消息的消息内容为 `MessageContent` 的子类的消息，例如文本消息内容（[TextMessage]），或自定义类型的普通消息内容。

```java
String targetId = "目标 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;//超级群会话类型
String channelId = "目标频道 ID";
TextMessage content = TextMessage.obtain("这是一条文本消息");
Message message = Message.obtain(targetId, conversationType, channelId, content);
```

发送超级群普通消息需要使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 中的提供的 `sendMessage` 方法。

#### 接口

```java
RongCoreClient.getInstance().sendMessage(message, pushContent, pushData, callback);
```

#### 参数说明

| 参数        | 类型                        | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|:------------|:----------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]                   | 要发送的消息实体，在消息实体中必须指定会话类型（`conversationType`），目标 ID（`targetId`），消息内容（`messageContent`）。详见[消息介绍]。                                                                                                                                                                                                                                                                                                                                                                          |
| pushContent | String                      | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#远程推送通知)。<ul><li>如果希望使用融云默认推送通知内容，可以填 `nil`。注意自定义消息类型无默认值。</li><li>如果要为该条消息指定个性化的离线推送通知，必须在以上任一处向融云提供 `pushContent` 字段内容。</li><li>如果您自定义的消息类型需要离线推送服务，必须在以上任一处向融云提供 `pushContent` 字段内容，否则用户无法收到离线推送通知。</li></ul> |
| pushData    | String                      | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`</p>。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知]。                                                                                                                                                                                                                        |
| callback    | [ISendMessageCallback] | 发送媒体消息的回调                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |

#### 示例代码

```java
RongCoreClient.getInstance().sendMessage(message, null, null, new IRongCoreCallback.ISendMessageCallback() {

        @Override
        public void onAttached(Message message) {

        }

        @Override
        public void onSuccess(Message message) {

        }

        @Override
        public void onError(Message message, IRongCoreEnum.CoreErrorCode errorCode) {

        }
    });

```

:::tip

 - 关于如何个性化配置接收方离线时收到的远程推送通知，详见下文[远程推送通知](#远程推送通知)。
 - 自定义消息类型默认不支持离线消息转推送机制。如需支持，详见下文[自定义消息如何支持远程推送](#自定义消息如何支持远程推送)。
:::

## 发送媒体消息

媒体消息 `Message` 对象的 `content` 字段必须传入 [MediaMessageContent] 的子类对象，表示媒体消息内容。例如图片消息内容（[ImageMessage]）、GIF 消息内容（[GIFMessage]），或继承自 [MediaMessageContent] 的自定义媒体消息内容。

图片消息（[ImageMessage]）支持设置为发送原图。

```java
String targetId = "目标 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;//超级群会话类型
String channelId = "目标频道 ID";
Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);

Message message = Message.obtain(targetId, conversationType, channelId, mediaMessageContent);
```

向超级群中发送媒体消息需要使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 中的 `sendMediaMessage` 方法。SDK 会为图片、小视频等生成缩略图，根据[默认压缩配置](https://help.rongcloud.cn/t/topic/1041)进行压缩，再将图片、小视频等媒体文件上传到融云默认的文件服务器（[文件存储时长](https://help.rongcloud.cn/t/topic/1049)），上传成功之后再发送消息。图片消息如已设置为发送原图，则不会进行压缩。

##### 接口

```java
RongCoreClient.getInstance().sendMediaMessage(message, pushContent, pushData, callback);
```

#### 参数说明

| 参数        | 类型                        | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                                                      |
|:------------|:----------------------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]                   | 要发送的消息实体，在消息实体中必须指定会话类型（`conversationType`），目标 ID（`targetId`），消息内容（`messageContent`）。详见[消息介绍]。                                                                                                                                                                                                                                                                                                                                                                          |
| pushContent | String                      | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知](../message/send.md#远程推送通知)。<ul><li>如果希望使用融云默认推送通知内容，可以填 `nil`。注意自定义消息类型无默认值。</li><li>如果要为该条消息指定个性化的离线推送通知，必须在以上任一处向融云提供 `pushContent` 字段内容。</li><li>如果您自定义的消息类型需要离线推送服务，必须在以上任一处向融云提供 `pushContent` 字段内容，否则用户无法收到离线推送通知。</li></ul> |
| pushData    | String                      | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`</p>。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见[自定义消息推送通知]。                                                                                                                                                                                                                        |
| callback    | [ISendMediaMessageCallback] | 发送媒体消息的回调                                                                                                                                                                                                                                                                                                                                                                                                                                                                                        |


#### 示例代码

```java
RongCoreClient.getInstance().sendMediaMessage(message, null, null, new IRongCoreCallback.ISendMediaMessageCallback() {
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

### 发送媒体消息并且上传到自己的服务器

您可以直接发送您服务器上托管的文件。将媒体文件的 URL（表示其位置）作为参数，在构建媒体消息内容时传入。在这种情况下，您的文件不会托管在融云服务器上。当您发送带有远程 URL 的文件消息时，文件大小没有限制，您可以直接使用 `sendMessage` 方法发送消息。

如果您希望 SDK 在您上传成功后发送消息，您可以使用 `sendMediaMessage` 方法，在回调接口 [ISendMediaMessageCallbackWithUploader] 的 `onAttached` 回调方法中自行实现媒体文件上传，并在上传成功后通知 SDK，提供回媒体文件的远端地址。SDK 会在收到上传成功的通知后发送消息。

```java
String path = "file://图片的路径";
Uri localUri = Uri.parse(path);

ConversationType conversationType = ConversationType.ULTRA_GROUP;//超级群会话类型
String targetId = "目标 ID";
ImageMessage imageMessage = ImageMessage.obtain(localUri, localUri);

Message message = Message.obtain(targetId, conversationType, imageMessage);

IRongCoreCallback.ISendMediaMessageCallbackWithUploader sendMediaMessageCallbackWithUploader =
            new IRongCoreCallback.ISendMediaMessageCallbackWithUploader() {
                @Override
                public void onAttached(
                        Message message, IRongCoreCallback.MediaMessageUploader uploader) {
                    /*上传图片到自己的服务器*/
                    uploadImg(imgMsg.getPicFilePath(), new UploadListener() {
                        @Override
                        public void onSuccess(String url) {
                            // 上传成功，回调 SDK 的 success 方法，传递回图片的远端地址
                            uploader.success(Uri.parse(url));
                        }


                        @Override
                        public void onProgress(float progress) {
                            //刷新上传进度
                            uploader.update((int) progress);
                        }


                        @Override
                        public void onFail() {
                            // 上传图片失败，回调 error 方法。
                            uploader.error();
                        }
                    });

                }

                @Override
                public void onError(
                        final Message message, final IRongCoreEnum.CoreErrorCode coreErrorCode) {
                    //发送失败
                }

                @Override
                public void onProgress(Message message, int progress) {
                    //发送进度
                }

                @Override
                public void onSuccess(Message message) {
                    //发送成功
                }

                @Override
                public void onCanceled(Message message) {
                    //发送取消
                }
            };

RongCoreClient.getInstance().sendMediaMessage(
        message, pushContent, pushData, sendMediaMessageCallbackWithUploader);
```


`sendMessage` 和 `sendMediaMessage` 中直接提供了用于控制推送通知内容（`pushContent`）和推送附加信息（`pushData`）的参数。不过，如果您需要更精细地控制离线推送通知，例如标题、内容、图标、或其他第三方厂商个性化配置，请使用消息推送属性进行配置，详见[自定义消息推送通知]。

- 如果发送的消息属于 SDK 内置的消息类型，例如 [ImageMessage]，这两个参数可设置为 `null`。一旦消息触发离线推送通知时，融云服务端会使用各个内置消息类型默认的 `pushContent` 值。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。
- 如果消息类型为自定义消息类型，且需要支持离线推送通知，则必须向融云提供 `pushContent` 字段，否则用户无法收到离线推送通知。
- 如果消息类型为自定义消息类型，但不需要支持远程推送通知（例如通过自定义消息类型实现的 App 业务层操作指令），可将 `pushContent` 字段留空。
- `Message` 的推送属性配置 `MessagePushConfig` 的 `pushContent` 和 `pushData` 会覆盖此处配置，并提供更多配置能力，例如自定义推送通知的标题。详见[自定义消息推送通知]。


## 如何发送 @ 消息 {#mentionedinfo}

@ 消息在融云即时通讯服务中不属于一种预定义的消息类型（详见服务端文档 [消息类型概述]）。您可以通过在消息中设置 `mentionedInfo` 数据，实现 @ 其他人的功能。

您可以在消息的 `MessageContent` 中的 [MentionedInfo] 字段中设置所 @ 人员的信息。无论是 SDK 内置消息类型，还是您自定义的消息类型，都直接或间接继承了 [MessageContent] 类，所以都支持设置 @ 人员的信息。


`MentionedInfo` 参数：

| 参数             | 类型           | 说明                                                                                                                                                                     |
|:-----------------|:---------------|:-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type             | MentionedType  | （必填）指定 `MentionedInfo` 的类型。`MentionedType.ALL` 表示需要提及（@）所有人。`MentionedType.PART` 表示需要 @ 部分人，被提及的人员需要在 `userIdList` 中指定。               |
| userIdList       | List\<String\> | 被提及（@）用户的 ID 集合。当 `type` 为 `MentionedType.PART` 时必填。                                                                                                        |
| mentionedContent | String         | 触发离线消息推送时，通知栏显示的内容。如果是 `NULL`，则显示默认提示内容（“有人 @ 你”）。@消息携带的 `mentionedContent` 优先级最高，会覆盖所有默认或自定义的 `pushContent` 数据。 |

以下示例展示了发送一条提及部分用户的文本消息，该消息发往一个超级群会话。

```java
List<String> userIdList = new ArrayList<>();
userIdList.add("userId1");
userIdList.add("userId2");
MentionedInfo mentionedInfo = new MentionedInfo(MentionedInfo.MentionedType.PART, userIdList, null);
TextMessage messageContent = TextMessage.obtain("文本消息");
messageContent.setMentionedInfo(mentionedInfo);

Message message = Message.obtain(targetId, conversationType, channelId, messageContent);

RongCoreClient.getInstance().sendMessage(message, null, null, new IRongCoreCallback.ISendMessageCallback() {

        @Override
        public void onAttached(Message message) {

        }

        @Override
        public void onSuccess(Message message) {

        }

        @Override
        public void onError(Message message, IRongCoreEnum.CoreErrorCode errorCode) {

        }
    });


```

IMLib SDK 接收消息后，您需要处理 @ 消息中的数据。您可以在获取 `Message`(消息对象)后，通过以下方法获取到消息对象携带的 `MentionedInfo` 对象。

```java
MentionedInfo mentionedInfo = message.getContent().getMentionedInfo();
```

## 远程推送通知

如果您的应用已经在融云配置第三方推送，在消息接收方离线时，融云服务端会根据消息类型、接收方支持的推送通道、接收方的免打扰设置等，决定是否触发远程推送。

远程推送通知一般会展现在系统的通知栏。融云内置的消息类型默认会在通知栏展现通知标题和通知内容。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。

### 配置消息推送属性

如果您需要个性化的离线推送通知，可以通过以下方式，修改或指定远程推送的通知标题、通知内容其他属性。

- 通过设置 [sendMessage] 和 [sendMediaMessage] 方法的输入参数中的 `pushContent` 控制推送通知内容。
- 如果您需要控制离线推送通知的更多属性，例如标题、内容、图标、或根据第三方厂商通道作个性化配置，请使用 [Message] 的推送属性（[MessagePushConfig]）进行配置。

:::tip
相对于发送消息时输入参数中的 `pushContent` 和 `pushData`，`MessagePushConfig` 中的配置具有更高优先级。发送消息时，如果您已经配置 `Message` 的 `messagePushConfig` 属性，则优先使用 `messagePushConfig` 中的配置。
:::


您可以在发送消息时提供 `MessagePushConfig` 配置，对单条消息的推送行为进行个性化配置。例如：

- 自定义推送标题、推送通知内容
- 自定义通知栏图标
- 添加远程推送附加信息
- 越过接收客户端的配置，强制在推送通知内显示通知内容
- 其他 APNs 或 Android 推送通道支持的个性化配置

关于 `MessagePushConfig` 的配置方式详见[自定义消息推送通知]。

### 自定义消息如何支持远程推送

IMLib SDK 为内置的消息类型实现了远程通知标题和通知内容（详见[用户内容类消息格式]）。如果您发送的是自定义类型的消息，则需要您自行提供 `pushContent` 字段，否则接收方用户无法收到离线推送通知。具体如下：

- 如果消息类型为自定义消息类型，且需要支持离线推送通知，请务必发送消息的入参或消息推送属性向 IMLib SDK 提供 `pushContent` 字段，否则用户无法收到离线推送通知。
- 如果消息类型为自定义消息类型，但不需要支持远程推送通知（例如通过自定义消息类型实现的 App 业务层操作指令），可将 `pushContent` 字段留空。

## 为消息禁用推送通知

在接收者未上线时，融云服务端默认触发推送服务，将消息通过推送通道下发（服务端是否触发推送也会收到应用级别、用户级别免打扰设置的影响）。

您的 App 用户可能希望在发送消息时就指定该条消息不需要触发推送，该需求可通过 `Message` 对象的 `MessageConfig` 配置实现。

以下示例中，我们将 `messageConfig` 的 `disableNotification` 设置为 `true` 禁用该条消息的推送通知。接收方再次上线时需要主动拉取。

```java
String targetId = "目标 ID";
String channelId = "目标频道 ID";
ConversationType conversationType = ConversationType.ULTRA_GROUP;
TextMessage messageContent = TextMessage.obtain("消息内容");
Message message = Message.obtain(targetId, conversationType, channelId, messageContent);

message.setMessageConfig(new MessageConfig.Builder().setDisableNotification(true).build());
```

## 如何透传自定义数据

如果您需要将自定义数据透传到对端，可通过以下方式实现：

- 消息内容 [MessageContent] 的附加信息字段 extra，**该字段会随即时消息一并发送到对端**。接收方在线接收消息后，可从消息内容中获取该字段。请区别于 `Message` 的 extra 字段，`Message` 的 extra 字段为本地操作的字段，不会被发往服务端。
- 远程推送附加信息 `pushData`。您可以在 [sendMessage] 和 [sendMediaMessage] 的输入参数中直接设置 `pushData`，也可以使用消息推送属性中的同名字段，后者中的 `pushData` 会覆盖前者。

:::tip
**`pushData` 仅会在消息触发离线远程推送时下发到对端设备**。对端收到远程推送通知时，可通过推送数据中的 `appData` 字段获取透传的数据内容。详见[集成远程推送](/android-imlib/push/overview)中的**获取推送数据**。
:::

## 如何处理消息发送失败

对于客户端本地会存储的消息类型（参见[消息类型概述]），如果触发（`onAttached`）回调，表示此时该消息已存入本地数据库，并已进入本地消息列表。

- 如果入库失败，您可以检查参数是否合法，设备存储是否可正常写入。
- 如果入库成功，但消息发送失败（`onError`），App 可以在 UI 临时展示这条发送失败的消息，并缓存 `onError` 抛出的 `Message` 实例，在合适的时机重新调用 `sendMessage` / `sendMediaMessage` 发送。请注意，如果不复用该消息实例，而是重发相同内容的新消息，本地消息列表中会存储内容重复的两条消息。

对于客户端本地不存储的消息，如果发送失败（`onError`），App 可缓存消息实例再重试发送，或直接新建消息实例进行发送。


<!-- links-->
[用户内容类消息格式]: /platform-chat-api/message-about/objectname
[消息类型概述]: /platform-chat-api/message-about/about-message-types
[消息介绍]: ../message/introduction.md
[自定义消息推送通知]: ../message/send.md#远程推送通知
<!-- api links -->
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[MediaMessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/MediaMessageContent.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[ImageMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-image-message/
[GIFMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-g-i-f-message/
[LocationMessage]: https://www.rongcloud.cn/docs/api/android/im_v5/latest/location/io/rong/imlib/location/message/LocationMessage.html
[ISendMessageCallback]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-i-send-message-callback/index.html
[ISendMediaMessageCallback]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-i-send-media-message-callback/index.html
[MentionedInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-mentioned-info/index.html
<!-- Server API -->
[创建超级群]: /platform-chat-api/ultragroup/create-ultragroup
[加入超级群]: /platform-chat-api/ultragroup/join-ultragroup
[创建频道]: /platform-chat-api/ultragroup/create-channel
[添加私有频道成员]: /platform-chat-api/ultragroup/private-channel/add-to-private-channel-users
[查询用户是否为群成员]: /platform-chat-api/ultragroup/query-member
[接收消息]: /android-imlib/message/receive
[MessagePushConfig]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-push-config/index.html?query=public%20class%20MessagePushConfig
[sendMessage]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-message.html
[sendMediaMessage]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-media-message.html
