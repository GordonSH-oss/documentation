---
sidebar_position: 50
title: 好友管理
---

本文档旨在指导开发者如何使用融云即时通讯 Android IMLib SDK 实现添加好友、删除好友、查看好友列表、管理好友等功能。

:::tip
此功能从 5.12.0 版本开始支持。
:::

## 开通服务

信息托管服务已默认开通，您可以直接使用此功能。

## 好友事件监听

为了及时接收好友添加删除、好友申请状态、好友全部清理、好友信息变更多端回调变更通知，你需要在应用调用 IMLib SDK 的初始化之后，且在调用 IMLib SDK 的连接之前，设置订阅好友事件监听器 [FriendEventListener]。

您可以调用 `setFriendEventListener` 设置好友事件监听器，如果不想再接收好友事件，接口传 `null` 即可。

:::tip
信息托管服务中，好友操作产生的 SDK 通知回调也是一种状态通知行为，不管应用中是否实现 SDK 的事件监听，服务端都会对 SDK 进行状态同步，确保本地最新的好友关系状态，所以会计入消息的分发、下行数据统计中。
:::

#### 示例代码
```java
FriendEventListener friendEventListener = new FriendEventListener() {
    @Override
    public void onFriendAdd(DirectionType directionType, String userId, String name, String portraitUri, long operationTime) {
        // 好友添加回调
    }

    @Override
    public void onFriendDelete(DirectionType directionType, List<String> userIds, long operationTime) {
        // 好友删除回调
    }

    @Override
    public void onFriendApplicationStatusChanged(String userId, FriendApplicationType applicationType, FriendApplicationStatus status, DirectionType directionType, long operationTime, String extra) {
        // 好友申请状态回调事件
    }

    @Override
    public void onFriendCleared(long operationTime) {
        // 好友全部清理回调事件。注意：此操作只能由服务端发起
    }

    @Override
    public void onFriendInfoChangedSync(String userId, String remark, Map<String, String> extProfile, long operationTime) {
        // 好友信息变更多端回调事件
    }
};
RongCoreClient.getInstance().setFriendEventListener(friendEventListener);
```

## 好友在线状态与资料变更

IMLib SDK 会自动处理好友之间的在线状态和用户资料变更。  
如果您需要接收变更事件通知，您可以在 SDK 初始化之后，建立连接之前调用 `addSubscribeEventListener` 设置变更事件监听器。  

订阅事件的变更，需要根据 [SubscribeType] 订阅类型来处理对应的业务

- `FRIEND_ONLINE_STATUS(3)` 时代表好友在线状态；
- `FRIEND_USER_PROFILE(4)` 代表好友资料。

:::tip
您需要通过开发者后台开启“客户端好友资料变更通知”和“客户端好友在线状态变更通知”功能后，才能使用此功能，实时接收好友之间的在线状态和用户资料变更通知，通知行为也会计入单聊消息的分发、下行数据统计。
:::

#### 示例代码

```java
RongCoreClient.getInstance().addSubscribeEventListener(new OnSubscribeEventListener() {
    /**
     * @param subscribeEvents 订阅事件的列表，包含所有发生变化的事件。
     * 被订阅者发生状态变更时，SubscribeEvent.operationType 无值。
     * 订阅过期没有通知， 开发者需自行关注过期时间。
     * 注意：需要判断 SubscribeInfoEvent 的 SubscribeType， 等于 FRIEND_ONLINE_STATUS 代表好友在线状态，FRIEND_USER_PROFILE 代表好友资料
     */
    @Override
    public void onEventChange(List<SubscribeInfoEvent> subscribeEvents) {

    }

    /**
     * 标记订阅数据同步完成。 该方法在订阅数据成功同步到设备或系统后调用，用于执行后续处理。
     * 注意：需要判断 SubscribeType， 等于 FRIEND_ONLINE_STATUS 代表好友在线状态，FRIEND_USER_PROFILE 代表好友资料
     *
     * @param type 同步完成的类型。需要通过根据类型来判断具体是哪种业务同步完成。
     * @since 5.10.0
     */
    @Override
    public void onSubscriptionSyncCompleted(SubscribeEvent.SubscribeType type) {

    }

});
```

## 好友操作

### 添加好友

