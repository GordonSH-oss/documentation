---
sidebar_position: 10
title: 群组管理
---

本文档旨在指导开发者如何使用融云即时通讯 Android IMLib SDK 实现创建群组、群组资料设置、踢出群组、退出群组、解散群组、转让群组等功能。


:::tip
- 此功能从 5.12.0 版本开始支持。
- 针对已经通过原群组功能接口`/group/create.json`创建的群组，默认不支持调用群托管的相关功能接口，需要调用**导入群托管数据**接口，设置群组所有者（群主）及群组的默认权限后才能使用。
:::

## 开通服务

信息托管服务已默认开通，您可以直接使用此功能。

## 群事件监听

群事件定义在 [GroupEventListener] 中，包含群操作、群信息群成员信息变更、加群申请相关通知、群备注名多端同步、群特别关注多端同步等。  
您可以调用 `setGroupEventListener` 设置群事件监听器，如果不想再接收群事件，接口传 `null` 即可。  

:::tip
信息托管服务中，群组操作产生的 SDK 通知回调也是一种状态通知行为，不管应用中是否实现 SDK 的事件监听，服务端都会对 SDK 进行状态同步，确保本地为最新的数据信息，所以会计入消息的分发、下行数据统计中。
:::


#### 代码示例
```java
GroupEventListener groupEventListener = new GroupEventListener() {
    @Override
    public void onGroupOperation(String groupId, GroupMemberInfo operatorInfo, GroupInfo groupInfo, GroupOperation operation, List<GroupMemberInfo> memberInfos, long operationTime) {
        // 执行群操作后的回调：包括创建群、加入群、踢出、退出、解散、添加管理员、移除管理员、转让群主，具体参照 [GroupOperation] 
    }
    @Deprecated
    @Override
    public void onGroupInfoChanged(GroupMemberInfo operatorInfo, GroupInfo groupInfo, List<GroupInfoKeys> updateKeys, long operationTime) {
        // 群组资料变更回调。注意：只有包含在 updateKeys 中的属性值有效
    }

    /**
     * 群组资料变更回调
     *
     * @param operatorInfo 操作人用户信息
     * @param fullGroupInfo 群组信息
     * @param changeGroupInfo 变更的群组信息
     * @param updateKeys 群组信息本次更新有变化的 Key 列表。
     * @param operationTime 操作时间。
     */
    default void onGroupInfoChanged(GroupMemberInfo operatorInfo,GroupInfo fullGroupInfo,GroupInfo changeGroupInfo,List<GroupInfoKeys> updateKeys,long operationTime) {
        // 群组资料变更回调。注意：只有包含在 updateKeys 中的属性值有效
    }

    @Override
    public void onGroupMemberInfoChanged(String groupId, GroupMemberInfo operatorInfo, GroupMemberInfo memberInfo, long operationTime) {
        // 群成员资料变更回调
    }

    @Override
    public void onGroupApplicationEvent(GroupApplicationInfo info) {
        // 群请求事件回调。包含以下事件：
        // 1，用户申请加入群组的 "申请" 或 "结果"
        // 2，邀请加入群组的 "申请" 或 "结果"
    }

    @Override
    public void onGroupRemarkChangedSync(String groupId, GroupOperationType operationType, String groupRemark, long operationTime) {
        // 群名称备注名更新多端同步回调事件。
    }

    @Override
    public void onGroupFollowsChangedSync(String groupId, GroupOperationType operationType, List<String> userIds, long operationTime) {
        // 群成员特别关注变更多端回调事件。
    }
};
RongCoreClient.getInstance().setGroupEventListener(groupEventListener);
```


## 群组管理

群组管理包含：创建群组、群组资料设置、踢出群组、退出群组、解散群组、群组转让。

### 创建群组

您可以调用 `createGroup` 方法创建一个新群组。

#### 接口
```java
RongCoreClient.getInstance().createGroup(groupInfo, inviteeUserIds, callback);
```

#### 群资料 `GroupInfo` 参数介绍：

| 属性名 | 类型                         | **必填** | 描述                                                                                                        |
| ------- |----------------------------|------------------|-----------------------------------------------------------------------------------------------------------|
| groupId      | String| 是 | 群组 ID：最大长度 64 个字符。支持大小写英文字母与数字的组合。                                                                        |
| groupName    | String| 是 | 群组名称：最大长度 64 个字符。                                                                                         |
| portraitUri  | String| 否 | 群头像地址：长度不超过 128 个字符                                                                                       |
| introduction | String| 否 | 群简介：最大长度不超过 512 个字符                                                                                       |
| notice       | String| 否 | 群公告：最大长度不超过 1024 个字符                                                                                      |
| extProfile   | Map\<String, String\>| 否 | 扩展信息：在现在群信息基础上，开发者可根据自身业务需求添加自定义扩展属性（Key、Value），最多可设置 10 个扩展信息。 自定义属性需要通过融云控制台 [群组自定义属性] 设置后才能使用，否则返回设置失败 |
| joinPermission  | [GroupJoinPermission]| 否 | 加入群组的权限：不用验证、需要群主验证（默认）、需要群管理员及群主验证、不允许任何人加入，不设置按默认值处理                                                    |                                                           
| removeMemberPermission   | [GroupOperationPermission] | 否 | 将群成员移出群组的权限：群主（默认）、群主+群管理员、所有人，不设置按默认值处理                                                                  |
| invitePermission         | [GroupOperationPermission] | 否 | 邀请他人加入群组的权限：群主（默认）、群主+群管理员、所有人，不设置按默认值处理                                                                  |
| inviteHandlePermission   | [GroupInviteHandlePermission] | 否 | 被邀请加入群组处理的权限：不需要被邀请人同意（默认）、需要被邀请人同意，不设置按默认值处理                                                             |
| groupInfoEditPermission  | [GroupOperationPermission] | 否 | 修改群资料的权限：群主（默认）、群主+群管理员、所有人，不设置按默认值处理。此权限只有群主可以修改                                                         |
| memberInfoEditPermission | [GroupMemberInfoEditPermission] | 否 | 设置群成员资料的权限：仅自已、群主+自已、群主+群管理员+自已（默认），不设置按默认值处理                                                             |

