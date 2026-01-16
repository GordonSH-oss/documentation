---
title: 退出聊天室
---

# 退出聊天室

退出聊天室支持以下几种情况：

- **被动退出聊天室**：聊天室具有离线成员自动踢出机制。该机制被触发时，融云服务端会将用户踢出聊天室。用户如被封禁，也会被踢出聊天室。
- **主动退出聊天室**：客户端提供 API，支持由用户主动退出聊天室。

## 被动退出聊天室

聊天室具有离线成员自动退出机制。用户离线后，如满足以下默认预设条件，融云服务端会自动将该用户踢出聊天室：

- 从用户离线开始 30 秒内，聊天室中产生第 31 条消息时，触发自动踢出。
- 或用户已离线 30 秒后，聊天室有新消息产生时，触发自动踢出。

:::tip

 - 默认预设条件均要求聊天室中必须要有新消息产生，否则无法触发踢出动作。如果聊天室中没有消息产生，则无法将异常用户踢出聊天室。
 - 如需修改默认行为对新消息的依赖，请提交工单申请开通**聊天室成员异常掉线实时踢出**。开通该服务后，服务端会通过 SDK 行为（要求 Android/iOS IMLib SDK 版本 ≧ 5.1.6，Web IMLib 版本 ≧ 5.3.2）判断用户是否处于异常状态，最迟 5 分钟可以将异常用户踢出聊天室。
 - 如需保护特定用户，即不自动踢出指定用户（如某些应用场景下可能希望用户驻留聊天室），可使用 Server API 提供的[聊天室用户白名单](/platform-chat-api/chatroom/whitelist/add-to-user-whitelist)功能。
:::


## 主动退出聊天室

### 客户端用户可主动退出聊天室。

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | `String` | 聊天室 ID |
| `callback` | `OperationCallback` | 回调接口 |



#### 示例代码

```java
String chatroomId = "聊天室 ID";

RongChatRoomClient.getInstance().quitChatRoom(chatroomId, new IRongCoreCallback.OperationCallback() {

    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});

```

### 退出聊天室时设置附加信息

退出聊天室时，您可以设置附加信息（`extra`）用于业务扩展。

#### 接口

```java
quitChatRoom(String chatRoomId, String extra, OperationCallback callback)
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | `String` | 聊天室 ID |
| `extra` | `String` | 附加信息，不能为 null |
| `callback` | `OperationCallback` | 操作结果回调 |

#### 示例代码

```java
String chatroomId = "chatroomId"; // 请替换为实际的聊天室 ID
String extra = "附加信息"; // 按需填写实际的附加信息

RongChatRoomClient.getInstance().quitChatRoom(chatroomId, extra, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 退出聊天室成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
        // 退出聊天室失败，处理错误码
    }
});
```
