---
title: IMLib 2.X 升级到 5.X
sidebar_position: 1
---

# IMLib 2.X 升级到 5.X

本文描述 IMLib SDK 从 2.X 到 5.X 版本的升级步骤。

5.x 相较于 2.x 版本，主要有以下变更：

* IMLib 拆分
* 连接接口变更
* 删除了部分废弃接口

## IMLib 拆分说明

5.x 版本对 IMLib SDK 进行了拆分，拆分成如下六个模块：

| 模块名称      | 功能说明     |
| ------------- | ------------ |
| LibCore       | 核心通讯模块 |
| chatroom      | 聊天室       |
| customservice | 客服         |
| disscussion   | 讨论组       |
| location      | 实时位置     |
| publicservice | 公众号       |

5.x 版本开始会提供两种形式的 SDK：完整包和拆分包

* 如果使用完整包的话，直接正常升级就行。
* 如果使用拆分包，首先 **必需依赖 LibCore 库**，然后按需集成即可。比如您除了核心模块，还使用到聊天室功能，那让项目同时依赖 LibCore 模块和 chatroom 模块，依次类推使用到哪个模块就需要依赖相关模块。

## 连接接口变更

### 连接接口 1

```java
connect(final String token, final RongIMClient.ConnectCallback connectCallback)
```

用户调用接口之后，如果因为网络原因暂时连接不上，SDK 会一直尝试重连，直到连接成功或者出现 SDK 无法处理的错误（如 token 无效，用户封禁等）。

```java
RongIMClient.connect(token, new RongIMClient.ConnectCallback() {
    @Override
    public void onDatabaseOpened(RongIMClient.DatabaseOpenStatus code) {
        //消息数据库打开，可以进入到主页面
    }

    @Override
    public void onSuccess(String s) {
        //连接成功
    }

    @Override
    public void onError(RongIMClient.ConnectionErrorCode errorCode) {
        if(errorCode.equals(RongIMClient.ConnectionErrorCode.RC_CONN_TOKEN_INCORRECT)) {
            //从 APP 服务获取新 token，并重连
        }else {
            //无法连接到 IM 服务器，请根据相应的错误码作出对应处理
        }
    }
})
```

### 连接接口 2

```java
connect(final String token, final int timeLimit, final ConnectCallback connectCallback)
```

用户调用接口之后，SDK 会在 timeLimit 秒内尝试连接，超过时间将会返回超时并停止连接，timeLimit 小于等于 0 行为和没有 timeLimit 的接口一样。

```java
RongIMClient.connect(token,5, new RongIMClient.ConnectCallback() {
    @Override
    public void onDatabaseOpened(RongIMClient.DatabaseOpenStatus code) {
        //消息数据库打开，可以进入到主页面
    }

    @Override
    public void onSuccess(String s) {
        //连接成功
    }

    @Override
    public void onError(RongIMClient.ConnectionErrorCode errorCode) {
        if(errorCode.equals(RongIMClient.ConnectionErrorCode.RC_CONN_TOKEN_INCORRECT)) {
            //从 APP 服务获取新 token，并重连
        } else if (errorCode.equals(RongIMClient.ConnectionErrorCode.RC_CONNECT_TIMEOUT)) {
            //连接超时，弹出提示，可以引导用户等待网络正常的时候再次点击进行连接
        } else {
            //无法连接 IM 服务器，请根据相应的错误码作出对应处理
        }
    }
})
```

### 错误回调变更

onError 回调参数以及回调机制变化如下：

| 版本           | 参数类型                         | 回调时机                         | 是否继续重连                                  |
| -------------- | -------------------------------- | -------------------------------- | --------------------------------------------- |
| 2.x 版本       | RongIMClient.ErrorCode           | 连接过程中发生的任何异常都会回调 | 业务错误时不再重连，其它情况 SDK 会继续重连。 |
| 4.x & 5.x 版本 | RongIMClient.ConnectionErrorCode | 连接过程出现业务错误时回调       | 否                                            |

### 连接错误信息说明

ConnectionErrorCode 说明如下：

```
31004：Token 无效。
处理方案：从 APP 服务器获取新的 token ，再调用 connect 接口进行连接。

31010： 用户被踢下线。
处理法案：退回到登录页面，给用户提示被踢掉线。

31023：用户在其它设备上登录。
处理方案：退回到登录页面，给用户提示其他设备登录了当前账号。

31009：用户被封禁。
处理方案：退回到登录页面，给用户提示被封禁。

34006：自动重连超时(发生在 timeLimit 为有效值并且网络极差的情况下)。
处理方案：重新调用 connect 接口进行连接。

31008：Appkey 被封禁。
处理方案：请检查您使用的 AppKey 是否被封禁或已删除。

33001：SDK 没有初始化。
处理方案：在使用 SDK 任何功能之前，必须先 Init。

33003：开发者接口调用时传入的参数错误。
处理方案：请检查接口调用时传入的参数类型和值。

33002：数据库错误。
处理方案：检查用户 userId 是否包含特殊字符，SDK userId支持大小写英文字母与数字的组合，最大长度 64 字节。
```