您可以通过调用 `addFriend` 接口根据用户 userId 将指定用户添加为好友，添加时可通过设置 extra 将好友申请的附加信息同步给被添加的用户。
用户 userId 的获取可以使用使用融云提供的[用户搜索]功能。

:::tip
一个用户最多可以添加 **3000** 个好友。
:::

#### 示例代码
```java
// 添加的好友用户Id
String userId = "user1";
// 添加双向好友
DirectionType directionType = DirectionType.Both;
// 发送好友请求时的附加信息，长度不超过 128 个字符
String extra = "请求添加好友";
RongCoreClient.getInstance().addFriend(userId, directionType, extra, new IRongCoreCallback.ResultCallback<IRongCoreEnum.CoreErrorCode>() {
    @Override
    public void onSuccess(IRongCoreEnum.CoreErrorCode processCode) {
        // 添加好友请求成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 添加好友请求失败
    }
});
```

### 设置加好友权限

您可以使用 `setFriendAddPermission` 设置当前用户的加好友权限，未设置时以 AppKey 默认的加好友权限为准。

#### 加好友权限说明

- AppKey 默认用户信息权限为 **需要用户同意添加好友**。
- 用户的权限设置将覆盖 AppKey 的权限设置，若用户未设定自己的权限，此时将采用 AppKey 的权限设置。

#### 加好友权限 `FriendAddPermission` 枚举介绍

| 枚举值          | 枚举说明 |
|--------------|------|
| Free         | 任何人直接添加好友 |
| NeedVerify         | 需要用户同意添加好友 |
| NoOneAllowed         | 不允许任何人添加好友 |

#### 示例代码
```java
// 通过 permission 参数设置当前加好友权限为 NeedVerify
FriendAddPermission permission = FriendAddPermission.NeedVerify;
RongCoreClient.getInstance().setFriendAddPermission(permission, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 设置加好友权限成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 设置加好友权限失败
    }
});
```

### 获取加好友权限

您可以使用 `getFriendAddPermission` 获取当前用户的加好友权限。

#### 示例代码
```java
RongCoreClient.getInstance().getFriendAddPermission(new IRongCoreCallback.ResultCallback<FriendAddPermission>() {
    @Override
    public void onSuccess(FriendAddPermission permission) {
        // 获取加好友权限成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 获取加好友权限失败
    }
});
```

#### 不同的权限设置，会产生以下两种流程

#### 场景1：添加好友不需要对方同意

1. 用户 A 和 B 调用 `setFriendListener` 设置好友事件监听。 
2. 用户 B 通过 `setFriendAddPermission` 设置加好友权限为 : 任何人直接添加好友（`Free`），见 [FriendAddPermission]。 
3. 用户 A 调用 `addFriend` 申请添加 B 为好友，回调 `onSuccess` 并且 `IRongCoreEnum.CoreErrorCode` 返回 `SUCCESS(0)`，表示直接加为好友，同时用户 A 和 B 都会收到好友添加回调 [onFriendAdd]。

#### 场景2：添加好友需要通过对方同意
1. 用户 A 和 B 调用 `setFriendListener` 设置好友事件监听。 
2. 用户 B 通过 `setFriendAddPermission` 设置加好友权限为 : 需要用户同意添加好友（`NeedVerify`），见[FriendAddPermission]。 
3. 用户 A 调用 `addFriend` 申请添加 B 为好友，回调 `onSuccess` 并且 `IRongCoreEnum.CoreErrorCode` 返回 `RC_FRIEND_NEED_ACCEPT(25461)`，表示需要等待用户 B 的同意，同时用户 A 和 B 都会收到好友申请状态回调 [onFriendApplicationStatusChanged]。
4. 用户 B 收到 [onFriendApplicationStatusChanged] 回调，参数 [FriendApplicationStatus] 为 `UnHandled` ，可以选择接受或者拒绝： 
   - 用户 B 调用 `acceptFriendApplication` 接受好友请求，双方都会收到 [onFriendAdd]回调，说明好友添加成功。 
   - 用户 B 调用 `refuseFriendApplication` 拒绝好友请求，双方都会收到 [onFriendApplicationStatusChanged] 回调，参数 [FriendApplicationStatus] 为 `Refused`。

### 解除好友

您可以使用 `deleteFriends` 批量解除好友关系。解除好友成功，双方都会收到好友删除回调 [onFriendDelete]。  

接口每次调用最多支持解除 **100** 个好友。  

