---
title: 删除消息
sidebar_position: 60
---

# 删除消息

针对单聊会话、群聊会话、系统会话，您可以通过 IMLib SDK 删除自己的历史消息，支持仅从本地数据库删除消息、仅从融云服务端删除自己的历史消息、或同时删除本地数据库以及融云服务端自己的历史消息。

IMLib SDK 的删除消息操作（下表中的 API）均指从当前登录操作的用户的历史消息记录中删除一条或一组消息，不影响会话中其他用户的历史消息记录。

| 功能                                 | 本地/服务端                                   | API                              |
|:-----------------------------------|:-----------------------------------------|:---------------------------------|
| [仅从本地删除指定消息（消息 ID）]      | 仅从本地删除（重载方法）                        | [deleteMessages][deleteMessages] |
| [仅从本地删除会话全部历史消息]       | 仅从本地删除（重载方法）                        | [deleteMessages][deleteMessages] |
| [删除会话内指定消息（消息对象）]       | 同时从本地和服务端删除消息                    | [deleteRemoteMessages]           |
| [删除会话历史消息（时间戳）]           | 可选仅本地删除、或者同时从本地和服务端删除消息 | [cleanHistoryMessages]           |
| [仅从服务端删除会话历史消息（时间戳）] | 仅从服务端删除                                | [cleanRemoteHistoryMessages]     |

