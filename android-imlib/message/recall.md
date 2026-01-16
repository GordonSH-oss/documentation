---
title: 撤回消息
sidebar_position: 70
---

您通过 IMLib SDK 成功发送了一条消息之后，可能发现消息内容错误等情况，希望将消息撤回，同时从接收者的消息记录中移除该消息。您可以使用撤回消息功能实现该需求。

撤回消息接口  `recallMessage` 会替换聊天记录中的原始消息为一条 objectName 为 `RC:RcNtf` 的撤回通知消息（[RecallNotificationMessage]）。同时消息接收方可通过 `OnRecallMessageListener` 监听器获取服务端通知，在收到对方已撤回通知时进行相应操作并刷新界面。

撤回通知消息的结构说明可参见服务端文档[通知类消息格式](/platform-chat-api/message-about/objectname-notification)。

默认情况下，融云对撤回消息的操作者不作限制。如需限制，可考虑以下方案：

- App 客户端自行限制撤回消息的操作者。例如，不允许 App 业务中的普通用户撤回他人发送的消息，允许 App 业务中的管理员角色撤回他人发送的消息。<!--public-cloud-only start-->
- 如需避免用户撤回非本人发送的消息，可以[提交工单]申请打开**IMLib SDK 只允许撤回自己发送的消息**。从融云服务端进行限制，禁止用户撤回非本人发送的消息。<!--public-cloud-only end-->

IMLib 对于撤回消息并没有做时间限制，现在主流的社交软件都会进行撤回时间限制，建议开发者自行做撤回时间限制。


## 监听消息被撤回事件

消息发送方调用 `recallMessage` 后，消息接收方可通过 `OnRecallMessageListener` 监听消息被撤回的事件，并进行相应处理。

消息接收方需要使用 `setOnRecallMessageListener` 设置一个监听器, 用于监听已接收的消息被撤回的事件。当接收到的某条消息被撤回时，会通过此监听器回调。

#### 接口

```java
RongIMClient.setOnRecallMessageListener(listener);
```

其中 `listener` 参数为消息被撤回监听器（`OnRecallMessageListener`），提供一个 `onMessageRecalled` 方法，如下：

#### 参数说明

| 返回值  | 方法                                                                                    |                                        |
|:--------|:----------------------------------------------------------------------------------------|----------------------------------------|
| boolean | onMessageRecalled(Message message, RecallNotificationMessage recallNotificationMessage) | 接收到的某条消息被撤回时，会触发此回调。 |


## 撤回消息

:::tip
从 5.16.1 版本开始支持撤回配置。
:::

您可以通过创建 `RecallMessageOption` 对象进行自定义配置
撤回指定消息，只有已发送成功的消息可被撤回。撤回的结果通过 `callback` 回调。

#### 接口

```java
RongCoreClient.getInstance().recallMessage(message, recallMessageOption, pushContent, callback);
```

#### 参数说明

| 参数        | 类型                                                      | 说明                                                                                                                                                                                  |
|:------------|:----------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | Message                                                   | 待撤回的消息对象                                                                                                                                                                      |
| recallMessageOption     | RecallMessageOption                                                   | 撤回消息的配置项象                                                                                                                                                                      |
| pushContent | String                                                    | 推送通知内容，推送消息时，通知栏里会显示的内容。 如果发送的是自定义消息，该字段必须填写，否则无法收到推送通知消息。SDK 中默认的消息类型，例如 `RC:TxtMsg`, `RC:VcMsg`, `RC:ImgMsg`，默认已经指定，不需要填写。 |
| callback    | IRongCallback.ResultCallback\<Message\> | 接口回调                                                                                                                                                                              |

#### 示例代码

```java
Message message = Message.obtain("123", ConversationType.GROUP, "12345");
 //撤回消息是否删除
boolean isDelete=true;
//是否为管理员
boolean isAdmin=false;
//是否屏蔽通知
boolean disableNotification=false;
RecallMessageOption recallMessageOption = new RecallMessageOption(isDelete, isAdmin, disableNotification);
RongCoreClient.getInstance().recallMessage(message, recallMessageOption, "", new IRongCoreCallback.ResultCallback<Message>() {
    @Override
    public void onSuccess(Message message) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```
## 撤回消息（废弃）

撤回指定消息，只有已发送成功的消息可被撤回。撤回的结果通过 `callback` 回调。`onSuccess` 中会返回替换后撤回提示小灰条消息（`RecallNotificationMessage`），您可以根据需要在界面展示。

#### 接口

```java
RongIMClient.getInstance().recallMessage(message, pushContent, callback);
```

#### 参数说明

| 参数        | 类型                                                      | 说明                                                                                                                                                                                  |
|:------------|:----------------------------------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| message     | Message                                                   | 待撤回的消息对象                                                                                                                                                                      |
| pushContent | String                                                    | Push 消息时，在通知栏里会显示这个字段。 如果发送的是自定义消息，该字段必须填写，否则无法收到 push 消息。sdk 中默认的消息类型，例如 RC:TxtMsg, RC:VcMsg, RC:ImgMsg，则不需要填写，默认已经指定 |
| callback    | IRongCallback.ResultCallback\<RecallNotificationMessage\> | 接口回调                                                                                                                                                                              |

#### 示例代码

```java
Message message = Message.obtain("123", ConversationType.GROUP, "12345");

RongIMClient.getInstance().recallMessage(message, " ", new RongIMClient.ResultCallback<RecallNotificationMessage>() {
    /**
     * 成功回调
     */
    @Override
    public void onSuccess(RecallNotificationMessage recallNotificationMessage) {

    }

    /**
     * 失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(RongIMClient.ErrorCode errorCode) {

    }
});
```

如果需要通过消息 messageId （本地数据库索引唯一值）或者 messageUId（服务器消息唯一 ID）撤回消息，可先 `getMessage` 以下方法获取待撤回的消息，再使用 `recallMessage` 撤回消息。


```java
    /**
     * 根据消息 id 获取消息体（数据库索引唯一值）。
     *
     * @param messageId 消息 id。
     * @param callback 回调。
     */
    public abstract void getMessage(final int messageId, final ResultCallback<Message> callback);

    /**
     * 通过全局唯一 id 获取消息实体。
     *
     * @param uid 全局唯一 id（服务器消息唯一 id）。
     * @param callback 获取消息的回调。
     */
    public abstract void getMessageByUid(final String uid, final ResultCallback<Message> callback);
```


<!--api links -->
[RecallNotificationMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-recall-notification-message/
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create

