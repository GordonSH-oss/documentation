---
title: 获取会话未读消息
sidebar_position: 30
---

# 获取会话未读消息

IMLib SDK 支持从指定会话中获取未读消息，可满足 App 跳转到第一条未读消息、展示全部未读 @ 消息的需求。

## 获取会话中第一条未读消息

获取会话中的最早一条未读消息。

#### 接口
```java
RongCoreClient.getInstance().getTheFirstUnreadMessage(conversationType, targetId, callback);
```
#### 参数说明
| 参数             | 类型                                        | 说明     |
|:-----------------|:--------------------------------------------|:-------|
| conversationType | [ConversationType]                          | 会话类型 |
| targetId         | String                                      | 会话 Id  |
| callback         | IRongCoreCallback.ResultCallback\<Message\> | 回调接口 |

获取成功后，`callback` 中会返回消息对象（[Message]）。

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";


RongCoreClient.getInstance().getTheFirstUnreadMessage(conversationType, targetId, new ResultCallback<Message>() {
        @Override
        public void onSuccess(Message message) {

        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode e) {

        }
 });
```

## 获取会话中未读的 @ 消息

:::tip

 - 低于 5.2.5 版本仅提供不带 `count` 与 `desc` 参数的 `getUnreadMentionedMessages` 方法，每次最多返回 10 条数据。
 - 从 5.2.5 版本开始，`getUnreadMentionedMessages` 支持 `count` 与 `desc` 参数。
:::


获取会话中最早或最新的未读 @ 消息，最多返回 100 条。

#### 接口

```java
RongCoreClient.getInstance().getUnreadMentionedMessages(conversationType, targetId, count, desc, callback);
```

#### 参数说明

| 参数             | 类型                                                | 说明                                                                |
|:-----------------|:----------------------------------------------------|:------------------------------------------------------------------|
| conversationType | [ConversationType]                                  | 会话类型                                                            |
| targetId         | String                                              | 会话 Id                                                             |
| count            | int                                                 | 消息条数。最大 100 条。                                               |
| desc             | boolean                                             | `true`：拉取最新的 `count` 条数据。`false`：拉取最旧的 `count` 条数据。 |
| callback         | IRongCoreCallback.ResultCallback\<List\<Message\>\> | 回调接口                                                            |

获取成功后，`callback` 中会返回消息对象（[Message]）列表。


#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
int count = 100;
boolean desc = true;

RongCoreClient.getInstance().getUnreadMentionedMessages(conversationType, targetId, count, desc, new ResultCallback<List<Message>>() {
        @Override
         public void onSuccess(List<Message> messageList) {

        }

        @Override
         public void onError(IRongCoreEnum.CoreErrorCode e) {

        }
});
```


## 获取 @ 我未读消息的会话列表

:::tip
此功能从 5.28.0 版本开始支持。
:::

#### 接口
```java
RongCoreClient.getInstance().getUnreadMentionMeConversationList(params, callback);
```

#### 参数说明

| 参数             | 类型                                                                        | 说明                                                                |
|:-----------------|:--------------------------------------------------------------------------|:------------------------------------------------------------------|
| params | GetUnreadMentionMeConversationListParams                                  | 请求参数                                                            |
| callback         | IRongCoreCallback.ResultCallback\<List\<Conversation\>\>                       | 回调接口                                                            |

### GetUnreadMentionMeConversationListParams 属性说明

| 参数             | 类型                   | 说明                                                                              |
|:-----------------|:---------------------|:--------------------------------------------------------------------------------|
| conversationTypes | `List<ConversationType>` | 会话类型列表，支持：单聊、群聊、 系统、超级群。                                                        |
| topPriority         | boolean              | 是否置顶优先，传 true 表示置顶会话优先返回，否则结果只以会话时间排序。                                          |
| timestamp            | long                 | 查询的开始时间, 传 0 表示查询最新时间的会话，后续查询传入 conversations 的最后一个元素的 conversation time 单位：毫秒。 |
| count             | int                  | 获取数量，有效值 1-100。 |

#### 示例代码

```java
List<Conversation.ConversationType> types = new ArrayList<>();
types.add(Conversation.ConversationType.PRIVATE);
types.add(Conversation.ConversationType.GROUP);
types.add(Conversation.ConversationType.SYSTEM);
types.add(Conversation.ConversationType.ULTRA_GROUP);

GetUnreadMentionMeConversationListParams params = new GetUnreadMentionMeConversationListParams();
params.setConversationTypes(types);
params.setTopPriority(true);
params.setTimestamp(0);
params.setCount(20);
        
RongCoreClient.getInstance().getUnreadMentionMeConversationList(params, new IRongCoreCallback.ResultCallback<List<Conversation>>() {
    @Override
    public void onSuccess(List<Conversation> conversations) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {

    }
});
```

<!--links-->
[提交工单]: https://console.rongcloud.cn/agile/formwork/ticket/create
[ConversationType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html

