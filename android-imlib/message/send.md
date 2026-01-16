---
title: 发送消息
sidebar_position: 10
---

# 发送消息

本文主要描述了如何使用 IMLib SDK 向单聊会话、群聊会话、聊天室会话中发送消息。

:::tip
 - 如发送超级群消息，请参见超级群文档 [收发消息](../ultragroup/chat.md)。
 - 默认情况下，融云允许未加入聊天室的用户在聊天室中发送消息。若您希望未加入聊天室的用户不能通过客户端 SDK 在该聊天室发送消息，可在[融云控制台]，在 **IM 服务**>**功能配置**>**聊天室**，开启 **SDK 用户不在聊天室中不能发送消息**功能。**注意**：服务端发送聊天室消息的接口不受此配置影响。
 :::

## 消息内容类型简介

IMLib SDK 定义的 [Message] 对象的 `content` 属性中可包含两大类消息内容：普通消息内容和媒体消息内容。普通消息内容父类是 [MessageContent]，媒体消息内容父类是 [MediaMessageContent]。发送媒体消息和普通消息本质的区别为是否有上传数据过程。

| 功能             | 消息内容的类型     | 父类                | 描述                                                             |
|:---------------|:-------------------|:--------------------|:---------------------------------------------------------------|
| 文本消息         | [TextMessage]      | MessageContent      | 文本消息的内容。                                                  |
| 引用回复         | [ReferenceMessage] | MessageContent      | 引用消息的内容，用于实现引用回复功能。                             |
| 图片消息         | [ImageMessage]     | MediaMessageContent | 图片消息的内容，支持发送原图。                                     |
| GIF 消息         | [GIFMessage]       | MediaMessageContent | GIF 消息的内容。                                                  |
| 文件消息         | [FileMessage]      | MediaMessageContent | 文件消息的内容。                                                  |
| 语音消息         | [HQVoiceMessage]   | MediaMessageContent | 高清语音消息的内容。                                              |
| 提及他人（@ 消息） | 不适用             | 不适用              | @消息并非预定义的消息类型。详见[如何发送 @ 消息](#mentionedinfo)。 |

以上为 IMLib SDK 内置的部分消息内容类型。您还可以创建自定义的消息内容类型，并使用 `sendMessage` 方法或`sendMediaMessage` 方法发送。详见[自定义消息类型](./customize.md)。

> **重要**
>
> - 发送普通消息使用 `sendMessage` 方法，发送媒体消息使用 `sendMediaMessage` 方法。
> - 客户端 SDK 发送消息存在频率限制，每秒最多只能发送 5 条消息。

本文使用 IMLib SDK 核心类 [RongCoreClient]（也可以用 [RongIMClient]）的发送消息方法。

## 普通消息

普通消息指文本消息、引用消息等不涉及媒体文件上传的消息。普通消息的消息内容为 `MessageContent` 的子类的消息，例如文本消息内容（[TextMessage]），或自定义类型的普通消息内容。

### 构造普通消息

在发送前，需要先构造 [Message] 对象。[conversationType] 字段为会话类型。`targetId` 表示会话的目标 ID。以下示例中构造了一条包含文本消息内容（[TextMessage]）的消息对象。

```java
String targetId = "Target ID";
ConversationType conversationType = Conversation.ConversationType.PRIVATE;

TextMessage messageContent = TextMessage.obtain("消息内容");

Message message = Message.obtain(targetId, conversationType, messageContent);
```

### 发送普通消息

如果 [Message] 对象包含普通消息内容，使用 IMLib SDK 核心类 [RongCoreClient]（也可以用 [RongIMClient]）的 `sendMessage` 方法发送消息。

```java
String pushContent = null;
String pushData = null;

RongCoreClient.getInstance().sendMessage(message, pushContent, pushData, new IRongCoreCallback.ISendMessageCallback() {

        @Override
        public void onAttached(Message message) {

        }

        @Override
        public void onSuccess(Message message) {

        }

        @Override
        public void onError(Message message, IRongCoreEnum.CoreErrorCode coreErrorCode) {

        }
    });

```

[sendMessage] 方法中直接提供了用于控制推送通知内容（`pushContent`）和推送附加信息（`pushData`）的参数。另一种方式是使用消息推送属性配置 [MessagePushConfig]，其中包含了 `pushContent` 和 `pushData`，并提供更多推送通知配置能力，例如标题、内容、图标、或其他第三方厂商个性化配置。详见[远程推送通知](#远程推送通知)。

关于是否需要配置输入参数或 `Message` 的 `messagePushConfig` 属性中 `pushContent`，请参考以下内容：

- 如果消息类型为[即时通讯服务预定义消息类型]中的[用户内容类消息格式]，例如 [TextMessage]，`pushContent` 可设置为 `null`。一旦消息触发离线推送通知时，远程推送通知默认使用服务端预置的推送通知内容。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。
- 如果消息类型为[即时通讯服务预定义消息类型]中通知类、信令类（"撤回命令消息" 除外），且需要支持远程推送通知，则必须填写 `pushContent`，否则收件人不在线时无法收到远程推送通知。如无需触发远程推送，可不填该字段。
- 如果消息类型为自定义消息类型，请参考[自定义消息如何支持远程推送](#自定义消息如何支持远程推送)。<!--public-cloud-only start-->
- 请注意，聊天室会话不支持离线消息机制，因此也不支持离线消息转推送。<!--public-cloud-only end-->

| 参数        | 类型                   | 说明                                                                                                                                                                                                                                                           |
|:------------|:-----------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]              | 要发送的消息体。必填属性包括会话类型（`conversationType`），会话 ID（`targetId`），消息内容（`content`）。详见[消息介绍] 中对 `Message` 的结构说明。                                                                                                                      |
| pushContent | String                 | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见下文[远程推送通知](#远程推送通知)。                                                                                                        |
| pushData    | String                 | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`。</p>您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见下文[远程推送通知](#远程推送通知)。 |
| callback    | [ISendMessageCallback] | 发送消息的回调。                                                                                                                                                                                                                                                |

通过 `RongCoreClient` 的 `sendMessage` 方法的回调处理程序 [ISendMessageCallback]，融云服务器始终会通知您的消息是否已发送成功。当因任何问题导致发送失败时，可通过回调方法返回异常。


:::tip


 - 关于如何个性化配置接收方离线时收到的远程推送通知，详见下文[远程推送通知](#远程推送通知)。
 - 自定义消息类型默认不支持离线消息转推送机制。如需支持，详见下文[自定义消息如何支持远程推送](#自定义消息如何支持远程推送)。
:::

## 媒体消息

媒体消息指图片、GIF、文件等需要处理媒体文件上传的消息。媒体消息的消息内容类型为 `MeidaMessageContent` 的子类，例如高清语音消息内容（[HQVoiceMessage]），或您自定义类型的媒体消息内容。

### 构造媒体消息

在发送前，需要先构造 [Message] 对象。媒体消息 `Message` 对象的 `content` 字段必须传入 [MediaMessageContent] 的子类对象，表示媒体消息内容。例如图片消息内容（[ImageMessage]）、GIF 消息内容（[GIFMessage]），或继承自 [MediaMessageContent] 的自定义媒体消息内容。

图片消息内容（[ImageMessage]）支持设置为发送原图。

```java
String targetId = "Target ID";
ConversationType conversationType = Conversation.ConversationType.ULTRA_GROUP;

Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);

Message message = Message.obtain(targetId, conversationType, mediaMessageContent);
```

在发送前，图片会被压缩质量，以及生成缩略图，在聊天界面中展示。GIF 无缩略图，也不会被压缩。

- **图片消息的缩略图**：SDK 会以原图 30% 质量生成符合标准大小要求的大图后再上传和发送。压缩后最长边不超过 240 px。缩略图用于在聊天界面中展示。
- **图片**：发送消息时如未选择发送原图，SDK 会以原图 85% 质量生成符合标准大小要求的大图后再上传和发送。压缩后最长边不超过 1080 px。

一般情况下不建议修改 SDK 默认压缩配置。如需调整 SDK 压缩质量，详见知识库文档[如何修改 SDK 默认的图片与视频压缩配置](https://help.rongcloud.cn/t/topic/1041)。

### 发送媒体消息

发送媒体消息需要使用 `sendMediaMessage` 方法。SDK 会为图片、小视频等生成缩略图，根据[默认压缩配置](https://help.rongcloud.cn/t/topic/1041)进行压缩，再将图片、小视频等媒体文件上传到融云默认的文件服务器（[文件存储时长](https://help.rongcloud.cn/t/topic/1049)），上传成功之后再发送消息。图片消息如已设置为发送原图，则不会进行压缩。

```java
String pushContent = null;
String pushData = null;

RongCoreClient.getInstance().sendMediaMessage(message, pushContent, pushData, new IRongCoreCallback.ISendMediaMessageCallback() {
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
    public void onError(final Message message, final IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});

```

[sendMediaMessage] 方法中直接提供了用于控制推送通知内容（`pushContent`）和推送附加信息（`pushData`）的参数。另一种方式是使用消息推送属性配置 [MessagePushConfig]，其中包含了 `pushContent` 和 `pushData`，并提供更多推送通知配置能力，例如标题、内容、图标、或其他第三方厂商个性化配置。详见[远程推送通知](#远程推送通知)。

关于是否需要配置输入参数或 `Message` 的 `messagePushConfig` 属性中 `pushContent`，请参考以下内容：

- 如果消息类型为[即时通讯服务预定义消息类型]中的[用户内容类消息格式]，例如 [HQVoiceMessage]，`pushContent` 可设置为 `null`。一旦消息触发离线推送通知时，远程推送通知默认使用服务端预置的推送通知内容。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。
- 如果消息类型为[即时通讯服务预定义消息类型]中通知类、信令类（"撤回命令消息" 除外），且需要支持远程推送通知，则必须填写 `pushContent`，否则收件人不在线时无法收到远程推送通知。如无需触发远程推送，可不填该字段。
- 如果消息类型为自定义消息类型，请参考[自定义消息如何支持远程推送](#自定义消息如何支持远程推送)。
- 请注意，聊天室会话不支持离线消息机制，因此也不支持离线消息转推送。

| 参数        | 类型                        | 说明                                                                                                                                                                                                                                                           |
|:------------|:----------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | [Message]                   | 要发送的消息体。必填属性包括会话类型（`conversationType`），会话 ID（`targetId`），消息内容（`content`）。详见[消息介绍] 中对 `Message` 的结构说明。                                                                                                                      |
| pushContent | String                      | 修改或指定远程消息推送通知栏显示的内容。您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见下文[远程推送通知](#远程推送通知)。                                                                                                        |
| pushData    | String                      | 远程推送附加信息。对端收到远程推送消息时，可通过以下方法获取该字段内容：<p>`io.rong.push.notification.PushNotificationMessage#getPushData()`。</p>您也可以在 `Message` 的推送属性（`MessagePushConfig`）中配置，会覆盖此处配置，详见下文[远程推送通知](#远程推送通知)。 |
| callback    | [ISendMediaMessageCallback] | 发送媒体消息的回调                                                                                                                                                                                                                                             |

通过 `RongCoreClient` 的 `sendMediaMessage` 方法的回调处理程序 [ISendMediaMessageCallback]，融云服务器始终会通知媒体文件上传进度，以及您的消息是否已发送成功。当因任何问题导致发送失败时，可通过回调方法返回异常。

发送媒体消息的方法默认将媒体文件上传到融云的文件服务器，您可以在客户端应用程序 [下载媒体消息文件](./download-media-file.md)。融云对上传媒体文件大小进行了限制，GIF 大小限制为 2 MB，文件上传限制为 100 MB。如果您需要提高 GIF 文件上限，可[提交工单]，文件上传的上限不可调。


:::tip


 - 关于如何个性化配置接收方离线时收到的远程推送通知，详见下文[远程推送通知](#远程推送通知)。
 - 自定义消息类型默认不支持离线消息转推送机制。如需支持，详见下文[自定义消息如何支持远程推送](#自定义消息如何支持远程推送)。
:::

### 发送媒体消息并且上传到自己的服务器

您可以直接发送您服务器上托管的文件。将媒体文件的 URL（表示其位置）作为参数，在构建媒体消息内容时传入。在这种情况下，您的文件不会托管在融云服务器上。当您发送带有远程 URL 的文件消息时，文件大小没有限制，您可以直接使用 `sendMessage` 方法发送消息。

如果您希望 SDK 在您上传成功后发送消息，您可以使用 `sendMediaMessage` 方法，在回调接口 [ISendMediaMessageCallbackWithUploader] 的 `onAttached` 回调方法中自行实现媒体文件上传，并在上传成功后通知 SDK，提供回媒体文件的远端地址。SDK 会在收到上传成功的通知后发送消息。

```java
String path = "file://图片的路径";
Uri localUri = Uri.parse(path);

ConversationType conversationType = ConversationType.PRIVATE;
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

## 如何发送 @ 消息 {#mentionedinfo}

@消息在融云即时通讯服务中不属于一种预定义的消息类型（详见服务端文档 [消息类型概述]）。融云通过在消息中携带 `mentionedInfo` 数据，帮助 App 实现提及他人（@）功能。

消息的 `MessageContent` 中的 [MentionedInfo] 字段存储了携带所 @ 人员的信息。无论是 SDK 内置消息类型，或者您自定义的消息类型，都直接或间接继承了 [MessageContent] 类。

融云支持在向群组和超级群中发消息时，在消息内容中添加 `mentionedInfo`。消息发送前，您可以构造 `MentionedInfo`，并设置到消息的 `MessageContent` 中。

```java
List<String> userIdList = new ArrayList<>();
userIdList.add("userId1");
userIdList.add("userId2");
MentionedInfo mentionedInfo = new MentionedInfo(MentionedInfo.MentionedType.PART, userIdList, null);
TextMessage messageContent = TextMessage.obtain("文本消息");
messageContent.setMentionedInfo(mentionedInfo);
```

`MentionedInfo` 参数：

| 参数             | 类型           | 说明                                                                                                                                                                                                            |
|:-----------------|:---------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| type             | MentionedType  | （必填）指定 `MentionedInfo` 的类型。`MentionedType.ALL` 表示需要提及（@）所有人。`MentionedType.PART` 表示需要 @ 部分人，被提及的人员需要在 `userIdList` 中指定。                                                      |
| userIdList       | List\<String\> | 被提及（@）用户的 ID 集合。当 `type` 为 `MentionedType.PART` 时必填；从 5.3.1 版本开始，支持当 `type` 为 `MentionedType.ALL` 时同时在 `userIdList` 中提及部分人。接收端可通过 `mentionedInfo` 获取 `userIdList` 数据。 |
| mentionedContent | String         | 触发离线消息推送时，通知栏显示的内容。如果是 `NULL`，则显示默认提示内容（“有人 @ 你”）。@消息携带的 `mentionedContent` 优先级最高，会覆盖所有默认或自定义的 `pushContent` 数据。                                        |

以下示例展示了发送一条提及部分用户的文本消息，该消息发往一个群聊会话。

```java
List<String> userIdList = new ArrayList<>();
userIdList.add("userId1");
userIdList.add("userId2");
MentionedInfo mentionedInfo = new MentionedInfo(MentionedInfo.MentionedType.PART, userIdList, null);
TextMessage messageContent = TextMessage.obtain("文本消息");
messageContent.setMentionedInfo(mentionedInfo);
Message message = Message.obtain(mTargetId, ConversationType.GROUP, messageContent);
RongCoreClient.getInstance().sendMessage(message, null , null, new IRongCoreCallback.ISendMessageCallback(){
        /**
         * 消息发送前回调, 回调时消息已存储数据库
         * @param message 已存库的消息体
         */
        @Override
        public void onAttached(Message message) {

        }
        /**
         * 消息发送成功。
         * @param message 发送成功后的消息体
         */
        @Override
        public void onSuccess(Message message) {

        }

        /**
         * 消息发送失败
         * @param message   发送失败的消息体
         * @param errorCode 具体的错误
         */
        @Override
        public void onError(Message message, IRongCoreEnum.CoreErrorCode coreErrorCode) {

        }
});

```

IMLib SDK 接收消息后，您需要处理 @ 消息中的数据。您可以在获取 `Message`(消息对象)后，通过以下方法获取到消息对象携带的 `MentionedInfo` 对象。

```java
MentionedInfo mentionedInfo = message.getContent().getMentionedInfo();
```

## 远程推送通知

:::tip

 聊天室会话不支持离线消息机制，因此也不支持离线消息转推送。
:::


如果您的应用已经在融云配置第三方推送，在消息接收方离线时，融云服务端会根据消息类型、接收方支持的推送通道、接收方的免打扰设置等，决定是否触发远程推送。

远程推送通知一般会展现在系统的通知栏。融云内置的消息类型默认会在通知栏展现通知标题和通知内容。关于各类型消息的默认推送通知内容，详见[用户内容类消息格式]。

如果您需要个性化的离线推送通知，可以通过以下方式，修改或指定远程推送的通知标题、通知内容其他属性。

- [sendMessage] 和 [sendMediaMessage] 方法的输入参数中直接提供了用于控制推送通知内容（`pushContent`）参数。
- 如果您需要控制离线推送通知的更多属性，例如标题、内容、图标、或根据第三方厂商通道作个性化配置，请使用 [Message] 的推送属性（`setMessagePushConfig`）方法进行配置。消息推送属性中的 `pushContent` 配置会覆盖发送消息接口中的 `pushContent`。

### 配置消息推送属性

在发送消息时，您可以通过设置消息的 [MessagePushConfig] 对象，对单条消息的推送行为进行个性化配置。例如：

- 自定义推送标题、推送通知内容
- 自定义通知栏图标
- 添加远程推送附加信息
- 越过接收客户端的配置，强制在推送通知内显示通知内容
- 其他 APNs 或 Android 推送通道支持的个性化配置

相对于发送消息方法输入参数中的 `pushContent` 和 `pushData`，`MessagePushConfig` 中的配置具有更高优先级。发送消息时，如已配置 `MessagePushConfig`，则优先使用 `MessagePushConfig` 中的配置。

```java
Message message = Message.obtain(mTargetId, mConversationType, textMessage);
        MessagePushConfig messagePushConfig = new MessagePushConfig.Builder().setPushTitle(title)
                .setPushContent(content)
                .setPushData(data)
                .setForceShowDetailContent(forceDetail)
                .setAndroidConfig(new AndroidConfig.Builder()
                    .setNotificationId(id)
                    .setChannelIdHW(hw)
                    .setCategoryHW("IM")
                    .setChannelIdMi(mi)
                    .setChannelIdOPPO(oppo)
                    .setTypeVivo(vivo ? AndroidConfig.SYSTEM : AndroidConfig.OPERATE)
                    .setCategoryVivo("IM")
                    .build())
                .setIOSConfig(new IOSConfig(threadId, apnsId)).setTemplateId("")
                build();
        message.setMessagePushConfig(messagePushConfig);

        // 请根据消息类型调用对应的发送方法
```

`MessagePushConfig` 属性说明如下：

| 参数                   | 类型          | 说明                                                                                                                                                                                                                                                                                                                                     |
|:-----------------------|:--------------|:---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| disablePushTitle       | boolean       | 是否屏蔽通知标题。此属性只针目标用户为 iOS 平台时有效，Android 第三方推送平台的通知标题为必填项，所以暂不支持。                                                                                                                                                                                                                              |
| pushTitle              | String        | 推送标题，此处指定的推送标题优先级最高。如不设置，可参考 [用户内容类消息格式] 中对各内置消息类型默认**推送通知标题**与**推送通知内容**的说明。                                                                                                                                                                                               |
| pushContent            | String        | 推送内容。此处指定的推送内容优先级最高。如不设置，可参考 [用户内容类消息格式] 中对各内置消息类型默认**推送通知标题**与**推送通知内容**的说明。                                                                                                                                                                                               |
| pushData               | String        | 远程推送附加信息。如不设置，则使用消息发送参数中设置的 `pushData` 值。对端收到远程推送通知时，可通过以下方法获取该字段内容：`io.rong.push.notification.PushNotificationMessage#getPushData()`。                                                                                                                                                |
| forceShowDetailContent | boolean       | 是否越过目标客户端的配置，强制在推送通知内显示通知内容（`pushContent`）。<br/><br/>客户端设备可通过 `setPushContentShowStatus` 设置在接收推送通知时仅显示类似「您收到了一条通知」的提醒。发送消息时，可设置 `forceShowDetailContent` 为 1 越过该配置，强制目标客户端在此条消息的推送通知中显示推送内容。                                           |
| iOSConfig              | IOSConfig     | iOS 平台推送通知配置。目标端设备为 iOS 平台设备时，适用该配置。详细说明见 **`iOSConfig` 属性说明**。                                                                                                                                                                                                                                         |
| androidConfig          | AndroidConfig | Android 平台推送通知配置。目标端设备为 Android 平台设备时，适用该配置。详细说明见 **`AndroidConfig` 属性说明**。                                                                                                                                                                                                                             |
| templateId             | String        | 推送模板 ID。根据目标客户端用户通过 `RongIMClient` 中的 `setPushLanguageCode` 设置的语言环境，匹配模板 ID 所对应的模板中设置的语言内容进行推送。未匹配成功时使用默认内容进行推送。<br/><br/>模板内容在**控制台** > **自定义推送文案**中进行设置，具体操作请参见 [配置和使用自定义多语言推送模板](https://help.rongcloud.cn/t/topic/1030)。 |

- `iOSConfig` 属性说明

    | 参数              | 类型   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                                                       |
    |:------------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | threadId          | String | iOS 平台通知栏分组 ID，相同的 `threadId` 推送分为一组（iOS10 开始支持）。                                                                                                                                                                                                                                                                                                                                                                                                      |
    | apnsCollapseId    | String | iOS 平台通知覆盖 ID，`apnsCollapseId` 相同时，新收到的通知会覆盖老的通知，最大 64 字节（iOS10 开始支持）。                                                                                                                                                                                                                                                                                                                                                                       |
    | richMediaUri      | String | iOS 推送自定义的通知栏消息右侧图标 URL，需要 App 自行解析 `richMediaUri` 并实现展示。图片要求：大小 120 * 120px，格式为 png 或者 jpg 格式。SDK 从 5.2.4 版本开始支持携带该字段。                                                                                                                                                                                                                                                                                                 |
    | interruptionLevel | String | 适用于 iOS 15 及之后的系统。取值为 `passive`，`active`（默认），`time-sensitive`，或 `critical`，取值说明详见对应的 [APNs 的 interruption-level](https://developer.apple.com/documentation/usernotifications/setting_up_a_remote_notification_server/generating_a_remote_notification?language=objc) 字段。在 iOS 15 及以上版本中，系统的 “定时推送摘要”、“专注模式” 都可能导致重要的推送通知（例如余额变化）无法及时被用户感知的情况，可考虑设置该字段。SDK 5.6.7 及以上版本支持该字段。 |

- `AndroidConfig` 属性说明

    | 参数            | 类型   | 说明                                                                                                                                                                                                                                                                                                                                                                                                                                            |
    |:----------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | notificationId  | String | Android 平台 Push 唯一标识，目前支持小米、华为推送平台，默认开发者不需要进行设置，当消息产生推送时，消息的 `messageUId` 作为 `notificationId` 使用。                                                                                                                                                                                                                                                                                                  |
    | channelIdMi     | String | 小米推送通知渠道的 ID。详细使用方式及创建方法请参见第三方文档 [小米推送消息分类新规](https://dev.mi.com/console/doc/detail?pId=2422#_4)。                                                                                                                                                                                                                                                                                                         |
    | imageUrlMi      | String | （**由于小米官方已停止支持该能力，该字段已失效**）小米通知类型的推送所使用的通知图片 url。图片要求：大小120 * 120px，格式为 png 或者 jpg 格式。<br />此属性 5.1.7 及以上版本支持。支持 MIUI 国内版（国内版要求为 MIUI 12 及以上）和国际版。                                                                                                                                                                                                                |
    | channelIdHW     | String | 华为推送通知渠道的 ID。详细使用方式及创建方法请参见第三方文档 [华为自定义通知渠道](https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/android-custom-chan-0000001050040122)。更多通知渠道信息，请参见 [Android 官方文档](https://developer.android.com/training/notify-user/channels)。                                                                                                                                        |
    | imageUrlHW      | String | 华为推送通知中自定义的通知栏消息右侧小图片 URL，如果不设置，则不展示通知栏右侧图片。图标文件须小于 512 KB，图标建议规格大小：40dp x 40dp，弧角大小为 8dp，超出建议规格大小的图标会存在图片压缩或显示不全的情况。<br />此属性 5.1.7 及以上版本支持。                                                                                                                                                                                                      |
    | importanceHW    | String | 华为推送的消息提醒级别。`LOW` 表示通知栏消息预期的提醒方式为静默提醒，消息到达手机后，无铃声震动。`NORMAL` 表示通知栏消息预期的提醒方式为强提醒，消息到达手机后，以铃声、震动提醒用户。终端设备实际消息提醒方式将根据 `categoryHW` 字段取值、或者控制台配置的 `category` 字段取值，或者[华为智能分类]结果进行调整。SDK 5.1.3 及以上版本支持该字段。                                                                                                     |
    | categoryHW      | String | 华为推送通道的消息自分类标识，默认为空。category 取值必须为大写字母，例如 `IM`。App 根据华为要求完成[华为自分类权益申请] 或 [申请特殊权限] 后可传入该字段有效。详见华为推送官方文档[华为消息分类标准]。该字段优先级高于控制台为 App Key 下的**应用标识**配置的华为推送 Category。SDK 5.4.0 及以上版本支持该字段。                                                                                                                                   |
    | imageUrlHonor   | String | 荣耀推送通知中用户自定义的通知栏右侧大图标 URL，如果不设置，则不展示通知栏右侧图标。图标文件须小于 512 KB，图标建议规格大小：40dp x 40dp，弧角大小为 8dp，超出建议规格大小的图标会存在图片压缩或显示不全的情况。SDK 5.6.7 及以上版本支持该字段。                                                                                                                                                                                                         |
    | importanceHonor | String | 荣耀推送的 Android 通知消息分类，决定用户设备消息通知行为。`LOW` 表示资讯营销类消息。`NORMAL`（默认值）表示服务与通讯类消息。SDK 5.6.7 及以上版本支持该字段。                                                                                                                                                                                                                                                                                          |
    | typeVivo        | String | VIVO 推送服务的消息类别。可选值 `0`（运营消息） 和 `1`（系统消息）。该参数对应 VIVO 推送服务的 `classification` 字段，详见 [VIVO 推送消息分类说明]。                                                                                                                                                                                                                                                                                                    |
    | categoryVivo    | String | VIVO 推送服务的消息二级分类。例如 `IM`（即时消息）。该参数对应 VIVO 推送服务的 `category` 字段。详细的 category 取值请参见 [VIVO 推送消息分类说明]。如果指定二级分类 `categoryVivo`，必须同时指定 `typeVivo`（系统消息或运营消息）。请注意遵照 VIVO 官方要求，确保二级分类属于 VIVO 系统消息场景或运营消息场景下允许发送的内容。`categoryVivo` 字段优先级高于控制台为 App Key 下的**应用标识**配置的 VIVO 推送 Category。SDK 5.4.2 及以上版本支持该字段。 |
    | channelIdOPPO   | String | OPPO 推送通知渠道的 ID。详细使用方式及创建方法请参见第三方文档 [OPPO PUSH 通道升级说明](https://open.oppomobile.com/new/developmentDoc/info?id=11227)。                                                                                                                                                                                                                                                                                           |
    | channelIdFCM    | String | FCM 推送通知渠道的 ID。应用程序必须先创建一个具有此频道 ID 的频道，然后才能收到具有此频道 ID 的任何通知。更多信息请参见[安卓官方文档](https://developer.android.com/guide/topics/ui/notifiers/notifications?hl=zh-cn#ManageChannels)。                                                                                                                                                                                                              |
    | collapseKeyFCM  | String | FCM 推送的通知分组 ID。SDK 5.1.3 及以上版本支持该字段。**注意**，如使用该字段，请确保控制台的 FCM 推送配置中推送方式为**通知消息方式**。                                                                                                                                                                                                                                                                                                         |
    | imageUrlFCM     | String | FCM 推送的通知栏右侧图标 URL。如果不设置，则不展示通知栏右侧图标。SDK 5.1.3 及以上版本支持该字段。**注意**，如使用该字段，请确保控制台的 FCM 推送配置鉴权方式为**证书**，推送方式为**通知消息方式**。                                                                                                                                                                                                                                               |

### 自定义消息如何支持远程推送

融云为内置的消息类型默认支提供了远程通知标题和通知内容（详见[用户内容类消息格式]）。不过，如果您发送的是自定义类型的消息，则需要您自行提供 `pushContent` 字段，否则用户无法收到离线推送通知。具体如下：

- 如果消息类型为自定义消息类型，且需要支持离线推送通知，则必须向融云提供 `pushContent` 字段，否则用户无法收到离线推送通知。
- 如果消息类型为自定义消息类型，但不需要支持远程推送通知（例如通过自定义消息类型实现的 App 业务层操作指令），可将 `pushContent` 字段留空。

:::tip

 如果要自定义消息启用离线推送通知能力，请务必发送消息的入参或消息推送属性向融云提供 `pushContent` 字段内容，否则接收方用户无法收到离线推送通知。
:::


### 为消息禁用推送通知

在接收者未上线时，融云服务端默认触发推送服务，将消息通过推送通道下发（服务端是否触发推送也会收到应用级别、用户级别免打扰设置的影响）。

您的 App 用户可能希望在发送消息时就指定该条消息不需要触发推送，该需求可通过 `Message` 对象的 `MessageConfig` 配置实现。

以下示例中，我们将 `messageConfig` 的 `disableNotification` 设置为 `true` 禁用该条消息的推送通知。接收方再次上线时会通过融云服务端的离线消息缓存（最多缓存 7 天）自动收取单聊、群聊、系统会话消息。超级群消息因不支持离线消息机制，需要主动拉取。

```java
String targetId = "目标 ID";
ConversationType conversationType = ConversationType.PRIVATE;
TextMessage messageContent = TextMessage.obtain("消息内容");
Message message = Message.obtain(targetId, conversationType, messageContent);

message.setMessageConfig(new MessageConfig.Builder().setDisableNotification(true).build());
```

## 如何透传自定义数据

如果应用需要将自定义数据透传到对端，可通过以下方式实现：

- 消息内容 [MessageContent] 的附加信息字段，**该字段会随即时消息一并发送到对端**。接收方在线接收消息后，可从消息内容中获取该字段。请区别于 `Message.extra`，`Message.extra` 为本地操作的字段，不会被发往服务端。
- 远程推送附加信息 `pushData`。您可以在 [sendMessage] 和 [sendMediaMessage] 的输入参数中直接设置 `pushData`，也可以使用消息推送属性中的同名字段，后者中的 `pushData` 会覆盖前者。**`pushData` 仅会在消息触发离线远程推送时下发到对端设备**。对端收到远程推送通知时，可通过以下方法获取该字段内容：`io.rong.push.notification.PushNotificationMessage#getPushData()`。

## 如何处理消息发送失败

对于客户端本地会存储的消息类型（参见[消息类型概述]），如果触发（`onAttached`）回调，表示此时该消息已存入本地数据库，并已进入本地消息列表。

- 如果入库失败，您可以检查参数是否合法，设备存储是否可正常写入。
- 如果入库成功，但消息发送失败（`onError`），App 可以在 UI 临时展示这条发送失败的消息，并缓存 `onError` 抛出的 `Message` 实例，在合适的时机重新调用 `sendMessage` / `sendMediaMessage` 发送。请注意，如果不复用该消息实例，而是重发相同内容的新消息，本地消息列表中会存储内容重复的两条消息。

对于客户端本地不存储的消息，如果发送失败（`onError`），App 可缓存消息实例再重试发送，或直接新建消息实例进行发送。

## 如何实现转发、群发消息

**转发消息**：IMLib SDK 未提供转发消息接口。App 实现转发消息效果时，可调用发送消息接口发送。

**群发消息**：群发消息是指向多个用户发送消息。更准确地说，是向多个 Target ID<sup><a href="/guides/glossary/imglossary#TargetID">?</a></sup>发送消息，一个 Target ID 可能代表一个用户，一个群组，或一个聊天室。客户端 SDK 未提供直接实现群发消息效果的 API。您可以考虑以下实现方式：

- App 循环调用客户端的发送消息 API。请注意，客户端 SDK 限制发送消息的频率上限为每秒 5 条。
- App 在业务服务端接入即时通讯服务端 API（IM Server API），由 App 业务服务端群发消息。IM Server API 的 [发送单聊消息] 接口支持单次向多个用户发送单聊消息。您也可以通过 [发送群聊消息]、[发送聊天室消息]接口实现单次向多个群组或多个聊天室发送消息。

<!-- links -->
[融云控制台]: https://console.rongcloud.cn/agile/im/service/config#%E8%81%8A%E5%A4%A9%E5%AE%A4
[消息介绍]: ./introduction.md
[即时通讯服务预定义消息类型]: /platform-chat-api/message-about/about-message-types
[消息类型概述]: /platform-chat-api/message-about/about-message-types
[发送单聊消息]: /platform-chat-api/message/send-private
[发送群聊消息]: /platform-chat-api/message/send-group
[发送聊天室消息]: /platform-chat-api/message/send-chatroom
[用户内容类消息格式]: /platform-chat-api/message-about/objectname
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
<!-- Javadoc -->
[RongIMClient]: https://doc.rongcloud.cn/apidoc/imlib-android/latest/zh_CN/html/-android--i-m-lib--s-d-k/io.rong.imlib/-rong-i-m-client/index.html
[RongCoreClient]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html
[sendMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-message.html
[sendMediaMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/send-media-message.html
[HQVoiceMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-h-q-voice-message/
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[MediaMessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/MediaMessageContent.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[ReferenceMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-reference-message/
[ImageMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-image-message/
[GIFMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-g-i-f-message/
[FileMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-file-message/
[IRongCoreCallback.MediaMessageUploader]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-media-message-uploader/index.html
[MessagePushConfig]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-push-config/index.html
[MentionedInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-mentioned-info/index.html
<!-- callbacks -->
[ISendMessageCallback]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-i-send-message-callback/index.html
[ISendMediaMessageCallback]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-i-send-media-message-callback/index.html
[ISendMediaMessageCallbackWithUploader]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-i-send-media-message-callback-with-uploader/index.html
<!-- external -->
[VIVO 推送消息分类说明]: https://dev.vivo.com.cn/documentCenter/doc/359
[华为自分类权益申请]: https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/message-classification-0000001149358835#section893184112272
[申请特殊权限]: https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/faq-0000001050042183#section037425218509
[华为消息分类标准]: https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/message-classification-0000001149358835
[华为智能分类]: https://developer.huawei.com/consumer/cn/doc/development/HMSCore-Guides/message-classification-0000001149358835#ZH-CN_TOPIC_0000001652651372__li19162756181511

