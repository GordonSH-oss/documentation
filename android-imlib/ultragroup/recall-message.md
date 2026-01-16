---
title: 撤回消息
sidebar_position: 170
---

# 撤回消息

超级群业务中，消息发送方可撤回已发送成功的消息。撤回成功后，服务端即删除原始消息。

默认情况下，融云对撤回消息的操作者不作限制。如需限制，可考虑以下方案：

- App 客户端自行限制撤回消息的操作者。例如，不允许 App 业务中的普通用户撤回他人发送的消息，允许 App 业务中的管理员角色撤回他人发送的消息。
- 如需避免用户撤回非本人发送的消息，可以[提交工单]申请打开**IMLib SDK 只允许撤回自己发送的消息**。从融云服务端进行限制，禁止用户撤回非本人发送的消息。

## 监听消息处理

您可以通过设置 `setUltraGroupMessageChangeListener` 监听器，监听远端用户对消息的撤回操作：

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

## 撤回指定消息

:::tip
只有已发送成功的消息可被撤回。
:::

### 接口

```java
class OperationCallback {
	onSuccess()
  onError(ErrorCode code)
}
//撤回消息
public void recallUltraGroupMessage(final Message message, final IRongCoreCallback.ResultCallback<RecallNotificationMessage> callback)
```

### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
| message | Message |消息对象|
|callback|IRongCoreCallback.ResultCallback\<RecallNotificationMessage\>|接口回调|

## 撤回指定消息并删除原始消息数据

:::tip
- 只有已发送成功的消息可被撤回。
- 此方法会同时删除客户端发送方、接收方的原始消息数据。
:::

### 接口

```java
class OperationCallback {
	onSuccess()
  onError(ErrorCode code)
}
//撤回消息
public void recallUltraGroupMessage(final Message message, final boolean isDelete, final IRongCoreCallback.ResultCallback<RecallNotificationMessage> callback)
```

### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:--------- |:------------------ |
| message | Message |消息对象|
| isDelete | Boolean | 指定移动端发送方与接收方是否需要从本地删除原始消息记录。为 `false` 时，移动端不会删除原始消息记录，会将消息内容替换为撤回提示（小灰条通知）。为 `true` 时，移动端会删除原始消息记录，不显示撤回提示（小灰条通知）。 |
|callback|IRongCoreCallback.ResultCallback\<RecallNotificationMessage\>|接口回调|


<!-- links -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create

