---
title: 会话草稿
sidebar_position: 50
---

# 会话草稿

## 保存草稿

支持保存一条草稿内容至指定会话。保存草稿会更新会话 `sentTime`，会话列表顺序会发生改变，该会话会排在会话列表前面。

#### 接口
```java
 RongCoreClient.getInstance().saveTextMessageDraft(conversationType,targetId,content,callback); 
```
#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------|:------------------ |
|conversationType|[ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html) |会话类型|
|targetId|String |会话 Id|
|content|String |草稿的文字内容|
|callback |ResultCallback\<Boolean\>|回调接口|

#### 示例代码

```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";
String content = "草稿内容";

 RongCoreClient.getInstance().saveTextMessageDraft(
                        conversationType,
                        targetId,
                        content,
                        new IRongCoreCallback.ResultCallback<Boolean>() {
                            @Override
                            public void onSuccess(Boolean aBoolean) {
                               
                            }

                            @Override
                            public void onError(IRongCoreEnum.CoreErrorCode e) {
                               
                            }
                        });

```



## 获取草稿

获取草稿内容。

#### 接口
```java
RongCoreClient.getInstance().getTextMessageDraft(conversationType,targetId, callback);
```
#### 参数说明

|       参数       |    类型   |        说明         |
|:---------------|:---------|:------------------ |
|conversationType|[ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html) |会话类型|
|targetId|String |会话 Id|
|callback |ResultCallback\<String\>|回调接口|

#### 示例代码
```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = " 会话 Id ";

RongCoreClient.getInstance().getTextMessageDraft(
                        conversationType,
                        targetId,
                        new IRongCoreCallback.ResultCallback<String>() {
                            @Override
                            public void onSuccess(String s) {
                            }

                            @Override
                            public void onError(IRongCoreEnum.CoreErrorCode e) {
                                
                            }
                        });

```


## 删除草稿

删除草稿后，因会话操作时间已经发生改变，所以会话列表顺序不会发生改变，依旧排序在会话列表前面。

#### 接口

```java
 RongCoreClient.getInstance().clearTextMessageDraft(conversationType,targetId,callback);
```

#### 参数说明

|       参数       |    类型     |        说明         |
|:--------------- |:---------  |:------------------|
|conversationType|[ConversationType](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-conversation/-conversation-type/index.html) |会话类型|
|targetId|String |会话 Id|
|callback |ResultCallback\<Boolean\>|回调接口|

#### 示例代码
```java
ConversationType conversationType = ConversationType.PRIVATE;
String targetId = "会话 Id";

 RongCoreClient.getInstance().clearTextMessageDraft(
                        conversationType,
                        targetId,
                        new IRongCoreCallback.ResultCallback<Boolean>() {
                            @Override
                            public void onSuccess(Boolean aBoolean) {
                                
                            }

                            @Override
                            public void onError(IRongCoreEnum.CoreErrorCode e) {
                                
                            }
                        });

```

