---
title: 用户信息托管
sidebar_position: 30
---

本文档旨在指导开发者如何在融云即时通讯 Android 客户端 SDK 中实现用户信息订阅、查询和监听，同时支持用户信息与权限的修改、查询。
通过本文档， Android 开发者将了解如何获取和跟踪用户信息，以及如何在用户信息变更、订阅状态变更时接收通知。

:::tip
此功能在 5.10.0 版本开始支持。
:::


## 开通服务

信息托管服务已默认开通，您可以直接使用此功能。

## 管理用户信息

可以修改或查询自已的用户信息，批量查询指定多个用户的用户信息。

### 设置用户信息

用户在应用中使用 `updateMyUserProfile` 可以修改自已的用户信息，您需要设置的相关用户信息可以通过创建 `RCUserProfile` 对象来配置相关属性。

:::tip
默认情况下，您设置的资料信息不会进行审核。如需要开通审核功能，请前往融云控制台 **IM 服务** > **功能配置** > **安全&审核** > **IM 信息托管审核**，开启并设置需要审核的内容。
:::

#### 接口

```java
RongCoreClient.getInstance().updateMyUserProfile(userProfile, callback);
```

#### 参数说明

下表描述 [UserProfile] 类的属性，也可参考 API 文档。

| 属性名            | 类型     | 描述                                                                                                                                                                             |
|:---------------|:-------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| name           | String | 昵称，长度不超过 32 个字符。                                                                                                                                                               |
| portraitUri    | String | 头像地址，长度不超过 128 个字符。                                                                                                                                                            |
| uniqueId       | String | 用户应用号，支持大小写字母、数字，长度不超过 32 个字符。**请注意** SDK不支持设置此字段。                                                                                                                             |
| email          | String | Email，长度不超过 128 个字符。                                                                                                                                                           |
| birthday       | String | 生日，长度不超过 32 个字符                                                                                                                                                                |
| gender         | int    | 性别，未知 0 、男 1、女 2。                                                                                                                                                              |
| location       | String | 所在地，长度不超过 32 个字符。                                                                                                                                                              |
| role           | int    | 角色，支持 0~100 以内数字。                                                                                                                                                              |
| level          | int    | 级别，支持 0~100 以内数字。                                                                                                                                                              |
| userExtProfile | long   | 自定义扩展信息，最多可以设置 20 个用户信息（以 Key、Value 方式设置，扩展用户信息通过开发者后台进行设置）<ol><li>Key：支持大小写字母、数字，长度不超过 32 个字符，需要保障 AppKey 下唯一。Key 不存在时，设置不成功返回错识提示。</li><li>Value：字符串，不超过 256 个字符。</li></ol>  |


#### 示例代码

```java
// 更新自己的用户信息
UserProfile userProfile = new UserProfile();
RongCoreClient.getInstance().updateMyUserProfile(userProfile, new IRongCoreCallback.UpdateUserProfileCallback() {
    @Override
    public void onSuccess() {
        // 更新成功
    }

    @Override
    public void onError(int errorCode, String errorKey) {
        // 更新失败
    }
});
```

### 获取当前用户信息

您可以使用 `getMyUserProfile` 方法获取当前用户信息。

#### 示例代码

```java
RongCoreClient.getInstance().getMyUserProfile(new IRongCoreCallback.ResultCallback<UserProfile>() {
    @Override
    public void onSuccess(UserProfile userProfile) {
        // 查询成功，返回自己的用户信息。
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询失败
    }
});
```

### 批量获取用户信息

您可以使用 `getUserProfiles` 方法，通过传入指定用户的 userId 来查询指定用户的用户信息。

:::tip
- 单次最多查询 20 个用户的用户信息。
:::

#### 示例代码

```java
// 设置查询用户ID列表
List<String> userIdList = new ArrayList<>();
userIdList.add("user1");
userIdList.add("user2");
userIdList.add("user3");
RongCoreClient.getInstance().getUserProfiles(userIdList, new IRongCoreCallback.ResultCallback<List<UserProfile>>() {
    @Override
    public void onSuccess(List<UserProfile> userProfiles) {
        // 查询成功，返回查询用户信息。
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询失败
    }
});
```

## 管理用户权限

融云 IMLib SDK 提供了对用户级别用户权限进行设置和获取接口，通过接口设置 `UserProfileVisibility` 枚举来设置不同的用户权限。

[UserProfileVisibility] 枚举介绍

| 枚举值 | 用户权限 |
| ------------------------- |-------------------------------|
| UserProfileVisibility.NotSet | 未设置：以 AppKey 权限设置为准，默认是此状态。   |
| UserProfileVisibility.Invisible | 都不可见：任何人都不能搜索到我的用户信息，名称、头像除外。 |
| UserProfileVisibility.Everyone | 所有人：应用中任何用户都可以查看到我的用户信息。      |
| UserProfileVisibility.FriendVisible | 仅好友可见：仅我的好友列表中用户可以查看到我的用户信息。      |