#### 示例代码
```java
// 好友用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("user1");
userIds.add("user2");
userIds.add("user3");
// 解除双向好友
DirectionType directionType = DirectionType.Both;
RongCoreClient.getInstance().deleteFriends(userIds, directionType, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 解除好友成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 解除好友失败
    }
});
```

### 检查好友关系

您可以使用 `checkFriends` 检查好友关系，目前仅支持双向好友检查。

:::tip
一次最多检查 **20** 个用户。
:::

#### 示例代码
```java
// 好友用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("user1");
userIds.add("user2");
userIds.add("user3");
// 检查双向好友
DirectionType directionType = DirectionType.Both;
RongCoreClient.getInstance().checkFriends(userIds, directionType, new IRongCoreCallback.ResultCallback<List<FriendRelationInfo>>() {
    @Override
    public void onSuccess(List<FriendRelationInfo> friendRelationInfos) {
        // 检查成功结果回调
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 检查失败结果回调
    }
});
```

## 好友申请操作

### 分页获取好友请求列表

您可以使用 `getFriendApplications` 查看所有发送和接收的添加好友请求记录信息。

- 此接口不支持返回请求总数。
- 好友请求处理有效期为 7 天，融云服务端最多存储 7 天的请求数据，超过 7 天后需要重新发起请求。

:::tip
分页拉取说明：
1. 首次拉取时，[PagingQueryOption] 的 `pageToken` 无需设置（不设置、设置null、设置""，效果等同）。
2. 拉取第二页需要传入首次拉取返回结果 [PagingQueryResult] 类型中的 `pageToken`。
   - 如果不为 ""，则可以传入该值再次拉取，直至 `pageToken` 返回为 ""，表示全部拉取完成。 
   - 如果为 ""，表示没有下一页或已拉取完成，无需再次拉取。如传递 ""，将视为拉取首页数据。
:::

#### 示例代码
```java
{
    // ...
    // 分页拉取参数（设置null与设置""，效果等同）
    String pageToken = "";
    getFriendApplications(pageToken);
    // ...
}

public void getFriendApplications(String pageToken) {
    // 分页请求参数
    PagingQueryOption option = new PagingQueryOption();
    // 设置分页大小，取值范围为 [1~100]。
    option.setCount(20);
    // 分页拉取参数
    option.setPageToken(pageToken);
    // 按操作时间正序、倒序获取。true：正序；false：倒序。
    option.setOrder(false);
    // 查询发送 + 接收类型的好友请求
    FriendApplicationType[] types = new FriendApplicationType[]{FriendApplicationType.Sent, FriendApplicationType.Received};
    // 查询未处理 + 已处理 + 已拒绝类型的好友请求
    FriendApplicationStatus[] status = new FriendApplicationStatus[]{FriendApplicationStatus.UnHandled, FriendApplicationStatus.Accepted, FriendApplicationStatus.Refused};
    RongCoreClient.getInstance().getFriendApplications(option, types, status, new IRongCoreCallback.PageResultCallback<FriendApplicationInfo>() {
        @Override
        public void onSuccess(PagingQueryResult<FriendApplicationInfo> result) {
            if (!TextUtils.isEmpty(result.getPageToken())) {
                // 使用返回的 pageToken 拉取下一页
                getFriendApplications(result.getPageToken());
            } else {
                // 拉取结束
            }
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode e) {
            // 拉取失败
        }
    });
}
```

### 同意加为好友

在收到添加好友请求后，调用 `acceptFriendApplication` 接受好友请求。触发成功回调后，好友添加成功，同时双方会触发 [onFriendAdd] 回调。

同时好友申请列表中的好友请求状态 [FriendApplicationStatus] 变为已同意(`Accepted`)。

#### 示例代码
```java
// 好友用户ID
String userId = "user1";
RongCoreClient.getInstance().acceptFriendApplication(userId, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 同意加好友成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 同意加好友失败
    }
});
```

### 拒绝加为好友

在收到添加好友请求后，用户可以拒绝成为好友，调用 `refuseFriendApplication` 拒绝好友请求。

拒绝成功后，双方会触发 [onFriendApplicationStatusChanged] 回调，好友请求状态 [FriendApplicationStatus] 为已拒绝(`Refused`)。

#### 示例代码
```java
// 好友用户ID
String userId = "user1";
RongCoreClient.getInstance().refuseFriendApplication(userId, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 拒绝加好友成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 拒绝加好友失败
    }
});
```