#### `inviteeUserIds` 参数介绍

您可以选择在创建群的同时，邀请用户加入群组。一次最多允许 **30** 个用户加入。

邀请的结果受群组的**被邀请人处理权限** `inviteHandlePermission` 参数影响：
- 需要被邀请人验证 ( `InviteeVerify` ) 时，创建群组成功后，成功回调的 `IRongCoreEnum.CoreErrorCode` 会返回 `RC_GROUP_NEED_INVITEE_ACCEPT(25427)`，被邀请人会收到 [onGroupApplicationEvent] 事件，需要调用 `acceptGroupInvite` 接口同意后才能加入群组。
- 无需被邀请人验证 ( `Free` ) 时，创建群组成功后，成功回调的 `IRongCoreEnum.CoreErrorCode` 会返回 `SUCCESS(0)`， 表示被邀请人已加入群组。

:::tip
- 群组创建的结果不受 `IRongCoreEnum.CoreErrorCode` 的影响，只要回调了 `onSuccess` 就表示群组创建成功。   
- 创建群组成功，群主会收到 [onGroupOperation] 回调，`GroupOperation` 类型 `Create`。
:::

#### 示例代码
```java
GroupInfo groupInfo = new GroupInfo();
// 群Id。必填，否则创建失败返回错误码
groupInfo.setGroupId("groupId");
// 群名称。必填，否则创建失败返回错误码
groupInfo.setGroupName("groupName");
// 设置加入群组的用户Id。非必填
List<String> inviteeUserIds = new ArrayList<>();
inviteeUserIds.add("userId1");
inviteeUserIds.add("userId2");
inviteeUserIds.add("userId3");
RongCoreClient.getInstance().createGroup(groupInfo, inviteeUserIds, new IRongCoreCallback.CreateGroupCallback() {
    @Override
    public void onSuccess(IRongCoreEnum.CoreErrorCode processCode) {
        // 创建群组请求成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e, String errorData) {
        // 创建群组请求失败，如果errorData不为空，代表GroupInfo具体出错的key
    }
});
```

### 修改群组资料

您可以调用 `updateGroupInfo` 方法修改群组名称、公告、权限等其他资料。修改群组资料时，只有 `groupId` 为必填项，其他字段可按需设置，接口只会更新有修改的项。

:::tip
- 群组信息更新成功后，群组内的所有成员会收到 [onGroupInfoChanged] 回调。
- 群信息更新权限：`groupInfoEditPermission` ，只有群主可以修改。
:::

#### 代码示例
```java
GroupInfo groupInfo = new GroupInfo();
// 群Id
groupInfo.setGroupId("groupId");
RongCoreClient.getInstance().updateGroupInfo(groupInfo, new IRongCoreCallback.OperationCallbackEx<String>() {
    @Override
    public void onSuccess() {
        // 更新群组信息成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e, String errorData) {
        // 更新群组信息失败，如果errorData不为空，代表GroupInfo具体出错的key
    }
});
```

### 踢出群组

您可以调用 `kickGroupMembers` 方法将用户从群组中移除。

该接口支持批量移除，您可以一次传入多个 `userID` 移除多个用户，最多不超过 **100** 个。

在移除时可以设置 [QuitGroupConfig] 配置项，控制是否移除特别关注的用户、白名单用户、群成员禁言状态。如果不设置，默认为都移除。

将群成员移出群组权限 `removeMemberPermission` ([GroupOperationPermission])，决定是否可以踢出群组，如果没有权限设置，调用将会返回错误码。