### 用户权限说明

AppKey 下默认用户信息访问权限为 **都不可见**，用户级别默认用户信息访问权限为 **未设置**。当您都未设置也就是二者访问权限都为默认值时， 您仅可查看他人的用户名和头像信息。

下面列举了 `AppKey 级` 权限与 `用户级` 权限综合说明：

| AppKey 级权限                | 用户级权限                          | 结果 |
| ------------------------- |-------------------------------|----|
| 都不可见、仅好友可见、所有人可见 | 未设置   | 以 AppKey 级权限设置为准   |
| 都不可见、仅好友可见、所有人可见 | 设置为（都不可见、仅好友可见、所有人可见） |  以用户级权限设置为准  |

### 设置用户权限

您可以使用 `updateMyUserProfileVisibility` 方法设置您自己的用户信息访问权限。

#### 示例代码

```java
// 用户信息权限
UserProfileVisibility visibility = UserProfileVisibility.Everyone;
RongCoreClient.getInstance().updateMyUserProfileVisibility(visibility, new IRongCoreCallback.ResultCallback<Boolean>() {
    @Override
    public void onSuccess(Boolean result) {
        // 设置成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 设置失败
    }
});
```

### 获取用户权限

您可以使用 `getMyUserProfileVisibility` 方法获取您自己的用户信息访问权限。

#### 示例代码

```java
// 获取自己的用户权限
RongCoreClient.getInstance().getMyUserProfileVisibility(new IRongCoreCallback.ResultCallback<UserProfileVisibility>() {
    @Override
    public void onSuccess(UserProfileVisibility userProfileVisibility) {
        // 获取成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 获取失败
    }
});
```

## 搜索用户

### 按用户应用号精确搜索

您可以使用 `searchUserProfileByUniqueId` 方法通过用户应用号搜索用户：

#### 示例代码

```java
// 使用应用号查询用户信息
String uniqueId = "uniqueId";
RongCoreClient.getInstance().searchUserProfileByUniqueId(uniqueId, new IRongCoreCallback.ResultCallback<UserProfile>() {
    @Override
    public void onSuccess(UserProfile userProfile) {
        // 查询成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询失败
    }
});
```

## 用户信息&权限变更订阅

### 监听订阅事件

为了及时接收订阅事件的变更通知，你需要在应用调用 IMLib SDK 的初始化之后，且在调用 IMLib SDK 的连接之前设置订阅监听器。

订阅事件的变更，需要根据 `SubscribeType` 订阅类型来处理对应的业务，等于 `USER_PROFILE` 时代表用户信息托管。

:::tip
- 从5.10.0版本开始，`SubscribeType` 包含了在线状态订阅 `ONLINE_STATUS(1)`、用户信息托管 `USER_PROFILE(2)` 这2种类型。
- 从5.12.0版本开始，`SubscribeType` 新增了好友在线状态订阅 `FRIEND_ONLINE_STATUS(3)`、好友用户信息托管 `FRIEND_USER_PROFILE(4)` 这2种类型。
:::

```java
RongCoreClient.getInstance().addSubscribeEventListener(new OnSubscribeEventListener() {
    /**
     * @param subscribeEvents 订阅事件的列表，包含所有发生变化的事件。
     * 被订阅者发生状态变更时，SubscribeEvent.operationType 无值。
     * 订阅过期没有通知， 开发者需自行关注过期时间。
     * 注意：需要判断 SubscribeInfoEvent 的 SubscribeType， 等于 USER_PROFILE 时代表用户信息托管
     */
    @Override
    public void onEventChange(List<SubscribeInfoEvent> subscribeEvents) {

    }

    /**
     * 标记订阅数据同步完成。 该方法在订阅数据成功同步到设备或系统后调用，用于执行后续处理。
     * 注意：需要判断 SubscribeType， 等于 USER_PROFILE 时代表用户信息托管
     *
     * @param type 同步完成的类型。需要通过根据类型来判断具体是哪种业务同步完成。
     * @since 5.10.0
     */
    @Override
    public void onSubscriptionSyncCompleted(SubscribeEvent.SubscribeType type) {

    }

    /**
     * 当用户在其他设备上的订阅信息发生变更时调用此方法。
     * 这可以用于更新当前设备上的用户状态，确保订阅信息的一致性。
     * 注意：需要判断 SubscribeInfoEvent 的 SubscribeType， 等于 USER_PROFILE 时代表用户信息托管
     * @param subscribeEvents 订阅事件的列表
     */
    @Override
    public void onSubscriptionChangedOnOtherDevices(List<SubscribeEvent> subscribeEvents) {

    }
});
```

### 订阅用户信息&权限变更

当您需要跟踪其他用户信息和权限的变更时，可以使用 `subscribeEvent` 方法订阅用户信息变更事件。

