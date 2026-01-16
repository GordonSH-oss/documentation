# 快速集成直播聊天室

本教程主要描述如何使用融云 IM SDK 在 Android 端（Java）快速实现一个直播聊天室。

## 前置条件

创建融云开发者账号，获取 [App Key]。

## 步骤 1：导入 SDK

利用 Android Studio 中的 Gradle 构建系统，您可以将融云即时通讯能力库（IMLib）作为远程依赖项或本地 Android 库模块（Module）添加到您的构建中。

本教程以在 Gradle 中添加远程依赖项为例。请注意使用 [融云的 Maven 仓库](http://maven.rongcloud.cn)。

1. 打开根目录下的 `build.gradle`（**Project** 视图下），声明融云的 Maven 代码库。

    ```groovy
    allprojects {
        repositories {
            ...
            //融云 maven 仓库地址
            maven {url "https://maven.rongcloud.cn/repository/maven-releases/"}
        }
    }
    ```

2. 在应用的 `build.gradle` 中，添加融云即时通讯能力库（IMLib）为远程依赖项。

    ```groovy
    dependencies {
    	...
    	api 'cn.rongcloud.sdk:im_libcore:5.4.1'
    	api 'cn.rongcloud.sdk:im_chatroom:5.4.1'
    }
    ```

## 步骤 2：初始化 SDK

在 Application 的 `onCreate()` 方法中初始化 SDK，传入 App Key 和初始化配置（`InitOption`）。如果 App Key 不属于中国（北京）数据中心，必须在初始化配置中传入指定的导航服务器和统计服务器地址。

```java
String appKey = "Your_AppKey";

InitOption initOption = new InitOption.Builder()
    .setNaviServer("http(s)://naviServer") // 如果 App Key 属于新加坡或北美数据中心，必须配置为对应导航服务器地址
    .setStatisticServer("http(s)://StatisticServer") // 如果 App Key 属于新加坡或北美数据中心，必须配置为对应的统计服务器地址
    .build();

RongCoreClient.init(context, appKey, initOption);
```

- 新加坡数据中心 Navi Server 地址：nav.sg-light-edge.com（主）、nav-b.sg-light-edge.com（备）
- 新加坡数据中心 StatisticServer 地址： stats.sg-light-edge.com

## 步骤 3：添加消息监听器

应用需要通过 SDK 提供的消息监听器接收消息与通知。当前用户会通过该监听器接收所有类型的消息。建议在应用生命周期内注册消息监听。

```java
RongCoreClient.addOnReceiveMessageListener(
        new OnReceiveMessageWrapperListener() {
            @Override
            public boolean onReceivedMessage(Message message, int left, boolean hasPackage, boolean offline) {
            ///
            return false;
            }
});
```

## 步骤 4：建立 IM 连接

使用融云即时通讯功能前必须与融云服务器建立 IM 连接。建立 IM 连接时需要传入用户 Token。

建立 IM 时传入的用户 Token 表示用户在融云的唯一标识。您需要自行维护 App 用户注册流程，为用户分配唯一的用户标识（User ID），并使用该用户 ID 向融云申请建立 IM 连接所需使用的 Token。

### 步骤 4.1：获取 Token

客户端 SDK 不提供获取 Token 的 API。您需要通过融云服务端 API 获取 Token。

通过 App 层定义的用户 ID（`userId`）换取融云服务中使用的身份验证 Token。同一个用户 ID 可多次获取 Token，如果 Token 在有效期内，均可用于连接融云服务。同一用户 ID 如需重新获取 Token 使用同一接口。

调用融云服务端 API 必须进行身份验证。请在请求中添加以下 HTTP 标头（Header）字段：

- `App-Key`: App Key
- `Nonce`: 随机数
- `Timestamp`: Unix 时间戳
- `Signature`: 按以下顺序将 App Secret、Nonce、Timestamp 拼接成一个字符串，进行 SHA1 哈希计算。App Secret 与 App Key 对应，可从融云控制台 [App Key] 页面获取。

如果 App Key 不属于中国（北京）数据中心，必须请求海外的 API 服务地址。

- 北京：api.rong-api.com
- 新加坡：api.sg-light-api.com（主）、api-b.sg-light-api.com（备）

在获取 Token 时，HTTP 请求正文中需要传入用户 ID（`userId`）、名称（`name`）和头像（`portraitUri`）。

```http
POST /user/getToken.json HTTP/1.1
Host: api.rong-api.com
App-Key: Your_AppKey
Nonce: 14314
Timestamp: 1408710653491
Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
Content-Type: application/x-www-form-urlencoded

userId=jlk456j5&name=Ironman&portraitUri=http%3A%2F%2Fabc.com%2Fmyportrait.jpg
```

用户 ID 支持大小写英文字母与数字的组合，最大长度 64 字节。**注意**，名称（`name`）和头像（`portraitUri`）仅供移动端远程推送时使用。在重新获取 Token 时如果传入新的数据，则会覆盖旧的名称与头像数据。

返回结果中会包含传入的用户 ID 与对应的 Token。Token 长度在 256 字节以内，您可以缓存在应用内。

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{"code":200, "userId":"jlk456j5", "token":"sfd9823ihufi"}
```

### 步骤 4.2：连接聊天服务器

在初始化完成后可建立 IM 连接，传入上一步中获取的用户 Token。`timeLimit` 参数为超时秒数。

```java
RongCoreClient.connect(userToken, timeLimit, new IRongCoreCallback.ConnectCallback(){
    @Override
    public void onDatabaseOpened(IRongCoreEnum.DatabaseOpenStatus code) {
        if(IRongCoreEnum.DatabaseOpenStatus.DATABASE_OPEN_SUCCESS.equals(code)) {
                    //本地数据库打开，跳转到会话列表页面
        } else {
            //数据库打开失败，可以弹出 toast 提示。
        }
        }

    @Override
    public void onSuccess(String s) {
        //连接成功，如果 onDatabaseOpened() 时没有页面跳转，也可在此时进行跳转。
    }

    @Override
    public void onError(IRongCoreEnum.ConnectionErrorCode errorCode) {
        if(errorCode.equals(IRongCoreEnum.ConnectionErrorCode.RC_CONN_TOKEN_EXPIRE)) {
            //从 APP 服务请求新 token，获取到新 token 后重新 connect()
        } else if (errorCode.equals(RongCoreClient.ConnectionErrorCode.RC_CONNECT_TIMEOUT)) {
            //连接超时，弹出提示，可以引导用户等待网络正常的时候再次点击进行连接
        } else {
            //其它业务错误码，请根据相应的错误码作出对应处理。
        }
}
})
```

一旦连接成功，SDK 的重连机制开始生效。当因为网络原因断线时，SDK 会不停重连直到连接成功为止，不需要您做额外的连接操作。

## 步骤 5：加入聊天室

在融云聊天室业务中，您需要先创建一个聊天室，并将用户加入聊天室，用户才能在聊天室中收发消息。

通常情况下，您通过服务端 API 创建聊天室，而用户一般通过客户端 SDK 加入聊天室。为了保持该教程的简单性，我们将直接调用客户端的创建并加入聊天室的接口 `RongChatRoomClient#joinChatRoom`，一次完成创建与加入聊天室的动作。

```java
String chatroomId = "chatroomA";
int defMessageCount = -1;

RongChatRoomClient.getInstance().joinChatRoom(chatroomId, defMessageCount, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

     @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

`defMessageCount` 变量的值表示加入聊天室时需要获取的历史消息数量，`-1` 表示加入聊天室不获取任何历史消息。

如果加入已存在的聊天室，可以调用 `RongChatRoomClient#joinExistChatRoom` 方法。

## 步骤 6：发送消息

发送消息需要使用核心类 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 请注意，客户端 SDK 发送消息存在频率限制，每秒最多只能发送 5 条消息。

### 步骤 6.1：构造消息

所有消息可分为两大类：普通消息和多媒体消息。普通消息父类是 [MessageContent]，媒体消息父类是 [MediaMessageContent]。发送媒体消息和普通消息本质的区别为是否有上传数据过程。

普通消息一般指文本消息。以下示例构造了一条文本消息。

```java
String targetId = "聊天室 ID";
ConversationType conversationType = Conversation.ConversationType.CHATROOM;
TextMessage messageContent = TextMessage.obtain("Hello");
Message message = Message.obtain(targetId, conversationType, messageContent);
```

媒体消息一般包含图片消息、GIF 等。以下示例构造了一条图片消息。

```java
String targetId = "聊天室 ID";
ConversationType conversationType = Conversation.ConversationType.CHATROOM;
Uri localUri = Uri.parse("file://图片的路径");//图片本地路径，接收方可以通过 getThumUri 获取自动生成的缩略图 Uri
boolean mIsFull = true; //是否发送原图
ImageMessage mediaMessageContent = ImageMessage.obtain(localUri, mIsFull);
Message message = Message.obtain(targetId, conversationType, channelId, mediaMessageContent);
```

### 步骤 6.2：发送消息

发送不涉及媒体文件上传的消息，例如融云内置消息类型中的文本消息等，或自定义的、不涉及媒体文件上传流程的自定义消息，可使用 `sendMessage` 方法。

```java
RongCoreClient.getInstance().sendMessage(message, null, null, new IRongCoreCallback.ISendMessageCallback() {

        @Override
        public void onAttached(Message message) {

        }

        @Override
        public void onSuccess(Message message) {

        }

        @Override
        public void onError(Message message, IRongCoreEnum.CoreErrorCode errorCode) {

        }
    });
```

发送媒体消息需要使用 `sendMediaMessage` 方法。该方法会先将图片、视频等媒体文件上传到融云默认的文件服务器（[文件存储时长](https://help.rongcloud.cn/t/topic/1049)），上传成功之后再发送消息。

```java
RongCoreClient.getInstance().sendMediaMessage(message, null, null, new IRongCoreCallback.ISendMediaMessageCallback() {
    @Override
    public void onProgress(Message message, int i) {

    }

    @Override
    public void onCanceled(Message message) {

    }

    @Override
    public void onAttached(Message message) {

    }

    @Override
    public void onSuccess(Message message) {

    }

    @Override
    public void onError(final Message message, final IRongCoreEnum.CoreErrorCode errorCode) {

    }
});

```

## 步骤 7：保存历史消息

当前用户的聊天室本地消息在退出聊天室时会被自动删除。在退出聊天室前，可以获取本地的历史消息记录。开通**聊天室消息云端存储**服务后，可将聊天室消息存储在融云服务端。客户端可以直接获取存储在服务端的消息。

一般先调用 `getHistoryMessages` 获取本地的历史记录，如果为空的话，再调用 `getChatroomHistoryMessages` 来获取远端历史记录。注意，`getChatroomHistoryMessages` 不会返回本地数据库中已存在的消息。

```java
Conversation.ConversationType conversationType = Conversation.ConversationType.CHATROOM;
int oldestMessageId = -1;
final int count = 10;

final String targetId = "聊天室 ID";
final long recordTime = 0;

RongCoreClient.getInstance().getHistoryMessages(conversationType, targetId, oldestMessageId, count,
                new IRongCoreCallback.ResultCallback<List<Message>>() {

                    @Override
                    public void onSuccess(List<Message> messages) {
                        if (messages == null || messages.isEmpty()) {
                           RongChatRoomClient.getInstance().getChatroomHistoryMessages(targetId, recordTime, count, IRongCoreEnum.TimestampOrder.RC_TIMESTAMP_ASC,
                                new IRongCoreCallback.IChatRoomHistoryMessageCallback() {

                                    @Override
                                    public void onSuccess(List<Message> messages, long syncTime) {

                                    }


                                    @Override
                                    public void onError(IRongCoreEnum.CoreErrorCode code) {

                                    }
                                });

                        }
                    }

                    @Override
                    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

                    }
                });

```

`oldestMessageId` 为 `-1` 表示从本地最新消息开始查询。

`getChatroomHistoryMessages` 在获取成功的回调中会返回获取到的历史消息数组和 `syncTime`。如果拉取顺序为 `RC_Timestamp_Desc`，为拉取结果中最早一条消息的时间戳（即最小的时间戳）。如果拉取顺序为 `RC_Timestamp_Asc`，为拉取结果中最晚消息的时间戳（即最大的时间戳）。在拉取顺序不变的情况下，当次返回的 `syncTime` 的值可以作为下次拉取时的 `recordTime` 传入，方便连续拉取。

## 步骤 8：撤回消息

如果 App 的管理员或者某普通用户希望在所有聊天室成员的聊天记录中删除一条消息，可使用撤回消息功能。消息成功撤回后，原始消息内容会在所有用户的本地与服务端历史消息记录中删除。


```java
Message message = Message.obtain("123", ConversationType.GROUP, "12345");

RongCoreClient.getInstance().recallMessage(message, "", new IRongCoreCallback.ResultCallback<RecallNotificationMessage>() {
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
    public void onError(IRongCoreEnum.CoreErrorCode errorCode) {

    }
});
```

## 步骤 9：封禁用户

即时通讯业务支持多种封禁与禁言能力。客户端不提供相关的 API，您可以通过服务端 API 实现。

### 封禁指定用户

对 App 用户进行封禁处理，单次请求最多封禁 20 个用户 ID（`userId`）。`minute` 参数代表封禁时长，最长封禁 43200 分钟。一旦封禁，被封禁用户立即断开 IM 连接。被封禁期间用户无法与融云建立 IM 连接，无法主动使用 IM 服务。

- 封禁：

    ```http
    POST /user/block.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    userId=UserA&minute=10
    ```

- 解除封禁：

    ```java
    POST /user/unblock.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    userId=UserA
    ```

### 禁言聊天室用户

聊天室业务支持禁言用户功能，单次可设置最多 20 个用户（`userId`）在指定聊天室（`chatroomId`）中禁言。`minute` 参数代表禁言时长，最长封禁 43200 分钟。被禁言用户可以接收查看聊天室中其他用户聊天信息，但不能通过客户端 SDK 往该聊天室内发送消息。用户退出聊天室不会使禁言状态失效。

- 禁言

    ```http
    POST /chatroom/user/gag/add.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    userId=UserA&chatroomId=16&minute=1
    ```

- 解除用户禁言：

    ```http
    POST /chatroom/user/gag/rollback.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    userId=UserA&chatroomId=16
    ```

### 禁言聊天室

聊天室业务支持将指定聊天室（`chatroomId`）禁言。禁言后，该聊天室中的所有用户均不能通过客户端 SDK 往该聊天室内发送消息。如需允许部分例外用户，可将用户加入聊天室禁言用户白名单（不在本教程范围内）。如果聊天室解散，全体禁言数据会被清除。

- 禁言聊天室

    ```http
    POST /chatroom/ban/add.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    chatroomId=123
    ```

- 取消聊天室成员全体禁言：

    ```http
    POST /chatroom/ban/rollback.json HTTP/1.1
    Host: api.rong-api.com
    App-Key: uwd1c0sxdlx2
    Nonce: 14314
    Timestamp: 1408710653491
    Signature: 45beb7cc7307889a8e711219a47b7cf6a5b000e8
    Content-Type: application/x-www-form-urlencoded

    chatroomId=123
    ```

聊天室业务还支持将指定用户在应用下的所有聊天室中禁言，支持设置禁言时长。在禁言时间段内，被禁言用户在应用下的所有聊天室中都无法通过客户端 SDK 发送消息。该功能要求开通**聊天室全局禁言**服务。本教程暂不做介绍。

## 附录：如何实现聊天室点赞

您可以用自定义消息实现聊天室内点赞功能。

1. 创建一个继承 `MessageContent` 的自定义消息类型，例如 XappChatroomLike。点赞消息中将包含该消息内容。

    ```java
    import android.os.Parcel;
    import io.rong.imlib.MessageTag;
    import io.rong.imlib.model.MessageContent;

    @MessageTag(value = "Xapp:Chatroom:Like")
    public class XappChatroomLike extends MessageContent {

        @Override
        public int describeContents() {
            return 0;
        }

        @Override
        public void writeToParcel(Parcel dest, int flags) {
        }

        public void readFromParcel(Parcel source) {
        }

        public XappChatroomLike() {
        }

        public XappChatroomLike(byte[] data){

        }

        @Override
        public byte[] encode() {
            return new byte[0];
        }

        protected XappChatroomLike(Parcel in) {
        }

        public static final Creator<XappChatroomLike> CREATOR = new Creator<XappChatroomLike>() {
            @Override
            public XappChatroomLike createFromParcel(Parcel source) {
                return new XappChatroomLike(source);
            }

            @Override
            public XappChatroomLike[] newArray(int size) {
                return new XappChatroomLike[size];
            }
        };
    }
    ```

1. 在调用 `RongCoreClient#connect` 方法前，向 SDK 注册 `XappChatroomLike`。

    ```java
    ArrayList<Class<? extends MessageContent>> myMessageContents = new ArrayList<>();
    myMessageContents.add(XappChatroomLike.class);
    RongCoreClient.registerMessageType(myMessageContents);
    ```

1. 在需要点赞时，构造点赞消息，使用 `sendMessage` 方法发送。

    ```java
    String targetId = "聊天室 ID";
    ConversationType conversationType = Conversation.ConversationType.CHATROOM;
    XappChatroomLike messageContent = XappChatroomLike();
    Message message = Message.obtain(targetId, conversationType, messageContent);
    ```

1. 聊天室内其他用户接收消息时，判断是否为 `XappChatroomLike`。如果接到点赞消息，可以在 UI 上展示点赞效果。

    ```java
    RongCoreClient.addOnReceiveMessageListener(
            new OnReceiveMessageWrapperListener() {
                @Override
                public boolean onReceivedMessage(Message message, int left, boolean hasPackage, boolean offline) {
                MessageContent msgContent = message.getContent();
                if (msgContent instanceof XappChatroomLike) {

                }
                return false;
                }
    });
    ```

<!-- links -->
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[MediaMessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/MediaMessageContent.html
[融云的 Maven 代码库]: http://maven.rongcloud.cn/#browse/browse:maven-releases
[App Key]: https://console.rongcloud.cn/agile/apps/km
