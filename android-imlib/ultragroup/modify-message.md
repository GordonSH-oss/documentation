---
title: 修改消息
sidebar_position: 160
---

# 修改消息

您在成功发送超级群消息后，可以主动修改已发送消息的消息内容。

## 监听远端用户的消息变更

您可以通过设置 `setUltraGroupMessageChangeListener` 监听器，来监听远端用户对消息的修改操作，应用程序可以通过 `onUltraGroupMessageModified` 方法收到通知。消息被修改后，`Message#isHasChanged()` 返回 `true`。

```java
//超级群消息变化通知
interface UltraGroupMessageChangeListener {
  //消息扩展更新，删除
  void onUltraGroupMessageExpansionUpdated(List<Message> messages);
  //消息内容发生变更
  void onUltraGroupMessageModified(List<Message> messages);
  //消息撤回
  void onUltraGroupMessageRecalled(List<Message> messages);
}

//设置超级群消息变化监听
public void setUltraGroupMessageChangeListener(IRongCoreListener.UltraGroupMessageChangeListener listener)

```

## 修改本端用户已发消息的内容

本端用户发送消息成功后，服务端会返回消息的 UID。如果需要修改消息内容，可以使用 `modifyUltraGroupMessage` ，传入待修改消息的 Message UID 和新的消息内容进行修改。消息被修改后，`Message#isHasChanged()` 会返回 `true`。

:::tip

- 消息类型无法修改。如果改前为文本消息，则传入的新消息内容必须为 `TextMessage` 类型。
- 无法修改他人发送的消息。

:::

#### 示例代码

```java
TextMessage newContent = TextMessage.obtain("修改后的文本消息内容");

ChannelClient.getInstance().modifyUltraGroupMessage(msgUid, newContent,

    new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() { }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode ErrorCode) { }
});
```