## 好友信息的设置与获取

### 获取好友列表

您可以使用 `getFriends` 获取当前用户的好友列表。

#### 示例代码
```java
// 查询双向好友列表
QueryFriendsDirectionType type = QueryFriendsDirectionType.Both;
RongCoreClient.getInstance().getFriends(type, new IRongCoreCallback.ResultCallback<List<FriendInfo>>() {
    @Override
    public void onSuccess(List<FriendInfo> friendInfos) {
        // 查询好友列表成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询好友列表失败
    }
});
```

### 好友信息设置

您可以使用 `setFriendInfo` 设置好友的备注名和扩展信息。

:::tip
- 扩展信息的 key 需要通过融云控制台 [好友自定义属性] 设置后才能使用，否则返回设置失败，最多可设置 **10** 个扩展信息 key。
- 好友信息修改成功后，本用户登录的其他终端会收到好友信息变更多端回调 [onFriendInfoChangedSync]。
- 默认情况下，设置的资料信息不支持审核。如需审核功能，请前往融云控制台 **IM 服务** > **功能配置** > **安全&审核** > **IM 信息托管审核**，开启并设置需要审核的内容。

:::

#### 示例代码
```java
// 好友用户ID
String userId = "user1";
// 好友备注名，最多为 64 个字符。不传或为空时清除备注名
String remark = "user1的好友备注名";
// 扩展信息，默认最多可设置 10 个扩展信息。
Map<String, String> extProfile = new HashMap<>();
extProfile.put("ext_key_1", "ext_value_1");
extProfile.put("ext_key_2", "ext_value_2");
RongCoreClient.getInstance().setFriendInfo(userId, remark, extProfile, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 设置好友信息成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 设置好友信息失败
    }
});
```

### 根据用户 ID 获取好友信息

您可以使用 `getFriendsInfo` 根据用户 ID 获取好友信息。  
该接口支持批量获取，您可以一次传入多个 `userId` 获取多个好友信息，最多不超过 **100** 个。

#### 示例代码
```java
// 好友用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("user1");
userIds.add("user2");
userIds.add("user3");
RongCoreClient.getInstance().getFriendsInfo(userIds, new IRongCoreCallback.ResultCallback<List<FriendInfo>>() {
    @Override
    public void onSuccess(List<FriendInfo> friendInfos) {
        // 查询好友成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询好友失败
    }
});
```

### 根据好友昵称搜索好友信息

您可以使用 `searchFriendsInfo` 根据好友昵称搜索好友信息。

搜索时默认先搜好友备注名 `remark`，再搜索好友名称 `name`。只要其中一个字段匹配成功，即返回搜索结果。

#### 示例代码
```java
// 用户昵称关键字，不能为空最长不超过 64 个字符
String name = "name";
RongCoreClient.getInstance().searchFriendsInfo(name, new IRongCoreCallback.ResultCallback<List<FriendInfo>>() {
    @Override
    public void onSuccess(List<FriendInfo> friendInfos) {
        // 查询好友列表成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 查询好友列表失败
    }
});
```
| AppKey 权限                    | 用户权限                             | 结果            |
|------------------------------|----------------------------------|---------------|
| Free、NeedVerify、NoOneAllowed | 未设置                              | 以 AppKey 权限设置为准 |
| Free、NeedVerify、NoOneAllowed | 设置为（Free、NeedVerify、NoOneAllowed） | 以用户权限设置为准     |

<!-- links -->
[FriendAddPermission]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-friend-add-permission/index.html
[FriendApplicationStatus]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-friend-application-status/index.html
[FriendEventListener]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-friend-event-listener/index.html
[onFriendAdd]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-friend-event-listener/on-friend-add.html
[onFriendDelete]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-friend-event-listener/on-friend-delete.html
[onFriendApplicationStatusChanged]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-friend-event-listener/on-friend-application-status-changed.html
[onFriendInfoChangedSync]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-friend-event-listener/on-friend-info-changed-sync.html
[SubscribeType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-subscribe-event/-subscribe-type/index.html
[PagingQueryOption]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-option/index.html
[PagingQueryResult]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-result/index.html

[用户搜索]: ./user_profiles#搜索用户
[好友自定义属性]: https://console.rongcloud.cn/agile/im/messageTrusteeship/customFriend