### 5. 连接错误解决方案：

1. connect() 的 onError 回调中判断是否是 token 无效和连接超时。

2. 连接状态监听中判断连接状态是否是 token 无效、踢掉线、封禁、自动重连超时等业务错误，并按照上述处理方案处理。

```java
//连接状态监听设置
//其中 this 最好为单例类，以此保证在整个 APP 生命周期，该类都能够检测到 SDK 连接状态的变更
RongIMClient.setConnectionStatusListener(this);

public void onChanged(ConnectionStatus connectionStatus) {
    if (connectionStatus.equals(ConnectionStatus.KICKED_OFFLINE_BY_OTHER_CLIENT)) {
        //当前用户账号在其他端登录，请提示用户并做出对应处理
    } else if (connectionStatus.equals(ConnectionStatus.CONN_USER_BLOCKED)) {
        //用户被封禁，请提示用户并做出对应处理
    }
}
```

## 移除接口说明

4.0.0 版本开始移除了以下废弃方法：

| 接口名称                                                     | 替代方法                                                     |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| `setUserPolicy()`                                              | 无                                                           |
| `List<> getConversationList()`                                 | `getConversationList(ResultCallback)`                          |
| `List<> getConversationList(Conversation.ConversationType...)` | `getConversationList(ResultCallback)`                          |
| `Conversation getConversation(Conversation.ConversationType , String )` | `getConversation(final Conversation.ConversationType ,  String ,  ResultCallback<> callback)` |
| `boolean removeConversation()`                                 | `removeConversation(final Conversation.ConversationType, String, ResultCallback<> callback)` |
| `boolean setConversationToTop()`                               | `setConversationToTop(final Conversation.ConversationType, String, boolean, ResultCallback<> callback)` |
| `int getTotalUnreadCount()`                                    | `getTotalUnreadCount( ResultCallback<> callback)`              |
| `int getTotalUnreadCount(ConversationType type)`               | `getUnreadCount(Conversation.ConversationType, String, ResultCallback)` |
| `int getUnreadCount(Conversation.ConversationType…)`           | `getUnreadCount(final Conversation.ConversationType[],  ResultCallback<> callback)` |
| `List<> getLatestMessages(Conversation.ConversationType, String, int)` | `getLatestMessages(Conversation.ConversationType, String, int, ResultCallback)` |
| `List<> getHistoryMessagesByMessageId(Conversation.ConversationType, String, int, int)` | `getHistoryMessages(Conversation.ConversationType, String, int, int, ResultCallback)` |
| `List<Message> getHistoryMessagesByObjectNames( Conversation.ConversationType, String, List<>,  long,  int, RongCommonDefine.GetMessageDirection)` | `getHistoryMessagesByObjectNamesSync(final Conversation.ConversationType conversationType, final String targetId,List<String> objectNames, final long timestamp, final int count, final RongCommonDefine.GetMessageDirection direction)` |
| `getUserOnlineStatus()`                                        | 无                                                           |
| `setSubscribeStatusListener()`                                 | 无                                                           |
| `setUserOnlineStatus()`                                        | 无                                                           |
| `getHistoryMessagesOneWay()`                                   | 无                                                           |
| `boolean deleteMessages(int[])`                                | `deleteMessages(int[], ResultCallback)`                        |
| `boolean clearMessages(Conversation.ConversationType , String)` | `clearMessages(Conversation.ConversationType, String, ResultCallback)` |
| `boolean clearMessagesUnreadStatus(Conversation.ConversationType, String)` | `clearMessagesUnreadStatus(Conversation.ConversationType, String, ResultCallback)` |
| `boolean setMessageExtra(int messageId, String value)`         | `setMessageExtra(int, String, ResultCallback)`                 |
| `boolean setMessageSentStatus(int, Message.SentStatus)`        | `setMessageReceivedStatus(int, Message.ReceivedStatus, ResultCallback)` |
| `void setMessageSentStatus(final int, Message.SentStatus,  ResultCallback<>)` | `setMessageSentStatus(Message, ResultCallback)`                |
| `String getTextMessageDraft()`                                 | `getTextMessageDraft(Conversation.ConversationType, String, ResultCallback)` |
| `boolean saveTextMessageDraft(Conversation.ConversationType, String, String)` | `saveTextMessageDraft(Conversation.ConversationType, String, String, ResultCallback)` |
| `boolean clearTextMessageDraft(Conversation.ConversationType, String)` | `clearTextMessageDraft(Conversation.ConversationType, String, ResultCallback)` |
| `void insertMessage(final Conversation.ConversationType,  String,  String,  MessageContent,  long ,  ResultCallback<>)` | `insertIncomingMessage(Conversation.ConversationType, String, String, Message.ReceivedStatus, MessageContent, long, ResultCallback)` <br />`insertOutgoingMessage(Conversation.ConversationType, String, Message.SentStatus, MessageContent, long, ResultCallback)` |
| `void insertMessage(final Conversation.ConversationType,  String,  String,  MessageContent, ResultCallback<>)` | 同上                                                         |
| `Message sendMessage( Conversation.ConversationType, String, MessageContent,  String,  String, SendMessageCallback)` | `sendMessage(Conversation.ConversationType, String, MessageContent, String, String, IRongCallback.ISendMessageCallback)` |
| `void sendMessage( Conversation.ConversationType , String, MessageContent, String,  String,  SendMessageCallback,  ResultCallback<Message> resultCallback)` | `sendMessage(Message, String, String, IRongCallback.ISendMessageCallback)` |
| `Message sendMessage( Message, pushContent, String,  SendMessageCallback)` | `sendMessage(Message, String, String, IRongCallback.ISendMessageCallback)` |
| `syncGroup()`                                                  | 无                                                           |
| `joinGroup()`                                                  | 无                                                           |
| `quitGroup()`                                                  | 无                                                           |
| `clearConversations()`                                         | `clearConversations(ResultCallback, Conversation.ConversationType...)` |
| `syncUserData()`                                               | 无                                                           |
| `updateRealTimeLocationStatus()`                               | 无                                                           |
| `RecallMessageListener()`                                      | 无                                                           |
| `setRecallMessageListener()`                                   | 无                                                           |
| `setPushNotificationListener()`                                | 无                                                           |
| `syncConversationNotificationStatus()`                         | 无                                                           |


