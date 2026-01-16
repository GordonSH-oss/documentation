---
title: 输入状态
sidebar_position: 60
---

# 输入状态

应用程序可以在单聊会话中发送当前用户输入状态。对端收到通知后可以在 UI 展示 “xxx 正在输入”。

## 发送输入状态消息

在当前用户输入文本时调用 [`sendTypingStatus`](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html#-1274792897%2FFunctions%2F1814687565)，发送当前用户输入状态。

IMLib SDK 内部逻辑 6s 内同一用户只能向同一会话发送一条输入状态消息。接收同一会话发送的状态消息处理时间间隔默认为 6s。目前无法修改此配置。

#### 接口
```java
RongCoreClient.getInstance().sendTypingStatus(conversationType, targetId, typingContentType);
```
#### 参数说明
|       参数       |    类型    |        说明         |
|:--------------- |:--------- |:------------------ |
| conversationType|[ConversationType] |会话类型。该接口仅支持单聊会话类型。|
| targetId|String |会话 ID。|
| typingContentType |String|正在输入的消息的类型名。如文本消息，应该传类型名 "RC:TxtMsg"。|

#### 示例代码
```java
 ConversationType conversationType = ConversationType.PRIVATE;
        String targetId = "会话 Id";
        String typingContentType = "RC:TxtMsg";
        RongCoreClient.getInstance()
                .sendTypingStatus(conversationType, targetId, typingContentType);
```

## 监听输入状态

应用程序可以使用 [`setTypingStatusListener`](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html#-1732639434%2FFunctions%2F1814687565) 设置对方输入状态监听，监听在单聊类型的会话收到的输入状态通知。在收到对端发来的输入状态通知时，SDK 会通过 `onTypingStatusChanged` 回调方法返回当前正在输入的用户列表和消息类型。

#### 示例代码

```java
 RongCoreClient.getInstance().setTypingStatusListener(new IRongCoreListener.TypingStatusListener() {
            @Override
            public void onTypingStatusChanged(Conversation.ConversationType type, String targetId, Collection<TypingStatus> typingStatusSet) {
                //当输入状态的会话类型和 targetID 与当前会话一致时，才需要显示
                if (type.equals(mConversationType) && targetId.equals(mTargetId)) {
                    // count 表示当前会话中正在输入的用户数量，目前只支持单聊
                    int count = typingStatusSet.size();
                    if (count > 0) {
                        Iterator iterator = typingStatusSet.iterator();
                        TypingStatus status = (TypingStatus) iterator.next();
                        String objectName = status.getTypingContentType();

                        MessageTag textTag = TextMessage.class.getAnnotation(MessageTag.class);
                        MessageTag voiceTag = VoiceMessage.class.getAnnotation(MessageTag.class);
                        //匹配对方正在输入的是文本消息还是语音消息
                        if (objectName.equals(textTag.value())) {
                            //显示 “对方正在输入”
                            mHandler.sendEmptyMessage(SET_TEXT_TYPING_TITLE);
                        } else if (objectName.equals(voiceTag.value())) {
                            //显示 "对方正在讲话"
                            mHandler.sendEmptyMessage(SET_VOICE_TYPING_TITLE);
                        }
                    } else {
                        //当前会话没有用户正在输入，标题栏仍显示原来标题
                        mHandler.sendEmptyMessage(SET_TARGETID_TITLE);
                    }
                }
            }
        });
```

<!-- api links -->
[ConversationType]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html

