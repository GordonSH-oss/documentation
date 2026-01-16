---
title: 设置消息内容
sidebar_position: 70
---

# 设置消息内容

当您需要设置某条消息的 `MessageContent` 与可搜索的内容时，可以使用该功能。

:::tip
此功能从 5.34.0 版本开始支持。
:::

:::warning 注意
SDK 不会验证传入的 `MessageContent` 和 `searchWords` 的正确性，您需要自行确保：
- 传入的 `MessageContent` 符合融云对 [Message] 的定义和规范
-  `MessageContent` 与 `objectName` 的类型相匹配
- `searchWords` 内容准确且符合您的业务需求

如传入不正确的内容，可能导致该消息在数据库中存在脏数据，影响消息的正常显示和搜索功能。
:::

消息内容如 `TextMessage`、`ImageMessage` 等都属于 `Message` 中的 `MessageContent`。

## 设置某条消息的 MessageContent 与可搜索的内容

您可以通过 `setMessageContent` 设置某条消息的消息内容（`MessageContent`）以及可搜索内容（`searchWords`）。

### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `messageId` | `int` | 消息 ID。 |
| `content` | [MessageContent] | 消息内容。如 `TextMessage`、`ImageMessage` 等。 |
| `objectName` | `String` | 消息类型。如 `RC:TxtMsg`（文本消息）、`RC:FileMsg`（文件消息）、`RC:ImgTextMsg`（图文消息）等。 |
| `callback` | `IRongCoreCallback.ResultCallback` | 结果回调。 |

### 示例代码

```java
int messageId = 1;
TextMessage textMessage = TextMessage.obtain("消息内容");

MessageTag msgTag = textMessage.getClass().getAnnotation(MessageTag.class);
String objectName1 = msgTag.value();    // 第一种方式
String objectName2 = "RC:TxtMsg";       // 第二种方式

List<String> searchWords = new ArrayList<>();
searchWords.add("word1");
searchWords.add("word2");
        
RongCoreClient.getInstance().setMessageContent(messageId, textMessage, objectName1, searchWords, new IRongCoreCallback.ResultCallback<Boolean>() {
    @Override
    public void onSuccess(Boolean result) {
        // 设置成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 设置失败
    }
});
```

<!--api links -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[消息介绍]: ./introduction.md
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
