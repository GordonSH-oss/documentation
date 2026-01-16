---
sidebar_position: 60
title: 群成员管理
---

本文档旨在指导开发者如何使用融云即时通讯 IMLib SDK 实现设置或查询群组成员资料，添加或删除群管理员等功能。

:::tip
此功能从 5.12.0 版本开始支持。
:::

## 开通服务

信息托管服务已默认开通，您可以直接使用此功能。

## 群成员

可以查询指定群的群成员，也可以设置群成员信息。

### 分页获取群成员信息

您可以使用 [getGroupMembersByRole] 按群成员角色分页获取群成员信息。  
此接口支持返回本次查询条件的总数，见 [PagingQueryOption] 的`getTotalCount()`。

群成员角色 `GroupMemberRole` 枚举介绍：

| 枚举值 | 群成员角色                     |
| --------------------- |---------------------------|
| Undef | 未定义角色（使用此枚举查询代表查询全部类型群成员） |
| Normal | 普通群成员                     |
| Manager | 管理员                       |
| Owner | 群主                        |


:::tip
分页拉取说明：
1. 首次拉取时，[PagingQueryOption] 的 `pageToken` 无需设置（不设置、设置 null、设置 ""，效果等同）。
2. 拉取第二页需要传入首次拉取返回结果 [PagingQueryResult] 类型中的 `pageToken`。
    - 如果不为 ""，则可以传入该值再次拉取，直至 `pageToken` 返回为 ""，表示全部拉取完成。
    - 如果为 ""，表示没有下一页或已拉取完成，无需再次拉取。如传递 ""，将视为拉取首页数据。
:::

#### 代码示例
```java
{
    //...
    // 分页拉取参数（设置null 与设置 ""，效果等同）
    String pageToken = "";
    getGroupMembersByRole(pageToken);
    //...
}

public void getGroupMembersByRole(String pageToken) {
    // 群 Id
    String groupId = "groupId";
    // 通过 role 参数指定拉取全部群成员角色类型的群成员信息
    GroupMemberRole role = GroupMemberRole.Undef;
    // 分页请求参数
    PagingQueryOption option = new PagingQueryOption();
    // 设置分页大小，取值范围为 [1~100]。
    option.setCount(20);
    // 分页拉取参数
    option.setPageToken(pageToken);
    // 按加入群组时间正序、倒序获取。true：正序；false：倒序。
    option.setOrder(false);
    RongCoreClient.getInstance().getGroupMembersByRole(groupId, role, option, new IRongCoreCallback.PageResultCallback<GroupMemberInfo>() {
        @Override
        public void onSuccess(PagingQueryResult<GroupMemberInfo> result) {
            if (!TextUtils.isEmpty(result.getPageToken())) {
                // 使用返回的 pageToken 拉取下一页
                getGroupMembersByRole(result.getPageToken());
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

### 获取群成员信息

您可以使用 [getGroupMembers] 获取指定群指定用户的群成员信息。  
该接口支持批量获取，您可以一次传入多个 `userId` 获取多个群成员信息，最多不超过 **100** 个。

:::tip
若当前用户未加入指定群组，则无法获取群成员信息。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 设置查询用户ID列表
List<String> userIdList = new ArrayList<>();
userIdList.add("userId1");
userIdList.add("userId2");
userIdList.add("userId3");
RongCoreClient.getInstance().getGroupMembers(groupId, userIdList, new IRongCoreCallback.ResultCallback<List<GroupMemberInfo>>() {
    @Override
    public void onSuccess(List<GroupMemberInfo> groupMemberInfos) {
        // 拉取成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 拉取失败
    }
});
```

### 根据昵称搜索群成员信息

您可以使用 [searchGroupMembers] 分页搜索本地群组中指定群的群成员信息。  

搜索时优先匹配群成员昵称 `nickname`，再匹配群成员用户名 `name`。只要其中一个字段匹配成功，即返回搜索结果。

此接口不支持返回所有群成员总数。

:::tip
分页拉取说明：
1. 首次拉取时，[PagingQueryOption] 的 `pageToken` 无需设置（不设置、设置 null、设置 ""，效果等同）。
2. 拉取第二页需要传入首次拉取返回结果 [PagingQueryResult] 类型中的 `pageToken`。
    - 如果不为 ""，则可以传入该值再次拉取，直至 `pageToken` 返回为 ""，表示全部拉取完成。
    - 如果为 ""，表示没有下一页或已拉取完成，无需再次拉取。如传递 ""，将视为拉取首页数据。
:::

