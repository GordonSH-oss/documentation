---
title: 绑定音视频房间
---

# 绑定音视频房间

:::tip
客户端 SDK 5.2.1 版本开始支持绑定音视频房间接口。
:::

**适用场景**：
聊天室具有自动销毁机制。在使用融云 RTC 业务的 App 中，可能配合使用 IMLib SDK 的聊天室业务实现直播聊天、弹幕、属性记录等功能。这种情况下，可以考虑将聊天室与音视频房间绑定，确保聊天室不会在语聊、直播结束前销毁，以免丢失关键数据。（在单独使用聊天室业务情况的下，无需调用该接口。）

聊天室绑定音视频房间后，当聊天室达到预设的自动销毁条件时，服务端会先检测已绑定的音视频房间（`RTCRoomId`）是否仍存在：

- 如果绑定的音视频房间仍存在，则聊天室不会销毁。
- 如果绑定的音视频房间已销毁，则直接销毁聊天室。

该接口仅创建从聊天室到音视频房间的**单向绑定关系**。因此在绑定音视频房间后，音视频房间的主动销毁或自动销毁，并不会直接触发聊天室房间的销毁。关于音视频房间的销毁机制，详见[音视频房间销毁机制](/server-rtc-api/room/about-auto-destroy)。


:::tip

 **关于聊天室自动销毁逻辑的说明**：

 聊天室具有自动销毁机制，默认情况下所有聊天室会在不活跃（连续时间段内无成员进出且无新消息）达到 1 小时后踢出所有成员并自动销毁，可延长该时间，也可配置为定时自动销毁。详见服务端文档[聊天室销毁机制](/platform-chat-api/chatroom/destroy-about)。
:::


调用该接口必须传入聊天室 ID，因此必须在聊天室房间已创建成功之后调用。客户端调用加入聊天室接口时会自动完成创建与加入动作。

该接口不具备用户权限控制功能，建议由业务侧的房主或者主播角色用户加入聊天室房间成功后调用一次，其他用户无需调用。

#### 接口说明

```java
public abstract void bindChatRoomWithRTCRoom(
            String chatRoomId, String rtcRoomId, IRongCoreCallback.OperationCallback callback)
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | String | 聊天室 ID，非空。聊天室 ID 必须已存在。需要 App 传入。 |
| `rtcRoomId` | String | 音视频房间 ID，非空。需要 App 传入。 |
| `callback` | IRongCoreCallback.OperationCallback | 事件监听回调 |

#### 示例代码

```java
String chatRoomId = "您的聊天室 ID";
String RTCRoomId = "您的 RTC 房间 ID";
RongChatRoomClient.getInstance().bindChatRoomWithRTCRoom(chatRoomId, rtcRoomId, new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {

        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

        }
});
```