:::tip
- 踢出成功后，群组内所有成员会收到 [onGroupOperation] 回调，`GroupOperation` 类型是 `Kick`。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 用户ID列表
List<String> userIds = new ArrayList<>();
userIds.add("userId1");
userIds.add("userId2");
userIds.add("userId3");
QuitGroupConfig config = new QuitGroupConfig();
// 关闭移除群组特别关注的用户（默认为 YES）
config.setRemoveFollow(false);
// 关闭移除白名单（默认为 YES）
config.setRemoveWhiteList(false);
// 关闭移除群成员禁言状态 （默认为 YES）
config.setRemoveMuteStatus(false);
RongCoreClient.getInstance().kickGroupMembers(groupId, userIds, config, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 踢出成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 踢出失败
    }
});
```

### 退出群组

您可以调用 `quitGroup` 方法主动退出群组。

在退出时可以设置 [QuitGroupConfig] 配置项，以控制是否移除特别关注的用户、白名单用户、群成员禁言状态。若未设置 `config`，默认在退出群组时会全部移除。

:::tip
- 退出成功后，群组内的所有成员会收到 [onGroupOperation] 事件，`GroupOperation` 类型是 `Quit`。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
QuitGroupConfig config = new QuitGroupConfig();
// 关闭移除群组特别关注的用户（默认为 YES）
config.setRemoveFollow(false);
// 关闭移除白名单（默认为 YES）
config.setRemoveWhiteList(false);
// 关闭移除群成员禁言状态 （默认为 YES）
config.setRemoveMuteStatus(false);
RongCoreClient.getInstance().quitGroup(groupId, config, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 退出成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 退出失败
    }
});
```

### 解散群组

您可以调用 `dismissGroup` 方法解散群组。

解散群组成功后，群会话消息仍然保留，本地历史消息不做删除处理。

:::tip
- 只有群主有权解散群组。
- 退出成功后，群组内的所有成员会收到 [onGroupOperation] 事件，`GroupOperation` 类型是 `Dismiss`。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
RongCoreClient.getInstance().dismissGroup(groupId, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 解散成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 解散失败
    }
});
```

### 转让群组

您可以调用 `transferGroupOwner` 方法将群组转让给其他用户。

调用 `transferGroupOwner` 方法时，您可以选择在转让成功后是否退出群组（`quitGroup`），若设置 `true` 退出群组，您还可以设置 [QuitGroupConfig] 配置项，以控制是否移除特别关注的用户、白名单用户、群成员禁言状态。若未设置 `config`，默认在退出群组时会全部移除。

退出成功后，群组内的所有成员会收到 [onGroupOperation] 事件，`GroupOperation` 类型是 `TransferGroupOwner`。

若在转让时选择了退出群组，转让成功后，群组内的所有成员除了会收到 [onGroupOperation] 操作类型为 `TransferGroupOwner` 的转让事件，还会收到操作类型为 `Quit` 的退出事件。

:::tip
- 只有群主有权转让群组。
:::

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 转让后的群主用户ID
String newOwnerId = "userId1";
// 转让后退出群组
boolean quitGroup = true;
QuitGroupConfig config = new QuitGroupConfig();
// 关闭移除群组特别关注的用户（默认为 YES）
config.setRemoveFollow(false);
// 关闭移除白名单（默认为 YES）
config.setRemoveWhiteList(false);
// 关闭移除群成员禁言状态 （默认为 YES）
config.setRemoveMuteStatus(false);
RongCoreClient.getInstance().transferGroupOwner(groupId, newOwnerId, quitGroup, config, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 转让成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 转让失败
    }
});
```

## 群组备注名

#### 功能介绍

- 群组备注名只对用户本人生效。当群组中有新消息产生且该用户不在线时，推送标题将优先显示设置的群备注名。若用户未对该群设置备注名，则默认显示群名称。
- 设置或移除成功后，其他终端会收到群组备注名变更多端回调 [onGroupRemarkChangedSync]。

### 设置群组备注名

您可以调用 `setGroupRemark` 方法设置群组备注名。

当需要移除群组备注名时，`remark` 参数传 `null` 或空字符串 `""`。

#### 代码示例
```java
// 群Id
String groupId = "groupId";
// 群备注名
String remark = "remark";
RongCoreClient.getInstance().setGroupRemark(groupId, remark, new IRongCoreCallback.OperationCallback() {
    @Override
    public void onSuccess() {
        // 设置成功
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
        // 设置失败
    }
});
```

### 查询群组备注名

您可以使用 `getGroupsInfo` 方法获取群组资料，其中包含有关 `remark` 信息。

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
        if (groupInfos != null && !groupInfos.isEmpty()) {
            for (GroupInfo info : groupInfos) {
                String remark = info.getRemark();
                // 获取群组的 remark 成功
            }
        }
    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode e) {
        // 获取失败
    }
});
```


<!-- links -->
[GroupJoinPermission]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-group-join-permission/index.html
[GroupOperationPermission]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-group-operation-permission/index.html
[GroupInviteHandlePermission]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-group-invite-handle-permission/index.html
[GroupMemberInfoEditPermission]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-group-member-info-edit-permission/index.html
[QuitGroupConfig]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-quit-group-config/index.html
[GroupEventListener]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-group-event-listener/index.html
[onGroupOperation]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-group-event-listener/on-group-operation.html
[onGroupInfoChanged]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-group-event-listener/on-group-info-changed.html
[onGroupApplicationEvent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-group-event-listener/on-group-application-event.html
[onGroupRemarkChangedSync]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.listener/-group-event-listener/on-group-remark-changed-sync.html
[群组自定义属性]: https://console.rongcloud.cn/agile/im/messageTrusteeship/customGroup

