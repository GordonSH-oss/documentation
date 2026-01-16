---
title: 黑名单管理
sidebar_position: 10
---

# 黑名单管理

融云的黑名单功能是指如果您将其他用户的 userId 加入黑名单之后，将不再收到此用户发送给您的任何单聊消息。您需要注意的是：

- 您将其他用户加入黑名单的操作为单向操作，例如：您通过调用融云 SDK 对应的方法拉黑了用户 A，那么用户 A 就无法给您发送单聊消息，并且用户 A 会收到 405 的错误码提示。但您仍然可以向用户 A 发送消息，用户 A 也可以正常接收。
- 每个用户的黑名单总人数存在上限，具体的限制与计费套餐有关。IM 旗舰版与 IM 尊享版上限为 3000 人，其他套餐详见[功能对照表](https://help.rongcloud.cn/t/topic/121)中的**服务限制**。
- 如果您调用服务端 API 发送单聊消息，消息能否送达默认是不受黑名单限制。如需服务端 API 发送单聊消息也要受到黑名单的限制，请您在调用 API 时设置参数 `verifyBlacklist` 为 `1`。

## 加入黑名单 {#add}

您可以通过调用 `addToBlacklist` 方法将指定用户（`userId`）加入到您的黑名单中。添加成功后，您仍然可以向被拉黑的用户发送消息，但被拉黑的用户无法给您发送单聊消息。

#### 接口

```java
RongIMClient.getInstance().addToBlacklist(userId, callback);
```

#### 参数说明

|        参数        |   类型   |          说明           |
| :----------------- | :------- | :---------------------- |
| userId         | String    | 用户 ID |
|callback |OperationCallback|回调接口|

## 移出黑名单  {#remove}

您可以通过调用 `removeFromBlacklist` 方法将指定用户（`userId`）从您的黑名单中移出。

#### 接口

```java
RongIMClient.getInstance().removeFromBlacklist(userId, callback);
```

#### 参数说明

|        参数        |   类型   |          说明           |
| :----------------- | :------- | :---------------------- |
| userId         | String    | 用户 ID |
|callback |OperationCallback|回调接口|

## 查询用户是否在黑名单中  {#lookup}

您可以通过调用 `getBlacklistStatus` 方法，查询指定的用户 ID（`userId`）是否在您的黑名单中。

#### 示例代码

```java
RongIMClient.getInstance().getBlacklistStatus(userId,
        new ResultCallback<BlacklistStatus>() {
            @Override
            public void onSuccess(BlacklistStatus blacklistStatus) {
            }

            @Override
            public void onError(ErrorCode e) {
            }
        });
```

`BlacklistStatus` 枚举值：

- 在黑名单中：`IN_BLACK_LIST(0)`
- 不在黑名单中：`NOT_IN_BLACK_LIST(1)`

## 获取黑名单列表 {#get}

您可以通过调用 `getBlacklist` 方法获取您的黑名单列表。

#### 示例代码

```java
RongIMClient.getInstance()
        .getBlacklist(
                new GetBlacklistCallback() {
                    @Override
                    public void onSuccess(String[] strings) {
                    }

                    @Override
                    public void onError(ErrorCode e) {
                    }
                });
```