#### 代码示例
```java
{
    //...
    // 分页拉取参数（设置null 与设置 ""，效果等同）
    String pageToken = "";
    searchGroupMembers(pageToken);
    //...
}

public void searchGroupMembers(String pageToken) {
    // 群Id
    String groupId = "groupId";
    // 群成员昵称，不能为空最长不超过 64 个字符
    String name = "name";
    // 分页请求参数
    PagingQueryOption option = new PagingQueryOption();
    // 设置分页大小，取值范围为 [1~200]。
    option.setCount(20);
    // 分页拉取参数
    option.setPageToken(pageToken);
    // 按加入群组时间正序、倒序获取。true：正序；false：倒序。
    option.setOrder(false);
    RongCoreClient.getInstance().searchGroupMembers(groupId, name, option, new IRongCoreCallback.PageResultCallback<GroupMemberInfo>() {
        @Override
        public void onSuccess(PagingQueryResult<GroupMemberInfo> result) {
            if (!TextUtils.isEmpty(result.getPageToken())) {
                // 使用返回的 pageToken 拉取下一页
                searchGroupMembers(result.getPageToken());
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

### 设置群成员信息

您可以使用 [setGroupMemberInfo] 设置指定群的群成员信息。

接口调用成功后，群内所有人会收到群成员信息变更事件 `onGroupMemberInfoChanged` 回调。

群成员资料更新权限 `memberInfoEditPermission` (`GroupMemberInfoEditPermission`)，决定是否可以修改群成员信息，如果没有权限设置，调用将会返回错误码。

 `GroupMemberInfoEditPermission` 枚举介绍：  

| 枚举值 | 群成员角色                     |
| ------------------------- |---------------------------|
| OwnerOrManagerOrSelf | 群主、管理员和当前用户，可以修改在群组中的成员信息 |
| OwnerOrSelf | 群主和当前用户，可以修改在群组中的成员信息                     |
| Self | 只有当前用户可以修改，自己的群成员资料信息                       |

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 群成员用户Id
String userId = "userId1";
// 群成员昵称
String nickname = "nickname";
// 群成员附加信息
String extra = "群成员附加信息";
RongCoreClient.getInstance().setGroupMemberInfo(groupId, userId, nickname, extra, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 操作成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 操作失败
    }
});
```

## 群管理员

可以添加或者移除群管理员。

### 添加群管理员

您可以使用 [addGroupManagers] 添加群管理员。

接口调用成功后，群内所有人会收到群组操作事件 `onGroupOperation` 回调，操作类型为 `AddManager`。

:::tip
- 只有群主可以添加群管理员。
- 要设置的成员必须在此群内，且群主不能被设置为管理员。
- 一个群组的管理员数量上限为 **10** 个。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 管理员用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("userId1");
userIds.add("userId2");
userIds.add("userId3");
RongCoreClient.getInstance().addGroupManagers(groupId, userIds, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 操作成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 操作失败
    }
});
```

### 移除群管理员

您可以使用 [removeGroupManagers] 移除群管理员。  
该接口支持批量移除，您可以一次传入多个 `userId` 移除多个群管理员，最多不超过 **10** 个。

接口调用成功后，群内所有人会收到群组操作事件 `onGroupOperation` 回调，操作类型为 `RemoveManager`。

:::tip
- 只有群主可以移除群管理员。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 管理员用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("userId1");
userIds.add("userId2");
userIds.add("userId3");
RongCoreClient.getInstance().removeGroupManagers(groupId, userIds, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 操作成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 操作失败
    }
});
```

## 特别关注群成员

可以添加、移除或者查询群特别关注成员。

#### 功能介绍
- 特别关注群成员数量上限为 **200** 个。
- 特别关注用户在该群发送的消息不受会话免打扰的设置影响，可正常发送通知提醒。
- 针对特别关注用户发送的消息，推送优先级说明如下：
  - 应用级别免打扰状态为不发推送 > 消息体中标识该消息为静音消息 > 发送消息用户为特别关注用户 > 会话免打扰设置。
- 免打扰相关说明详见 [免打扰功能概述](../conversation/do-not-disturb-about)。
- 特别关注群成员设置或移除成功后，该用户登录的其他终端会收到好友信息变更多端事件回调 `onGroupFollowsChangedSync`。

### 添加特别关注群成员

您可以使用 [addGroupFollows] 添加特别关注群成员。  
该接口支持批量添加，您可以一次传入多个 `userId` 添加多个特别关注群成员，最多不超过**100**个。

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 群成员用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("userId1");
userIds.add("userId2");
userIds.add("userId3");
RongCoreClient.getInstance().addGroupFollows(groupId, userIds, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 操作成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 操作失败
    }
});
```

### 移除特别关注群成员

您可以使用 [removeGroupFollows] 移除特别关注群成员。  
该接口支持批量移除，您可以一次传入多个 `userId` 移除多个特别关注群成员，最多不超过**100**个。

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 群成员用户Id列表
List<String> userIds = new ArrayList<>();
userIds.add("userId1");
userIds.add("userId2");
userIds.add("userId3");
RongCoreClient.getInstance().removeGroupFollows(groupId, userIds, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 操作成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 操作失败
    }
});
```

### 查询特别关注群成员

您可以使用 [getGroupFollows] 查询特别关注群成员。

#### 代码示例
```java
// 群Id
String groupId = "groupId";
RongCoreClient.getInstance().getGroupFollows(groupId, new IRongCoreCallback.ResultCallback<List<FollowInfo>>() {
    @Override
    public void onSuccess(List<FollowInfo> followInfos) {
        // 拉取成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 拉取失败
    }
});
```

<!-- 链接区域 -->
[getGroupFollows]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-group-follows.html
[removeGroupFollows]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/remove-group-follows.html
[addGroupFollows]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/add-group-follows.html
[removeGroupManagers]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/remove-group-managers.html
[addGroupManagers]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html#-46806077%2FFunctions%2F1814687565
[setGroupMemberInfo]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/set-group-member-info.html
[searchGroupMembers]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/search-group-members.html
[getGroupMembers]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-group-members.html
[getGroupMembersByRole]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-group-members-by-role.html
[PagingQueryOption]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-option/index.html
[PagingQueryResult]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-result/index.html
