---
title: 扩展消息
sidebar_position: 180
---

# 扩展消息

消息扩展功能可为消息对象（`Message`）增加基于 Key/Value 的状态标识。消息的扩展信息可在发送前、后设置或更新，可用于实现消息评论、礼物领取、订单状态变化等业务需求。

:::tip
- 一条消息是否可携带或可设置扩展信息，由发送消息时 `Message` 的可扩展（`canIncludeExpansion`）属性决定，该属性必须在发送前设置，发送后无法修改。
 - 单条消息单次最多可设置 20 个扩展信息 KV 对，总计不可超过 300 个扩展信息 KV 对。在并发情况下如出现设置超过 300 个的情况，超出部分会被丢弃。
 - 如已开通历史消息云存储功能，从服务端获取的历史消息也会携带已设置的扩展信息。
 - 每次设置消息扩展将会产生内置通知消息，频繁设置扩展会产生大量消息。
:::

## 监听消息处理

您可以通过设置 `setUltraGroupMessageChangeListener` 方法来监听远端用户对消息扩展的修改操作：

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


## 设置、更新消息扩展信息

:::tip

 消息扩展功能要求在发送消息前设置该消息为可扩展消息。关于如何修改消息的的可扩展属性，请参考[消息扩展](../message/expansion.md)中的对 `setCanIncludeExpansion()` 的说明。
:::


```java
class OperationCallback {
	onSuccess()
  onError(ErrorCode code)
}

//更新消息扩展
public void updateUltraGroupMessageExpansion(final Map<String, String> expansion, final String messageUId, final IRongCoreCallback.OperationCallback callback)

//删除消息扩展
public void removeUltraGroupMessageExpansion(final String messageUId, final List<String> keyArray, final IRongCoreCallback.OperationCallback callback)
```