1. 您需要创建订阅请求对象 `SubscribeEventRequest`，设置订阅类型为用户信息与权限变更状态 `SubscribeEvent.SubscribeType.USER_PROFILE`， 并设置好需要订阅的所有用户的 userId 列表和订阅时长。需要注意您一次订阅的用户的上限为 200 个，订阅的用户数最多为 1000 个，同时您最多可以被 5000 个用户订阅。

2. 设置订阅时间：定义一个整数值,该值表示订阅的持续时间,订阅时长的范围为 60 秒到 2592000 秒。

**示例代码**

```java
//设置订阅类型。
SubscribeEvent.SubscribeType type= SubscribeEvent.SubscribeType.USER_PROFILE;
//设置订阅时间，取值范围为[60,2592000]（单位:秒）。
int expiry=180000;
//订阅用户userId，即单聊的 targetId (一次最多订阅 200 个)。
List<String> userList=new ArrayList<>();
userList.add("user1");
userList.add("user2");
userList.add("user3");
SubscribeEventRequest request = new SubscribeEventRequest(type,expiry,userList);
RongCoreClient.getInstance().subscribeEvent(request, new IRongCoreCallback.SubscribeEventCallback<List<String>>() {
    @Override
    public void onSuccess() {
        //订阅成功。
    }

    @Override
    public void onError(int errorCode, List<String> strings) {
        //订阅失败。
    }
});
```

### 取消用户信息&权限订阅

1. 当您不再需要跟踪已被订阅用户的用户信息与权限变更时,您需要创建订阅请求对象 `SubscribeEventRequest`，设置订阅类型为订阅用户信息与权限变更状态 `SubscribeEvent.SubscribeType.USER_PROFILE`，并设置好需要取消订阅用户信息与权限变更的所有用户的 userId 列表，需要注意您一次取消订阅的用户的上限为 200 个。

#### 示例代码

```java
//设置取消订阅的类型。
SubscribeEvent.SubscribeType type= SubscribeEvent.SubscribeType.USER_PROFILE;
//取消订阅用户userId，即单聊的 targetId，一次最多取消订阅 200 个。
List<String> userList=new ArrayList<>();
userList.add("user1");
userList.add("user2");
userList.add("user3");
SubscribeEventRequest request = new SubscribeEventRequest(type,userList);
RongCoreClient.getInstance().unSubscribeEvent(request, new IRongCoreCallback.SubscribeEventCallback<List<String>>() {
    @Override
    public void onSuccess() {
        //取消订阅成功。
    }

    @Override
    public void onError(int errorCode, List<String> strings) {
        //取消订阅失败。
    }
});
```

### 指定用户查询用户信息托管的订阅状态信息

您可以使用 `querySubscribeEvent` 方法查询指定用户和订阅类型的状态信息。一次最多查询200个用户的订阅状态信息。

#### 示例代码

```java
//设置查询类型。
SubscribeEvent.SubscribeType type= SubscribeEvent.SubscribeType.USER_PROFILE;
//查询用户在线状态。
List<String> userList=new ArrayList<>();
userList.add("user1");
userList.add("user2");
userList.add("user3");
SubscribeEventRequest request = new SubscribeEventRequest(type,userList);
RongCoreClient.getInstance().querySubscribeEvent(request, new IRongCoreCallback.ResultCallback<List<SubscribeInfoEvent>>() {
    @Override
    public void onSuccess(List<SubscribeInfoEvent> subscribeInfoEvents) {
        //查询成功，返回查询用户信息。
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        //查询失败。
    }
});
```

### 分页查询已订阅用户信息托管的用户的状态信息

如果您需要分页获取已订阅的所有事件状态信息，可以使用 `querySubscribeEvent` 方法，并指定分页大小和起始索引。

- Parameter pageSize: 分页大小 [1~200]。
- Parameter startIndex: 第一页传 0， 下一页取返回所有数据的数组数量（比如 pageSize = 20，第二页传 20，第三页传40）。

#### 示例代码

```java
//查询类型
SubscribeEvent.SubscribeType type= SubscribeEvent.SubscribeType.USER_PROFILE;
SubscribeEventRequest request = new SubscribeEventRequest(type);
//分页大小，取值范围为[1,200]。
int pageSize=20;
//分页起始索引，第一页传 0， 下一页取返回所有数据的数组数量，比如 pageSize = 20，则第二页传 20，第三页传 40。
int startIndex=0;
RongCoreClient.getInstance().querySubscribeEvent(request,pageSize,startIndex, new IRongCoreCallback.ResultCallback<List<SubscribeInfoEvent>>() {
    @Override
    public void onSuccess(List<SubscribeInfoEvent> subscribeInfoEvents) {
        //查询成功，返回查询用户信息。
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        //查询失败。
    }
});
```




<!-- 链接区域 -->
[UserProfile]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-user-profile/index.html
[UserProfileVisibility]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-user-profile-visibility/index.html?query=public%20enum%20UserProfileVisibility
