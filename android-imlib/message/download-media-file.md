---
title: 下载媒体消息文件
sidebar_position: 40
---

# 下载媒体消息文件

IMLib SDK 提供了对媒体消息中的多媒体文件进行下载的功能。

## 下载媒体消息中的媒体文件

如果消息 [Message] 对象中包含媒体消息内容（指 `Message#getContent()` 返回媒体消息内容`MediaMessageContent`），其中可能携带了媒体文件地址。IMLib SDK 内置的媒体消息类型如下：

- [FileMessage]：文件消息内容
- [ImageMessage]：图片消息内容
- [GIFMessage]：GIF 消息内容
- [HQVoiceMessage]：高清语音消息
- [SightMessage]：小视频消息内容

在收到此类消息时，您可以使用 `downloadMediaMessage` 下载其中的媒体文件。

#### 示例代码

```java
RongCoreClient.getInstance().downloadMediaMessage(message, new IRongCallback.IDownloadMediaMessageCallback() {
            @Override
            public void onSuccess(Message message) {

            }

            @Override
            public void onProgress(Message message, int progress) {

            }

            @Override
            public void onError(Message message, IRongCoreEnum.CoreErrorCode code) {

            }

            @Override
            public void onCanceled(Message message) {

            }
        });
```

## 获取当前下载的文件信息

在调用 `downloadMediaMessage` 下载多媒体文件的过程中，可调用 `getDownloadInfo` 获取下载文件总大小、存储路径等信息。该接口仅在下载过程中调用时会返回正确信息。下载完成后调用该接口会返回 `null`。

`tag` 是文件唯一识别标志，可以使用 `messageId` 字符串。

```java
String tag = message.getMessageId();

RongCoreClient.getInstance().getDownloadInfo(tag, new IRongCoreCallback.ResultCallback<DownloadInfo>() {
            @Override
            public void onSuccess(DownloadInfo downloadInfo) {

            }

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode code) {

            }
        });
```

## 取消下载多媒体文件

使用 `cancelDownloadMediaMessage` 取消下载多媒体文件，需要传入当前正在下载的 `Message` 对象。

```java
RongCoreClient.getInstance().cancelDownloadMediaMessage(message, new IRongCoreCallback.OperationCallback() {
            @Override
            public void onSuccess() {

            }

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode code) {

            }
        });

```

## 暂停下载多媒体文件

您在下载媒体文件的过程中可以暂停下载。如需恢复下载，需重新调用下载方法，下载支持断点续传。

```java
RongCoreClient.getInstance().pauseDownloadMediaMessage(message, new IRongCoreCallback.OperationCallback() {
            @Override
            public void onSuccess() {

            }

            @Override
            public void onError(IRongCoreEnum.CoreErrorCode code) {

            }
        });
```



<!-- 链接区域 -->
[消息类型概述]: /platform-chat-api/message-about/about-message-types
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[ImageMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-image-message/
[GIFMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-g-i-f-message/
[FileMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-file-message/
[HQVoiceMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-h-q-voice-message/
[SightMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-sight-message/

