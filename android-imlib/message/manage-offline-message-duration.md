---
title: 管理离线消息存储配置
sidebar_position: 130
---

# 管理离线消息存储配置

即时通讯业务支持修改 App 级别与用户级别的离线消息配置。

:::tip

 离线消息配置仅适用于单聊、群聊。聊天室、超级群因业务特性不支持离线消息，因此无离线消息配置。
:::


## 了解离线消息

离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>是指当用户不在线时收到的消息。融云服务端会自动为用户保留离线期间接收的消息，默认的离线消息保留时长为  7 天。7 天内客户端如果上线，服务端会直接将离线消息发送到该接收端。如果 7 天内客户端都没有上线，服务端将抛弃掉过期的消息。

即时通讯业务下并非所有会话类型都支持离线消息：

- **支持离线消息**：单聊、群聊、系统消息
- **不支持离线消息**：聊天室、超级群

## App 级别离线消息配置

您可以修改如下 App 级别的离线消息存储设置：
- **离线消息存储时长**。您可以在[融云控制台](https://console.rongcloud.cn/agile/im/service/config#%E5%85%A8%E5%B1%80%E6%B6%88%E6%81%AF)，在 **IM 服务**>**功能配置**>**全局消息**>**离线消息存储时长**，来修改存储时长。目标用户未在线时，该用户接收的消息会保存到离线消息中，默认存储 7 天，下次登录时会获取到离线消息，如果需要调整可在此进行设置，设置范围为 1 ~ 7 天。改设置仅针对单聊、群聊、系统消息会话类型有效。
- **群组离线消息存储天数**。当目标用户未在线时，其接收的群组消息会存储到离线消息中，默认存储时长为 7 天，用户在下次登录时可获取这些消息。如未配置此项，则群组的离线消息存储时长由“设置离线消息存储时长”控制。如已配置此项，则群组消息的离线存储天数将依据该配置。
- **群组离线消息存储数量**。默认存储 7 天内的所有群消息，可以提工单修改。配置修改将影响 App 下所有群聊会话。

## 用户级别离线消息配置

:::tip

 设置、获取用户的离线消息存储时长功能均要求已开通**用户级别功能设置**。如需开通，请提交工单。
:::


即时通讯业务支持用户级别的离线消息配置，仅支持修改离线消息存储时长。未修改的情况下，用户的离线消息存储时长为 7 天。设置范围为 1 - 7 天。

App Key 开通**用户级别功能设置**功能后，客户端 SDK 支持修改当前登录用户的离线消息存储时长。

### 设置用户的离线消息存储时长

设置当前用户的离线消息存储时长，以天为单位。

#### 接口

```java
RongIMClient.getInstance().setOfflineMessageDuration(duration, callback);
```

#### 参数说明

|        参数        |   类型   |          说明           |
| :-----------------| :-------| :---------------------- |
| duration         | int    | 离线消息存储时长，范围为 1 - 7 天。             |
| callback           | RongIMClient.ResultCallback\<Long\> | 回调接口。 |


#### 示例代码

```java
int duration = 3;

RongIMClient.getInstance().setOfflineMessageDuration(duration, new RongIMClient.ResultCallback<Long>() {
            @Override
            public void onSuccess(Long aLong) {

            }

            @Override
            public void onError(RongIMClient.ErrorCode e) {

            }
        });
```


### 获取用户的离线消息存储时长

获取当前用户的离线消息存储时长，以天为单位。

```java
RongIMClient.getInstance().getOfflineMessageDuration(new RongIMClient.ResultCallback<String>() {
            @Override
            public void onSuccess(String s) {

            }

            @Override
            public void onError(RongIMClient.ErrorCode e) {

            }
});
```

|        参数        |   类型   |          说明           |
| :----------------- | :-------| :---------------------- |
| callback           | RongIMClient.ResultCallback\<String\>  | 回调接口。 |

