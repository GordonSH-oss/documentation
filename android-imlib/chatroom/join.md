---
title: 加入聊天室
---

# 加入聊天室

加入聊天室分为以下几种情况：

- （推荐）应用程序后端通过即时通讯服务端 API [创建聊天室]，将聊天室 ID 分发至客户端。客户端获取聊天室 ID 后可加入聊天室。
- （已废弃）应用程序通过直接调用客户端 SDK 加入聊天室方法，创建并加入该聊天室。这种方式已不推荐，相关 API 已于 5.6.3 版本废弃，未来可能会删除。
- SDK 断网重连后会自动重新加入聊天室，应用程序无需处理。

应用程序后端可以通过提前注册服务端回调[聊天室状态同步]，收取聊天室创建成功与成员加入等事件通知。客户端可以通过[监听聊天室事件]收取相关通知。

## 局限

- 默认同一用户不能同时加入多个聊天室。用户加入新的聊天室后会自动退出之前的聊天室。您可以在融云控制台 IM 服务的[服务配置](https://console.rongcloud.cn/agile/im/service/config#%E8%81%8A%E5%A4%A9%E5%AE%A4)页面，开启**单个用户加入多个聊天室**。
- 客户端的加入聊天室方法允许在加入时获取最新的历史消息（默认 10 条，最多 50 条），但不支持指定消息类型。您可以在融云控制台 IM 服务的[服务配置](https://console.rongcloud.cn/agile/im/service/config#%E8%81%8A%E5%A4%A9%E5%AE%A4)页面，开启**加入聊天室获取指定消息配置**，限制加入聊天室时只获取指定类型的消息。

## 加入已存在的聊天室

:::tip

 IMLib 从 5.6.3 开始新增重载方法，支持在回调中返回 [JoinChatRoomResponse]，其中包含聊天室的创建时间、成员数量、聊天室禁言状态、用户禁言状态等信息。
:::

如果 IMLib 版本 ≧ 5.6.3，可使用 [RongChatRoomClient] 的 [joinExistChatRoom] 方法加入一个已存在的聊天室。

#### 接口

```java
RongChatRoomClient.getInstance().joinExistChatRoom(chatRoomId, defMessageCount, callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | `String` | 聊天室会话 ID，最大长度为 64 个字符。 |
| `defMessageCount` | `Int` | 进入聊天室时获取历史消息的数量，数量范围：1-50。如果传 `-1`，表示不获取任何历史消息。如果传 `0`，表示使用 SDK 默认设置（默认为获取 10 条）。 |
| `callback` | `ResultCallback<JoinChatRoomResponse>` | 回调接口。 |


#### 示例代码

```java
String chatroomId = "聊天室 ID";
int defMessageCount = 50;

RongChatRoomClient.getInstance().joinExistChatRoom(chatRoomId, defMessageCount, new IRongCoreCallback.ResultCallback<JoinChatRoomResponse>() {
    @Override
    public void onSuccess(JoinChatRoomResponse response) {
        // 处理成功加入聊天室的情况
        System.out.println("成功加入聊天室");

        // 解析每个返回值
        long createTime = response.getCreateTime(); // 聊天室创建时间（毫秒时间戳）
        int memberCount = response.getMemberCount(); // 成员数量
        boolean isAllChatRoomBanned = response.isAllChatRoomBanned(); // 是否全局禁言
        boolean isCurrentUserBanned = response.isCurrentUserBanned(); // 当前用户是否被禁言
        boolean isCurrentChatRoomBanned = response.isCurrentChatRoomBanned(); // 当前用户是否在此聊天室被禁言
        boolean isCurrentChatRoomInWhitelist = response.isCurrentChatRoomInWhitelist(); // 当前用户是否在此聊天室的白名单中
        long joinTime = response.getJoinTime(); // 用户加入聊天室时间（毫秒时间戳）

        // 可以根据需要对解析的值进行进一步处理
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 处理加入聊天室错误的情况
        System.out.println("加入聊天室失败。错误代码: " + e.getValue());
        // 可以根据错误代码进行适当的处理和向用户提供反馈
    }
});
```

如果 IMLib 版本 \< 5.6.3，[joinExistChatRoom] 方法的成功回调中不携带额外信息。该方法在 5.6.3 版本上已废弃。

#### 示例代码

```java
String chatroomId = "聊天室 ID";
int defMessageCount = 50;

RongChatRoomClient.getInstance().joinExistChatRoom(chatroomId, defMessageCount, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

     @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode)    {

    }
});
```


## 加入聊天室

:::tip

 IMLib 从 5.6.3 开始废弃该方法。
:::

#### 接口

```java
RongChatRoomClient.getInstance().joinChatRoom(chatroomId, defMessageCount, callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | `String` | 聊天室会话 ID，最大长度为 64 个字符。 |
| `defMessageCount` | `Int` | 进入聊天室时获取历史消息的数量，数量范围：1-50。如果传 `-1`，表示不获取任何历史消息。如果传 `0`，表示使用 SDK 默认设置（默认为获取 10 条）。 |
| `callback` | `OperationCallback` | 回调接口 |


#### 示例代码

`joinChatRoom` 接口会创建并加入聊天室。如果聊天室已存在，则直接加入。如果您使用了服务端回调[聊天室状态同步]，融云会将聊天室创建成功的通知发送到您指定的服务器地址。

```java
String chatroomId = "聊天室 ID";
int defMessageCount = 50;

RongChatRoomClient.getInstance().joinChatRoom(chatroomId, defMessageCount, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

     @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

                                                                                                                     

## 断线重连后重新加入聊天室

SDK 具备断线重连机制。重连成功后，如果当前登录用户曾经加入过聊天室，且没有退出，则 SDK 会自动重新加入聊天室，您不需要应用处理。应用可以通过[监听聊天室状态]收到通知。

:::tip

 断网重连的场景下，一旦 SDK 重新加入聊天室成功，会自动收取一定数量的聊天室消息。
 - 如果 SDK 版本 \< 5.3.1，SDK 不会拉取消息。
 - 如果 SDK 版本 ≥ 5.3.1，SDK 会按照加入聊天室时传入的 `defMessageCount` 拉取固定数量的消息。拉取的消息有可能在本地已存在，应用可能需要进行排重后再显示。
:::


<!-- links -->
[聊天室服务配置]: ./service-config.md
[监听聊天室状态]: ./monitor-events.md#监听聊天室状态
[聊天室状态同步]: /platform-chat-api/chatroom/status
[监听聊天室事件]: ./monitor-events.md
[创建聊天室]: /platform-chat-api/chatroom/create
[RongChatRoomClient]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/index.html
[joinExistChatRoom]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/join-exist-chat-room.html
[JoinChatRoomResponse]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.model/-join-chat-room-response/index.html