:::tip

 - App 用户的单聊会话、群聊会话、系统会话的消息默认仅存储在本地数据库中，仅支持从本地删除。<!--public-cloud-only start-->如果 App（App Key/环境）已开通[单群聊消息云端存储](https://console.rongcloud.cn/agile/im/service/purchase#4)，<!--public-cloud-only end-->该用户的消息还会保存在融云服务端（默认 6 个月），可从远端历史消息记录中删除消息。
 - 针对单聊会话、群聊会话，如果通过任何接口以传入时间戳的方式删除远端消息，服务端默认不会删除对应的离线消息补偿（该机制仅会在打开[多设备消息同步](https://console.rongcloud.cn/agile/im/service/config#%E5%A4%9A%E7%AB%AF)开关后生效）。此时如果换设备登录或卸载重装，仍会因为[消息补偿机制](/guides/glossary/imglossary#compensation)获取到已被删除的历史消息。如需彻底删除消息补偿，请[提交工单]，申请开通**删除服务端历史消息时同时删除多端补偿的离线消息**。如果以传入消息对象的方式删除远端消息，则服务端一定会删除消息补偿中的对应消息。<!--public-cloud-only start-->
 - 针对单聊会话、群聊会话，如果 App 的管理员或者某普通用户希望在所有会话参与者的历史记录中彻底删除一条消息，应使用撤回消息功能。消息成功撤回后，原始消息内容会在所有用户的本地与服务端历史消息记录中删除。
 - IMLib SDK 不提供删除聊天室消息的接口。当前用户的聊天室本地消息在退出聊天室时会被自动删除。开通**聊天室消息云端存储服务**后，如需清除全部用户的聊天室历史消息，可使用服务端 API [清除消息](/platform-chat-api/message/delete)。
 - IMLib SDK 提供删除超级群会话的消息的 API，详见「超级群管理」下的[删除消息][delete-message-ultragroup]。<!--public-cloud-only end-->
:::


## 仅从本地删除指定消息（消息 ID） {#bymessageidlocal}

您可以按消息的 messageId 删除存储在本地数据库内的消息。待删除消息可以属于不同会话。一次最多删除 100 条消息。

如果 App 已经开通**单群聊消息云端存储**服务，服务端保存的该用户的历史消息记录不受影响。如果该用户从服务端获取历史消息，可能会获取到在本地已删除的消息。

#### 接口

```java
RongCoreClient.getInstance().deleteMessages(messageIds, callback);
```

#### 参数说明

|    参数    |                    类型                     |                        说明                         |
|:----------:|:-------------------------------------------:|:-------------------------------------------------:|
| messageIds |                    int[]                    | 消息的 ID 数组。详见[消息介绍]中的 `MessageId` 属性。 |
|  callback  | IRongCoreCallback.ResultCallback\<Boolean\> |                      接口回调                       |


#### 示例代码

```java
int[] messageIds = {12, 13};
RongCoreClient.getInstance().deleteMessages(messageIds, new IRongCoreCallback.ResultCallback<Boolean>() {
             /**
             * 删除消息成功回调
             */
            @Override
            public void onSuccess(Boolean bool) {

            }
            /**
             * 删除消息失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });
```


## 仅从本地删除会话全部历史消息

如果您需要本地清空自己的单聊、群聊或系统会话的历史记录，可以删除指定会话保存在本地数据库中的全部消息。
如果您已经开通**单群聊消息云端存储**服务，服务端保存的操作删除消息用户的历史消息记录不受影响。如果该用户从服务端获取历史消息，可能会获取到在本地已删除的消息。
该接口一次仅允许删除指定的单个会话的消息，会话由会话类型和会话 targetId 参数指定。

#### 接口

```java
RongCoreClient.getInstance().deleteMessages(conversationType, targetId, callback);
```

#### 参数说明

|       参数       |                    类型                     | 说明     |
|:----------------:|:-------------------------------------------:|:-------|
| conversationType |             [ConversationType]              | 会话类型 |
|     targetId     |                   String                    | 会话 ID  |
|     callback     | IRongCoreCallback.ResultCallback\<Boolean\> | 接口回调 |


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";

RongCoreClient.getInstance().deleteMessages(conversationType, targetId, new IRongCoreCallback.ResultCallback<Boolean>() {
        /**
         * 删除消息成功回调
         */
        @Override
        public void onSuccess(Boolean bool) {

        }
        /**
         * 删除消息失败回调
         * @param errorCode 错误码
         */
        @Override
        public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

        }
    });
```

## 删除会话内指定消息（消息对象） {#bymessagelocalremote}

:::tip

 如果 App 已经开通**单群聊消息云端存储**服务，您可以调用以下接口删除 App 用户在融云服务端的历史消息记录。该接口同时从本地和服务端删除消息。
:::


如果 App 用户希望从自己的单聊、群聊或系统会话的历史记录中彻底删除一条消息，可以使用 `deleteRemoteMessage` 从本地和服务端同时删除消息。删除成功后，该用户无法从本地数据库获取消息。如果从服务端获取历史消息，也无法获取到已删除的消息。

该接口允许一次删除指定的单个会话内的一条或一组消息。请确保所提供的消息均属于同一会话（由会话类型和会话 targetId 指定）。


#### 接口

```java
RongCoreClient.getInstance().deleteRemoteMessages(conversationType, targetId, messages, callback);
```

#### 参数说明

|       参数       |                    类型                     | 说明                                                              |
|:----------------:|:-------------------------------------------:|:----------------------------------------------------------------|
| conversationType |             [ConversationType]              | 会话类型。不支持聊天室。                                            |
|     targetId     |                   String                    | 会话 ID                                                           |
|     messages     |                  Message[]                  | 要删除的消息对象 [Message] 数组。请确保所提供的消息均属于同一会话。 |
|     callback     | IRongCoreCallback.ResultCallback\<Boolean\> | 接口回调                                                          |


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";
Message[] messages = {message1, message2};

RongCoreClient.getInstance().deleteRemoteMessages(conversationType, targetId, messages, new IRongCoreCallback.OperationCallback() {
    /**
     * 删除消息成功回调
     */
    @Override
    public void onSuccess() {

    }
    /**
     * 删除消息失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 删除会话历史消息（时间戳） {#bytimestamplocalremote}

:::tip

 此方法可以清除服务器端历史消息和本地消息，如果清除服务器端消息必须先开通**单群聊消息云端存储**服务。

:::

如果 App 用户希望从自己的单聊、群聊或系统会话的历史记录中删除一段历史消息记录，可以使用 `cleanHistoryMessages` 删除会话中早于某个时间点（`recordTime`）的消息。`cleanRemote` 参数控制是否同时删除服务端对应的历史消息。

如果 `cleanRemote` 参数设置为 `true`，表示需要同时删除服务端对应消息，该接口会从本地和服务端同时删除早于 （`recordTime`）的消息。服务端消息删除后，该用户无法再从服务端获取到已删除的消息。该接口一次仅允许删除指定的单个会话的消息，会话由会话类型和会话 ID 参数指定。

#### 接口

```java
RongCoreClient.getInstance().cleanHistoryMessages(conversationType, targetId, recordTime, cleanRemote,callback);
```

#### 参数说明

|       参数       |        类型        | 说明                                                                             |
|:----------------:|:------------------:|:-------------------------------------------------------------------------------|
| conversationType | [ConversationType] | 会话类型                                                                         |
|     targetId     |       String       | 会话 ID                                                                          |
|    recordTime    |        long        | 时间戳。默认删除小于等于 `recordTime` 的消息。如果传 `0`，则删除所有消息。           |
|   cleanRemote    |      boolean       | 是否删除服务器端消息。`true`：同时删除服务端对应消息。`false`：不删除服务端对应消息。 |
|     callback     | OperationCallback  | 接口回调                                                                         |


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";
String recordTime = 0;
Boolean cleanRemote = true; // 同时从服务端删除对应的消息历史记录

RongCoreClient.getInstance().cleanHistoryMessages(conversationType, targetId, recordTime, cleanRemote,
    new IRongCoreCallback.OperationCallback() {
    /**
     * 删除消息成功回调
     */
    @Override
    public void onSuccess() {

    }
    /**
     * 删除消息失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});

```

## 仅从服务端删除会话历史消息（时间戳） {#bytimestampremote}

:::tip

 - 如果 App 已经开通**单群聊消息云端存储**服务，您可以调用以下接口删除 App 用户在融云服务端的历史消息记录。该接口仅从服务端删除消息。
 - 使用融云即时通讯服务端 API 也可以直接删除服务端消息。具体操作请参阅服务端文档[消息清除](/platform-chat-api/message/delete)。
:::


App 用户可以仅删除指定会话保存在服务端的历史消息。该接口提供时间戳参数（`recordTime`），支持删除早于指定时间的消息。如果 `recordTime` 设置为 `0` 则删除该会话保存在服务端的全部历史消息。
#### 接口

```java
RongCoreClient.getInstance().cleanRemoteHistoryMessages(conversationType, targetId, recordTime,callback);
```

#### 参数说明

|       参数       |        类型        |                                       说明                                       |
|:----------------:|:------------------:|:------------------------------------------------------------------------------:|
| conversationType | [ConversationType] |                                     会话类型                                     |
|     targetId     |       String       |                                     会话 ID                                      |
|    recordTime    |        long        | 时间戳。默认删除所有发送时间小于等于该时间戳的消息。传 `0` 表示清除会话中所有消息。 |
|     callback     | OperationCallback  |                                     接口回调                                     |

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE
String targetId = "会话 Id";
String recordTime = 0;

RongCoreClient.getInstance().cleanRemoteHistoryMessages(conversationType, targetId, recordTime, new IRongCoreCallback.OperationCallback() {
    /**
     * 删除消息成功回调
     */
    @Override
    public void onSuccess() {

    }
    /**
     * 删除消息失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

<!-- links -->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[仅从本地删除指定消息（消息 ID）]: #bymessageidlocal
[仅从本地删除会话全部历史消息]: #仅从本地删除会话全部历史消息
[删除会话内指定消息（消息对象）]: #bymessagelocalremote
[删除会话历史消息（时间戳）]: #bytimestamplocalremote
[仅从服务端删除会话历史消息（时间戳）]: #bytimestampremote
[delete-message-ultragroup]: ../ultragroup/delete-message.md
[消息介绍]: ./introduction.md
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[deleteMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/delete-messages.html
[deleteRemoteMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/delete-remote-messages.html
[cleanHistoryMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/clean-history-messages.html
[cleanRemoteHistoryMessages]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/clean-remote-history-messages.html

