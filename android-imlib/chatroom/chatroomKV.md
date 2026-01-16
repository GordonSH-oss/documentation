---
title: 聊天室属性管理（KV）
---

# 聊天室属性管理（KV）

IMLib SDK 在聊天室业务核心类 [RongChatRoomClient] 中提供了聊天室属性（KV）管理接口，用于在指定聊天室中设置自定义属性。

在语音直播聊天室场景中，可利用此功能记录聊天室中各麦位的属性；或在狼人杀等卡牌类游戏场景中记录用户的角色和牌局状态等。

## 开通服务

使用聊天室属性（KV）接口要求开通**聊天室属性自定义设置**服务。您可以前往融云控制台 [**IM 服务** > **功能配置** > **聊天室**](https://console.rongcloud.cn/agile/im/service/config#%E8%81%8A%E5%A4%A9%E5%AE%A4) 页面开启该服务。

聊天室业务提供服务端回调功能，支持由融云服务端将应用下的全部聊天室属性变化（设置，删除，全部删除等操作）同步到您指定的地址，方便 App 业务服务端了解聊天室属性变化。详见服务端文档[聊天室属性同步（KV）](/platform-chat-api/chatroom/kv-attribute/sync-kv)。

## 功能局限

:::tip

 - 聊天室销毁后，聊天室中的自定义属性同时销毁。
 - 每个聊天室中，最多允许设置 **100** 个属性信息，以 `Key-Value` 的方式进行存储。
 - 客户端 SDK 未针对聊天室属性 KV 的操作频率进行限制。建议每个聊天室，每秒钟操作 `Key-Value` 频率保持在 **100** 次及以下（一秒内单次操作 100 个 KV 等同于操作 100 次）。
:::


## 监听聊天室属性变化

SDK 在聊天室业务核心类 [RongChatRoomClient] 中提供了 [KVStatusListener]，用于监听聊天室属性（KV）变化。

| 返回值 | 方法                                                                   | 说明                                                                                   |
|:-------|:-----------------------------------------------------------------------|--------------------------------------------------------------------------------------|
| void   | `onChatRoomKVSync(String roomId)`                                      | 聊天室 KV 列表同步完成时触发。                                                          |
| void   | `onChatRoomKVUpdate(String roomId, Map<String, String> chatRoomKvMap)` | 聊天室 KV 列表更新完成时触发，首次同步 KV 时返回全量 KV，后续触发时仅返回新增、修改的 KV。 |
| void   | `onChatRoomKVRemove(String roomId, Map<String, String> chatRoomKvMap)` | KV 被删除时触发。                                                                       |

- `onChatRoomKVSync(String roomId)`

    触发时机：加入聊天室成功后，IMLib SDK 默认从服务端同步 KV 列表，同步完成后触发。

    | 参数   | 类型   | 说明      |
    |:-------|:-------|:--------|
    | roomId | String | 聊天室 Id |

- `onChatRoomKVUpdate(String roomId, Map<String, String> chatRoomKvMap)`

    触发时机：聊天室 KV 更新时。

    如果刚进入聊天室时存在 KV，会通过此回调将所有 KV 返回，再次回调时为其他人设置或者修改 KV 的增量数据。

    | 参数          | 类型                  | 说明                |
    |:--------------|:----------------------|:------------------|
    | roomId        | String                | 聊天室 Id           |
    | chatRoomKvMap | Map\<String, String\> | 发生变化的聊天室 KV |


- `onChatRoomKVRemove(String roomId, Map<String, String> chatRoomKvMap)`

    触发时机：KV 被删除时触发。

    | 参数          | 类型                  | 说明              |
    |:--------------|:----------------------|:----------------|
    | roomId        | String                | 聊天室 Id         |
    | chatRoomKvMap | Map\<String, String\> | 被删除的聊天室 KV |

### 添加聊天室 KV 监听器

使用 [addKVStatusListener] 方法，添加聊天室 KV 监听器 [KVStatusListener]。支持设置多个监听器。为了避免内存泄露，请在不需要监听时将监听器移除。

#### 接口原型

```java
RongChatRoomClient.getInstance().addKVStatusListener(listener);
```

### 移除聊天室 KV 监听器

SDK 支持移除监听器。为了避免内存泄露，请在不需要监听时将监听器移除。

#### 接口

```java
RongChatRoomClient.getInstance().removeKVStatusListener(listener);
```

### 聊天室 KV 监听器扩展

如果需要严格按照聊天室属性设置的顺序返回发生变化的 KV 键值对，可以使用 `KVStatusExListener` 接口。该接口继承自 `KVStatusListener`，并新增了 `onChatRoomKVChange()` 方法。


#### 示例代码

```java
RongChatRoomClient.getInstance().addKVStatusListener(new RongChatRoomClient.KVStatusExListener() {
    @Override
    public void onChatRoomKVSync(String roomId) {
    }

    @Override
    public void onChatRoomKVUpdate(String roomId, Map<String, String> chatRoomKvMap) {
    }

    @Override
    public void onChatRoomKVRemove(String roomId, Map<String, String> chatRoomKvMap) {
    }

    /**
     * 聊天室 KV 变化的回调。
     * 本回调包含 onChatRoomKVUpdate 和 onChatRoomKVRemove 的回调数据。
     * 
     * @param roomId 聊天室 Id
     * @param changeInfos 发生变化的 KV，严格按照聊天室属性设置时的顺序。
     */
    @Override
    public void onChatRoomKVChange(String roomId, ChatroomKVChangeInfo[] changeInfos) {
    }
});
```

## 获取单个属性

使用 [getChatRoomEntry]方法， 通过属性的 Key 获取指定聊天室中的单个属性 KV。

#### 接口

```java
RongChatRoomClient.getInstance().getChatRoomEntry(chatRoomId, key, callback);
```

#### 参数说明

| 参数       | 类型                                   | 说明                                                                                           |
|:-----------|:---------------------------------------|:---------------------------------------------------------------------------------------------|
| chatRoomId | String                                 | 聊天室 ID                                                                                      |
| key        | String                                 | 聊天室属性名称,Key 支持大小写英文字母、数字、部分特殊符号 + = - _ 的组合方式，最大长度 128 个字符 |
| callback   | ResultCallback\<Map\<String, String\>> | 回调接口                                                                                       |

#### 示例代码

```java
String chatRoomId = "聊天室 ID";
String key = "name";

RongChatRoomClient.getInstance().getChatRoomEntry(chatRoomId, key, new IRongCoreCallback.ResultCallback<Map<String, String>>() {
        @Override
        public void onSuccess(Map<String, String> stringStringMap) {

            }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode e) {

           }
 });
```

## 获取所有属性

使用 [getAllChatRoomEntries] 方法获取指定聊天室中所有属性 KV 对。

#### 接口

```java
RongChatRoomClient.getInstance().getAllChatRoomEntries(chatRoomId, callback);
```

#### 参数说明


| 参数       | 类型                                   | 说明      |
|:-----------|:---------------------------------------|:--------|
| chatRoomId | String                                 | 聊天室 ID |
| callback   | ResultCallback\<Map\<String, String\>> | 回调接口  |

#### 示例代码

```java
String chatRoomId = "聊天室 ID";

RongChatRoomClient.getInstance().getAllChatRoomEntries(chatRoomId, new IRongCoreCallback.ResultCallback<Map<String, String>>() {
         @Override
         public void onSuccess(Map<String, String> stringStringMap) {

            }

          @Override
         public void onError(IRongCoreEnum.CoreErrorCode e) {

            }
        });
```


## 设置单个属性

使用 [setChatRoomEntry] 方法在指定单个聊天室中设置单个属性 KV 对。注意，该接口不支持更新已存在的属性。

#### 接口 

```java
RongChatRoomClient.getInstance().setChatRoomEntry(chatRoomId, key, value, sendNotification, isAutoDel, notificationExtra, callback);
```

#### 参数说明

在设置 KV 时，可指定 SDK 发送一条聊天室属性通知消息（`ChatRoomKVNotiMessage`）。消息内容结构说明可参见[通知类消息格式]。

| 参数              | 类型              | 说明                                                                                                                                                        |
|:------------------|:------------------|:----------------------------------------------------------------------------------------------------------------------------------------------------------|
| chatRoomId        | String            | 聊天室 ID                                                                                                                                                   |
| key               | String            | 聊天室属性名称，Key 支持大小写英文字母、数字、部分特殊符号 + = - _ 的组合方式，最大长度 128 个字符                                                              |
| value             | String            | 聊天室属性对应的值，最大长度 4096 个字符                                                                                                                     |
| sendNotification  | boolean           | `true` 发送通知; `false` 不发送。如果发送通知，SDK 会接收到类型标识为 `RC:chrmKVNotiMsg` 的聊天室属性通知消息（`ChatRoomKVNotiMessage`），并且消息内容中包含 K，V |
| autoDelete        | boolean           | 退出后是否删除                                                                                                                                              |
| notificationExtra | String            | 通知的自定义字段，RC:chrmKVNotiMsg(ChatRoomKVNotiMessage), 通知消息中会包含此字段，最大长度 2048 个字符                                                       |
| callback          | OperationCallback | 回调接口                                                                                                                                                    |

#### 示例代码

```java
String chatRoomId = "聊天室 ID";
String key = "name";
String value = "融融";
boolean sendNotification = true;
boolean isAutoDel = false;
String notificationExtra = "通知消息扩展";

RongChatRoomClient.getInstance().setChatRoomEntry(chatRoomId, key, value, sendNotification, isAutoDel, notificationExtra, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});

```

## 强制设置单个属性

使用 [forceSetChatRoomEntry] 方法强制设置聊天室属性。以 `key , value ` 的形式存储。当 key 不存在时，代表增加属性。当 key 已经存在时，代表更新属性的值。使用强制设置可修改他人创建的属性值。在设置 KV 时，可指定 SDK 发送一条聊天室属性通知消息（`ChatRoomKVNotiMessage`）。消息内容结构说明可参见[通知类消息格式]。

#### 接口

```java
RongChatRoomClient.getInstance().forceSetChatRoomEntry(chatRoomId, key, value, sendNotification, isAutoDel, notificationExtra, callback);
```

#### 参数说明


| 参数              | 类型              | 说明                                                                                                                          |
|:------------------|:------------------|:----------------------------------------------------------------------------------------------------------------------------|
| chatRoomId        | String            | 聊天室 ID                                                                                                                     |
| key               | String            | 聊天室属性名称,Key 支持大小写英文字母、数字、部分特殊符号 + = - _ 的组合方式，最大长度 128 个字符                                |
| value             | String            | 聊天室属性对应的值，最大长度 4096 个字符                                                                                       |
| sendNotification  | boolean           | true 发送通知; false 不发送. 如果发送通知，SDK 会接收到 RC:chrmKVNotiMsg 通知消息 ChatRoomKVNotiMessage，并且消息内容中包含 K，V |
| autoDelete        | boolean           | 退出后是否删除                                                                                                                |
| notificationExtra | String            | 通知的自定义字段，类型标识为 `RC:chrmKVNotiMsg` 的 `ChatRoomKVNotiMessage` 通知消息中会包含此字段，最大长度 2048 个字符         |
| callback          | OperationCallback | 回调接口                                                                                                                      |

#### 示例代码

```java
String chatRoomId = "聊天室 ID";
String key = "name";
String value = "融融";
boolean sendNotification = true;
boolean isAutoDel = false;
String notificationExtra = "通知消息扩展";

RongChatRoomClient.getInstance().forceSetChatRoomEntry(chatRoomId, key, value, sendNotification, isAutoDel, notificationExtra, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }


    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 批量设置属性

使用 [setChatRoomEntries] 方法批量设置聊天室属性。通过 `isForce` 参数可控制是否强制覆盖他人设置的属性。

#### 接口

```java
RongChatRoomClient.getInstance().setChatRoomEntries(chatRoomId, chatRoomEntryMap, isAutoDel, isForce, callback);
```

#### 参数说明


| 参数             | 类型                  | 说明                                                                                  |
|:-----------------|:----------------------|:------------------------------------------------------------------------------------|
| chatRoomId       | String                | 聊天室 ID                                                                             |
| chatRoomEntryMap | Map                   | chatRoomEntryMap集合size最大限制为10,超过限制返回错误码 KV_STORE_OUT_OF_LIMIT (23429) |
| isAutoDel        | boolean               | 用户掉线或退出时，是否自动删除该 Key、Value 值                                          |
| isForce          | boolean               | 是否强制覆盖                                                                          |
| callback         | SetChatRoomKVCallback | 回调接口                                                                              |

如果部分失败，SDK 会触发 [SetChatRoomKVCallback] 的 `onError` 回调方法：

| 回调参数  | 回调类型      | 说明                                                                                                             |
|:----------|:--------------|:---------------------------------------------------------------------------------------------------------------|
| errorCode | CoreErrorCode | 错误码                                                                                                           |
| map       | Map           | 当 errorCode 为 KV_STORE_NOT_ALL_SUCCESS（23428）时,map 才会有值（key 为设置失败的 key，value 为该 key 对应的错误码） |

#### 示例代码

```java
RongChatRoomClient.getInstance().setChatRoomEntries(chatRoomId, chatRoomEntryMap, isAutoDel, isForce, new IRongCoreCallback.SetChatRoomKVCallback() {
    /**
     * 成功回调
     */
    @Override
    public void onSuccess() {

    }

    /**
     * 失败回调
     * @param errorCode
     * @param map
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode, Map<String, IRongCoreEnum.CoreErrorCode> map) {

    }
});
```

## 删除单个属性

使用 [removeChatRoomEntry] 方法删除指定单个聊天室的单个属性。该接口仅支持删除当前用户所创建的属性。在删除 KV 时，可指定 SDK 发送一条聊天室属性通知消息（`ChatRoomKVNotiMessage`）。消息内容结构说明可参见[通知类消息格式]。

#### 接口

```java
RongChatRoomClient.getInstance().removeChatRoomEntry(chatRoomId, key, sendNotification, notificationExtra, callback);
```

#### 参数说明

| 参数              | 类型              | 说明                                                                                                                          |
|:------------------|:------------------|:----------------------------------------------------------------------------------------------------------------------------|
| chatRoomId        | String            | 聊天室 ID                                                                                                                     |
| key               | String            | 聊天室属性名称,Key 支持大小写英文字母、数字、部分特殊符号 + = - _ 的组合方式，最大长度 128 个字符                                |
| sendNotification  | boolean           | true 发送通知; false 不发送. 如果发送通知，SDK 会接收到 RC:chrmKVNotiMsg 通知消息 ChatRoomKVNotiMessage，并且消息内容中包含 K，V |
| notificationExtra | String            | 通知的自定义字段，类型标识为 `RC:chrmKVNotiMsg` 的 `ChatRoomKVNotiMessage` 通知消息中会包含此字段，最大长度 2048 个字符         |
| callback          | OperationCallback | 回调接口                                                                                                                      |


#### 示例代码

```java
String chatRoomId = "聊天室 ID";
String key = "name";
boolean sendNotification = true;
String notificationExtra = "通知消息扩展";

RongChatRoomClient.getInstance().removeChatRoomEntry(chatRoomId, key, sendNotification, notificationExtra, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }


    @Override
   public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 强制删除单个属性

使用 [forceRemoveChatRoomEntry] 方法强制删除指定单个聊天室的单个属性。可删除他人所创建的属性。
在强制删除 KV 时，可指定 SDK 发送一条聊天室属性通知消息（`ChatRoomKVNotiMessage`）。消息内容结构说明可参见[通知类消息格式]。

#### 接口

```java
RongChatRoomClient.getInstance().forceRemoveChatRoomEntry(chatRoomId, key, sendNotification, notificationExtra, callback);
```

#### 参数说明

| 参数              | 类型              | 说明                                                                                                                          |
|:------------------|:------------------|:----------------------------------------------------------------------------------------------------------------------------|
| chatRoomId        | String            | 聊天室 ID                                                                                                                     |
| key               | String            | 聊天室属性名称，Key 支持大小写英文字母、数字、部分特殊符号 + = - _ 的组合方式，最大长度 128 个字符。                               |
| sendNotification  | boolean           | true 发送通知; false 不发送. 如果发送通知，SDK 会接收到 RC:chrmKVNotiMsg 通知消息 ChatRoomKVNotiMessage，并且消息内容中包含 KV。 |
| notificationExtra | String            | 通知的自定义字段，类型标识为 `RC:chrmKVNotiMsg` 的 `ChatRoomKVNotiMessage` 通知消息中会包含此字段，最大长度 2048 个字符。        |
| callback          | OperationCallback | 回调接口                                                                                                                      |


#### 示例代码

```java
String chatRoomId = "聊天室 ID";
String key = "name";
boolean sendNotification = true;
String notificationExtra = "通知消息扩展";

RongChatRoomClient.getInstance().forceRemoveChatRoomEntry(chatRoomId, key, sendNotification, notificationExtra, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```


## 批量删除属性

使用 [deleteChatRoomEntries] 方法从指定单个聊天室中批量删除属性。单次最多删除 10 个属性。通过 `isForce` 参数可控制是否强制删除他人设置的属性。

#### 接口

```java
RongChatRoomClient.getInstance().deleteChatRoomEntries(chatRoomId, keys, isForce, callback);
```

#### 参数说明

| 参数       | 类型                  | 必填 | 说明                                     |
|:-----------|:----------------------|:----|:---------------------------------------|
| chatRoomId | String                | 是   | 聊天室 ID                                |
| keys       | List                  | 是   | 聊天室属性名称集合，集合 size 最大限制 10 |
| isForce    | boolean               | 是   | 是否强制覆盖                             |
| callback   | SetChatRoomKVCallback | 是   | 回调接口                                 |

如果部分失败，SDK 会触发 [SetChatRoomKVCallback] 的 `onError` 回调方法：

| 回调参数  | 回调类型                                   | 说明                                                                                                             |
|:----------|:-------------------------------------------|:---------------------------------------------------------------------------------------------------------------|
| errorCode | CoreErrorCode                              | 错误码                                                                                                           |
| map       | `Map<String, IRongCoreEnum.CoreErrorCode>` | 当 errorCode 为 KV_STORE_NOT_ALL_SUCCESS（23428）时,map 才会有值（key 为设置失败的 key，value 为该 key 对应的错误码） |


#### 示例代码

```java
RongChatRoomClient.getInstance().deleteChatRoomEntries(chatRoomId, keys, isForce, new IRongCoreCallback.SetChatRoomKVCallback() {
    /**
     * 成功回调
     */
    @Override
    public void onSuccess() {

    }

    /**
     * 失败回调
     * @param errorCode 错误码
     */
    @Override
    public void onError(IRongCoreEnum.CoreErrorCode errorCode, Map<String, IRongCoreEnum.CoreErrorCode> map) {

    }
});
```

<!-- links -->
[deleteChatRoomEntries]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/delete-chat-room-entries.html
[forceRemoveChatRoomEntry]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/force-remove-chat-room-entry.html
[removeChatRoomEntry]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/remove-chat-room-entry.html
[setChatRoomEntries]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/set-chat-room-entries.html
[forceSetChatRoomEntry]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/force-set-chat-room-entry.html
[setChatRoomEntry]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/set-chat-room-entry.html
[getAllChatRoomEntries]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/get-all-chat-room-entries.html
[addKVStatusListener]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/add-k-v-status-listener.html
[通知类消息格式]: /platform-chat-api/message-about/objectname-notification
[RongChatRoomClient]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/index.html
[SetChatRoomKVCallback]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-callback/-set-chat-room-k-v-callback/index.html
[KVStatusListener]: https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/-k-v-status-listener/index.html
[getChatRoomEntry]:https://doc.rongcloud.cn/apidoc/chatroom-android/latest/zh_CN/html/-android--chatroom--s-d-k/io.rong.imlib.chatroom.base/-rong-chat-room-client/get-chat-room-entry.html