## 旧版本快速兼容方案

说明：以下是旧版本快速兼容方案，但是我们依然建议您参考上面的详细建议进行处理；如果您是直接使用 4.0.0 新版本建议参见上面的详细处理流程。

1. 将 connect() 的回调更改为新的 callback。
2. 在 connect() 的 onError() 回调中，判断 ConnectionErrorCode 为 RC_CONN_TOKEN_INCORRECT()时，将 原 onTokenIncorrect() 中的处理逻辑拷贝过来。

```java
RongIMClient.connect(token, new RongIMClient.ConnectCallback() {
        @Override
        public void onDatabaseOpened(RongIMClient.DatabaseOpenStatus code) {
            //如果消息数据库打开，可以进入到主页面
        }

        @Override
        public void onSuccess(String s) {
            //连接成功
        }

        @Override
        public void onError(RongIMClient.ConnectionErrorCode e) {
            if(e.equals(RongIMClient.ConnectionErrorCode.RC_CONN_TOKEN_INCORRECT)) {
                //将旧版本 token 无效的回调处理代码写到这里
                //从 APP 服务获取新 token，并重连
            }else {
                //无法连接 im 服务器，请根据相应的错误码作出对应处理
            }
        }
});
```

## 常见问题

### 1.为什么一部分无法重连错误码的处理逻辑并没有在示例代码中写明？

都是开发阶段的问题，不需要代码兼容处理。

`31008`：Appkey 被封禁

当发生这个问题的时候，您可以自行在控制台查看您的 appkey 使用状态，大多情况是 appkey 被自行删除或者欠费。

`33001`：SDK 没有初始化

这个错误只会发生在开发阶段，只要您保证先 init 后 connect 就不会有这个问题。

`33003`：开发者接口调用时传入的参数错误

这个错误只会发生在开发阶段，很可能是传入的 token 为空，只要保证 connect 传入正确合法的 token 就不会有这个问题。

`33002`：数据库错误

这个问题只会发生在开发阶段，很可能是您的用户 id 体系和我们 SDK 的不一致，一般该情况很少发生。

### 2.旧版本连接过程中一旦出现中间错误码就会立即触发 error 回调，新版本中间错误码不会触发 error 回调了，可能会等很长时间没有任何回调，这要怎么处理？

以下的建议，择一选取即可。

建议1.

用户第一次登录，设置 timeLimit 为有效值，网络极差情况下超时回调 error。

用户后续登录，调用没有 timeLimit 的接口，SDK 就会保持旧版本的自动重连。

建议2.

设置 SDK 的连接状态监听，APP 自行做超时的记录，如果超时了，APP 可以自动断开连接再调用 connect 进行连接。

