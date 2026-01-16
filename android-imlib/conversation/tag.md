---
title: 设置与使用会话标签
sidebar_position: 80
---

# 设置与使用会话标签

> - SDK 从 5.1.1 版本开始支持会话标签功能，相关接口仅在 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 中提供。
> - 在为会话设置标签前，请确保已创建标签信息。详见[管理标签信息数据]。<!--public-cloud-only start-->
> - 本功能不适用于聊天室、超级群。<!--public-cloud-only end-->

每个用户最多可以创建 20 个标签，每个标签下最多可以添加 1000 个会话。如果标签下已添加 1000 个会话，继续在该标签下添加会话仍会成功，但会导致最早添加标签的会话被移除标签。

## 场景描述

会话标签常实现 App 用户对会话进行分组的需求。创建标签信息（[TagInfo]）后，App 用户可以为会话设置一个或多个标签。

设置标签后，可以利用会话的标签数据实现会话的分组获取、展示、删除等特性。还可以获取指定标签下所有会话的消息未读数，或在特定标签下设置某个会话置顶。

- **场景 1**：对会话列表中的每个会话打 tag，类似企业微信会话列表中的外部群，部门群，个人群等 tag。
- **场景 2**：通讯录根据 tag 来分组，类似 QQ 好友列表中的家人，朋友，同事分组等。
- **场景 3**：前两个场景的结合，按照 tag 来进行会话列表分组，类似 Telegram 的会话列表分组。

## 使用标签标记会话

在创建标签信息（[TagInfo]）后，App 用户可以使用标签标记会话。SDK 将用标签标记会话的操作视为将会话添加到标签中。

支持以下操作：

- 标记会话，即将一个或多个会话添加到指定标签
- 从标签中移除一个或多个会话
- 为指定会话移除一个或多个标签

### 将一个或多个会话添加到指定标签

#### 接口

```java
RongCoreClient.getInstance().addConversationsToTag(tagId, conversationIdentifierList,callback)
```

#### 参数说明
|参数 | 类型 | 说明|
|:-----|:-----|:----|
|tagId | String | 标签 ID |
|conversationIdentifierList | List\<ConversationIdentifier\> | 会话标识列表。每个 [ConversationIdentifier] 中需要指定会话类型（[ConversationType]）和 Target ID。|
|callback | OperationCallback | 回调|

#### 示例代码
```java
RongCoreClient.getInstance().addConversationsToTag(tagId, conversationIdentifierList,
    new IRongCoreCallback.OperationCallback() {
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
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });
```



### 从指定标签下移除会话

App 用户可能需要携带指定标签的会话中移除一个或多个会话。例如，在所有添加了「培训班」标签的会话中移除与「Tom」的私聊会话。SDK 将该操作视为从指定标签中移除会话。移除成功后，会话仍然存在，但不再携带该标签。

#### 接口
```java
RongCoreClient.getInstance().removeConversationsFromTag(tagId, conversationIdentifierList, callback);
```
#### 参数说明
|参数 | 类型 | 说明|
|:-----|:-----|:----|
|tagId | String | 标签 ID |
|conversationIdentifierList | List\<ConversationIdentifier\> | 会话标识列表。每个 [ConversationIdentifier] 中需要指定会话类型（[ConversationType]）和 Target ID。|
|callback | OperationCallback | 回调|

#### 示例代码
```java
RongCoreClient.getInstance().removeConversationsFromTag(
                        tagId,
                        conversationIdentifierList,
                        new IRongCoreCallback.OperationCallback() {
                            @Override
                            public void onSuccess() {
                            }

                            @Override
                            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
                                
                            }
                        });

```

### 为指定会话中移除标签

App 用户可能为指定会话中添加了多个标签。SDK 支持一次移除单个或多个标签。移除时需要传入所有待移除 [TagInfo] 的 `tagId` 列表。


#### 接口
```java
RongCoreClient.getInstance(). removeTagsFromConversation(conversationIdentifier, tagIds, callback);

```

#### 参数说明

|参数 | 类型 | 说明|
|:-----|:-----|:----|
|conversationIdentifier | [ConversationIdentifier] | 会话标识，需要指定会话类型（[ConversationType]）和 Target ID。|
|tagIds | List\<String\> tagIds | 标签 id 列表。 |
|callback | IRongCoreCallback.OperationCallback() | 结果回调。|


#### 示例代码
```java
RongCoreClient.getInstance(). removeTagsFromConversation(conversationIdentifier, tagIds,
    new IRongCoreCallback.OperationCallback() {
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
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });
```

### 获取指定会话的所有标签

