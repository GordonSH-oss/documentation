---
title: 查询聊天室房间信息
---

# 查询聊天室房间信息


您可以使用 `RongChatRoomClient` 下的 [getChatRoomInfo] 方法获取聊天室的信息，可返回以下数据：

- 聊天室成员总数
- 指定数量（最多 20 个）的聊天室成员的列表，包括该成员的用户 ID 以及加入聊天室的时间

:::tip

 **频率限制**：单个设备每秒钟支持调用一次，每分钟单个设备最多调用 20 次。
:::

#### 接口

```java
RongChatRoomClient.getInstance().getChatRoomInfo(chatroomId, defMemberCount, ChatRoomMemberOrder.RC_CHAT_ROOM_MEMBER_ASC, callback);
```

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `chatRoomId` | String | 聊天室 ID |
| `defMemberCount` | Int | 需要获取的聊天室成员数量。范围 0-20。因为聊天室一般成员数量巨大，权衡效率和用户体验，获取的聊天室成员数上限为 20。如果 `count` 为 0，则返回的聊天室信息仅包含成员总数，不包含具体的成员列表。|
| `order` | [ChatRoomMemberOrder] | 按照何种顺序返回聊天室成员信息。`RC_CHAT_ROOM_MEMBER_ASC`（升序），表示从最早加入聊天室成员开始，按加入时间递增的顺序获取，返回最早加入的成员列表。`RC_CHAT_ROOM_MEMBER_DESC`（降序），表示从最晚加入聊天室成员开始，按加入时间递减的顺序获取，返回最晚加入的成员列表。|
| `callback` | ResultCallback\<ChatRoomInfo\>  | 回调接口。在成功回调中返回 [ChatRoomInfo]，其中包含按要求获取的聊天室成员列表结果。列表元素为聊天室成员对象 [ChatRoomMemberInfo]，内部包含用户 ID（`userId`）和 Unix 时间戳格式的加入时间（`joinTime`），单位为毫秒。列表中的成员按加入时间从旧到新排列。 |


#### 示例代码

```java
String chatroomId = "Chatroom Target ID";
int defMemberCount = 10;

RongChatRoomClient.getInstance().getChatRoomInfo(chatroomId, defMemberCount, ChatRoomMemberOrder.RC_CHAT_ROOM_MEMBER_ASC, new IRongCoreCallback.ResultCallback<ChatRoomInfo>() {

    @Override
    public void onSuccess(ChatRoomInfo chatRoomInfo) {
        // Get ChatRoomInfo properties

        String chatRoomId = chatRoomInfo.getChatRoomId();
        int totalMemberCount = chatRoomInfo.getTotalMemberCount();

        // Get ChatRoomMemberInfo properties
        List<ChatRoomMemberInfo> memberInfoList = chatRoomInfo.getMemberInfo();
        if (memberInfoList != null) {
            for (ChatRoomMemberInfo memberInfo : memberInfoList) {
                String MemberId = memberInfo.getUserId();
                long JoinTime = memberInfo.getJoinTime();
            }
        }
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // Handle error
    }
});
```



[ChatRoomInfo]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.model/-chat-room-info/index.html
[ChatRoomMemberOrder]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.model/-chat-room-info/-chat-room-member-order/index.html
[ChatRoomMemberInfo]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.model/-chat-room-member-info/index.html
[getChatRoomInfo]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/get-chat-room-info.html

