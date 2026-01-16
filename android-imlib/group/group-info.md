---
sidebar_position: 50
title: 群查询
---

本文档旨在指导开发者如何使用融云即时通讯 IMLib SDK 实现获取本人已加入的群组、获取指定群组资料等功能。

## 开通服务

信息托管服务已默认开通，您可以直接使用此功能。

## 群查询

可查询或搜索我加入的群组。


### 获取群组资料

您可以调用 [getGroupsInfo] 方法获取群组资料。

- 该方法优先本地查找，当用户不在群组中时，本地不存在或者本地群组信息缓存超过 10 分钟的才会从服务端拉取最新的群组信息。
- 单次调用最多支持获取 20 个群组资料。

#### 代码示例
```java
// 群Id列表
List<String> groupIds = new ArrayList<>();
groupIds.add("groupId1");
groupIds.add("groupId2");
groupIds.add("groupId3");
RongCoreClient.getInstance().getGroupsInfo(groupIds, new IRongCoreCallback.ResultCallback<List<GroupInfo>>() {
    @Override
    public void onSuccess(List<GroupInfo> groupInfos) {
        // 获取群信息成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 获取群信息失败
    }
});
```

### 获取指定已加入群组的资料

您可以使用 [getJoinedGroups] 根据当前用户已加入的群组 ID 获取群组资料。  
该接口支持批量获取，您可以一次传入多个 `groupId` 获取多个群组资料，最多不超过 **20** 个。

#### 代码示例
```java
// 群Id列表
List<String> groupIds = new ArrayList<>();
groupIds.add("groupId1");
groupIds.add("groupId2");
groupIds.add("groupId3");
RongCoreClient.getInstance().getJoinedGroups(groupIds, new IRongCoreCallback.ResultCallback<List<GroupInfo>>() {
    @Override
    public void onSuccess(List<GroupInfo> groupInfos) {
        // 拉取成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 拉取失败
    }
});
```

### 按角色获取已加入群组的资料

您可以使用 [getJoinedGroupsByRole] 按群成员角色分页获取已加入的群组。

此接口支持返回本次查询条件的总数，见 [PagingQueryOption] 的`getTotalCount()`。

#### 参数说明
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

#### 示例代码
```java
{
    //...
    String pageToken = "";
    getJoinedGroupsByRole(pageToken);
    //...
}

public void getJoinedGroupsByRole(String pageToken) {
    // 通过 role 参数指定拉取全部群成员角色类型的群
    GroupMemberRole role = GroupMemberRole.Undef;
    // 分页请求参数
    PagingQueryOption option = new PagingQueryOption();
    // 设置分页大小，取值范围为 [1~100]。
    option.setCount(20);
    // 分页拉取参数
    option.setPageToken(pageToken);
    // 按加入群组时间正序、倒序获取。true：正序；false：倒序
    option.setOrder(false);
    RongCoreClient.getInstance().getJoinedGroupsByRole(role, option, new IRongCoreCallback.PageResultCallback<GroupInfo>() {
        @Override
        public void onSuccess(PagingQueryResult<GroupInfo> result) {
            if (!TextUtils.isEmpty(result.getPageToken())) {
                // 使用返回的 pageToken 拉取下一页
                getJoinedGroupsByRole(result.getPageToken());
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

### 按群名称搜索已加入群组的资料

您可以使用 `searchJoinedGroups` 根据群名称搜索已加入的群组。  
此接口支持返回本次查询条件的总数，见 [PagingQueryOption] 的`getTotalCount()`。

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
    // 分页拉取参数（设置 null 与设置 ""，效果等同）
    String pageToken = "";
    searchJoinedGroups(pageToken);
    //...
}

public void searchJoinedGroups(String pageToken) {
    // 群名称
    String groupName = "groupName";
    // 分页请求参数
    PagingQueryOption option = new PagingQueryOption();
    // 设置分页大小，取值范围为 [1~200]。
    option.setCount(20);
    // 分页拉取参数
    option.setPageToken(pageToken);
    // 按加入群组时间正序、倒序获取。true：正序；false：倒序。
    option.setOrder(false);
    RongCoreClient.getInstance().searchJoinedGroups(groupName, option, new IRongCoreCallback.PageResultCallback<GroupInfo>() {
        @Override
        public void onSuccess(PagingQueryResult<GroupInfo> result) {
            if (!TextUtils.isEmpty(result.getPageToken())) {
                // 使用返回的 pageToken 拉取下一页
                searchJoinedGroups(result.getPageToken());
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

<!-- links -->
[getJoinedGroups]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-joined-groups.html
[getJoinedGroupsByRole]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-joined-groups-by-role.html
[getGroupsInfo]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-groups-info.html
[PagingQueryOption]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-option/index.html
[PagingQueryResult]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-paging-query-result/index.html