获取指定会话携带的所有标签。获取成功后，回调中返回 [ConversationTagInfo] 的列表。每个 [ConversationTagInfo] 中包含对应的标签信息 [TagInfo] 和置顶状态信息（会话是否在携带该标签信息的所有会话中置顶）。


#### 参数说明

|参数 | 类型 | 说明|
|:-----|:-----|:----|
|conversationIdentifier | [ConversationIdentifier] | 会话标识，需要指定会话类型（[ConversationType]）和 Target ID。|
|callback | ResultCallback\<List\<ConversationTagInfo\>\> | 结果回调，返回会话携带的所有标签。|

#### 示例代码

```java
RongCoreClient.getInstance().getTagsFromConversation(conversationIdentifier,
    new IRongCoreCallback.ResultCallback<List<ConversationTagInfo>>() {
            /**
             * 成功回调
             */
            @Override
            public void onSuccess(List<ConversationTagInfo> conversationTagInfos) {

            }

            /**
             * 失败回调
             * @param errorCode 错误码
             */
            @Override
            public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

            }
        });
```

## 多端同步会话标签修改


融云支持同一用户账号在多端登录。设置会话标签变更监听器 [IRongCoreListener.ConversationTagListener](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-listener/index.html#957566688%2FClasslikes%2F1814687565) 后，当用户在其它端添加、移除、编辑会话上的标签时，都会触发 `onConversationTagChanged` 回调，可在当前设备上接收到来自其他设备的会话标签修改通知。收到通知后需调用 `getTagsFromConversation` 方法从融云服务端获取指定会话的最新标签数据。

:::tip

 - 请在初始化之后，连接之前调用该方法。
 - 在当前设备上修改标签信息不会触发该回调方法。服务端仅会通知 SDK 在同一用户账号登录的其他设备上触发回调。
:::

#### 示例代码

```java
RongCoreClient.getInstance().setConversationTagListener(new IRongCoreListener.ConversationTagListener{
    @Override
    public void onConversationTagChanged() {
    }
});
```

## 按标签操作会话数据

IMLib SDK 支持对携带指定标签的会话进行操作。App 用户为会话添加标签后，可以实现以下操作：

- 配合使用[会话置顶]功能，可以实现在携带指定标签的会话中置顶会话
- 按标签获取会话列表，即获取携带指定标签的所有会话
- 按标签获取未读消息数
- 清除标签对应会话的未读消息数
- 删除标签对应的会话

### 在携带指定标签的会话中置顶

您可以使用会话标签按照业务需求对会话进行分类和展示。如果需要在同一类会话（携带同一标签的所有会话）中置顶显示会话，可以使用会话置顶功能。

详细实现方式请参见[会话置顶]的**在标签下置顶会话**。

### 分页获取本地指定标签下会话列表

以会话中的操作时间 `operationTime` 为界，分页获取本地指定标签下会话列表。该方法仅从本地数据库中获取数据。从 5.6.4 版本开始，该接口返回的 `Conversation` 对象新增 `isTopForTag` 属性，如果为 `true`，表示该会话在当前 `tagId` 下为置顶会话。


#### 接口

```java
RongCoreClient.getInstance().getConversationsFromTagByPage(tagId, ts, count, callback);
```

#### 参数说明

|参数 | 类型 | 说明|
|:-----|:-----|:----|
|tagId | String | 标签 ID |
|ts | long | 会话的时间戳。获取这个时间戳之前的会话列表。首次可传 `0`，后续可以使用返回的 [Conversation] 对象的 `sentTime` 或 `operationTime` 属性值，作为下一次查询的 `startTime`。推荐使用 `operationTime`（该属性仅在 5.6.8 及之后版本提供）。|
|count | int | 获取数量(20 ≦ count ≦ 100) |
|callback | ResultCallback\<List\<Conversation\>\> | 返回携带指定 tagId 的会话列表。 |

#### 示例代码
```java
String tagId = "peixunban";
long ts = 0;
int count = 20;

RongCoreClient.getInstance().getConversationsFromTagByPage(tagId, ts, count, new IRongCoreCallback.ResultCallback<List<Conversation>>() {
    @Override
    public void onSuccess(List<Conversation> conversations) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```



### 分页获取本地没有标签的会话列表

:::tip

 从 5.24.0 版本开始支持。

:::

您可使用 `getUntaggedConversationListByPage` 接口获取未标记的会话列表。

#### 接口

```java
RongCoreClient.getInstance().getUntaggedConversationListByPage(timestamp, count， topPriority，callback);
```

#### 参数说明

| 参数        | 类型                                                 | 说明                                                         |
| :---------- | :--------------------------------------------------- | :----------------------------------------------------------- |
| `timestamp`   |`Long`                                                 | 会话的时间戳（获取这个时间戳之前的会话列表，0表示从最新开始获取，单位：毫秒）。 |
| `count`       | `Int`                                                  | 获取的数量，有效值 [1, 100]。（当实际取回的会话数量小于 count 值时，表明已取完数据）。当 count 小于 1 时，取 20，大于 100 *     时，取 100。 |
| `topPriority` | `Boolean`                                              | 查询结果的排序方式，是否置顶优先，传 true 表示置顶会话优先返回，否则结果只以会话时间排序。 |
| `callback`    | `IRongCoreCallback.ResultCallback<List<Conversation>>` | 结果回调。                                                   |

#### 示例代码

```
long timestamp = 0;
int count = 20;
boolean topPriority = true;

RongCoreClient.getInstance().getUntaggedConversationListByPage(timestamp, count,topPriority,new IRongCoreCallback.ResultCallback<List<Conversation>>() {

	@Override
	public void onSuccess(List<Conversation> conversations) {
		// do nothing
	}

	@Override
	public void onError(IRongCoreEnum.CoreErrorCode e) {
		// do nothing
	}
});
```



### 按标签获取未读消息数

获取携带指定标签的所有会话的未读消息数。

#### 接口

```java
RongCoreClient.getInstance().getUnreadCountByTag(tagId, containBlocked, callback);
```

#### 参数说明
|参数 | 类型 | 说明 |
|:-----|:-----|:----|
|tagId | String | 标签 ID |
|containBlocked | boolean | 是否包含免打扰消息 |
|callback | ResultCallback\<Integer\> | 按标签获取未读消息数回调|

#### 示例代码
```java
String tagId = "peixunban";
boolean containBlocked = true;

RongCoreClient.getInstance().getUnreadCountByTag(tagId, containBlocked, new IRongCoreCallback.ResultCallback<Integer>() {
    @Override
    public void onSuccess(Integer integer) {
                
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```

### 清除标签对应会话的未读消息数

:::tip

 SDK 从 5.1.5 开始提供该接口。
:::


清除携带指定标签的所有会话的未读消息数。

#### 接口

```java
RongCoreClient.getInstance().clearMessagesUnreadStatusByTag(tagId, callback)
```
#### 参数说明

| 参数     | 类型                             | 说明                       |
| -------- | -------------------------------- | -------------------------- |
| tagId    | String                           | 标签 ID |
| callback | IRongCoreCallback.ResultCallback | 操作结果回调               |

### 删除标签对应的会话

:::tip

 SDK 从 5.1.5 开始提供该接口。
:::


删除指定标签下的全部会话，同时解除这些会话和标签的绑定关系。删除成功后，会话不再携带指定的标签。这些会话收到新消息时，会产生新的会话。

#### 接口

```java
RongCoreClient.getInstance().clearConversationsByTag(tagId, deleteMessage, callback)
```
#### 参数说明

| 参数          | 类型                             | 说明                                     |
| ------------- | -------------------------------- | ---------------------------------------- |
| tagId         | String                           | 标签 ID。             |
| deleteMessage | boolean                          | 是否清除该标签下所有会话的本地历史消息。 |
| callback      | IRongCoreCallback.ResultCallback | 操作结果回调。                           |

#### 示例代码
```java
String tagId = "peixunban";
boolean deleteMessage = true;

RongCoreClient.getInstance().clearConversationsByTag(tagId, deleteMessage, new IRongCoreCallback.ResultCallback<Boolean>() {
    @Override
    public void onSuccess(Boolean aBoolean) {
                
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```
您可以通过 `deleteMessage` 参数配置是否同时清除这些会话对应的本地消息。

- 如果不删除会话对应的本地消息，再接收到新消息时，可以看到历史聊天记录。
- 如果删除会话对应的本地消息，再接收到新消息时，无法看到历史聊天记录。如果开通了**单群聊消息云端存储服务**，服务端仍保存有消息历史，使用 `getMessages` 方法仍可以获取到历史消息。如需删除，请使用[删除服务端历史消息接口](../message/delete.md)。


<!-- links -->
[管理标签信息数据]: ./manage-tag-info.md
[会话置顶]: ./stick-to-top.md
<!-- api links -->
[TagInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-tag-info/
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[ConversationIdentifier]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation-identifier/index.html
[ConversationTagInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation-tag-info/index.html
[IRongCoreListener.ConversationTagListener]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-listener/-conversation-tag-listener/index.html
[Conversation]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/index.html

