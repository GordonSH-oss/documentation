---
title: 监听用户组状态变更
sidebar_position: 60
---

# 监听用户组状态变更

> SDK 从 5.4.0 版本开始支持超级群用户组功能。

您可以设置监听，在超级群用户组发生变更时收取到对应通知。

IMLib SDK 针对以下操作提供回调。回调的通知范围与回调数据有差异具体如下：

- **用户加入用户组**：被加入用户组的用户可收到通知，通知中携带超级群 ID、用户组 ID。
- **用户被移出用户组**：被移出的用户可收到通知，通知中携带超级群 ID、用户组 ID。
- **频道与用户组绑定**：用户组下所有用户均可收到通知，通知中携带超级群 ID、频道 ID、频道类型、用户组 ID。
- **频道与用户组解除绑定**：用户组下所有用户均可收到通知，通知中携带超级群 ID、频道 ID、频道类型、用户组 ID。
- **删除用户组**：用户组下所有用户均可收到通知，通知中携带超级群 ID、用户组 ID。

创建用户组时，SDK 不会收到回调。

## 监听用户组变更通知

您可通过 `setUserGroupStatusListener` 方法，在初始化 IMLib SDK 后，连接 IM 之前设置用户组变更监听器。

#### 示例代码

```java
ChannelClient.getInstance().setUserGroupStatusListener(
    new IRongCoreListener.UserGroupStatusListener() {
        @Override
        public void userGroupDisbandFrom(
                ConversationIdentifier identifier, String[] userGroupIds) {
        }

        @Override
        public void userAddedTo(
                ConversationIdentifier identifier, String[] userGroupIds) {
        }

        @Override
        public void userRemovedFrom(
                ConversationIdentifier identifier, String[] userGroupIds) {
        }

        @Override
        public void userGroupBindTo(
                ConversationIdentifier identifier,
                IRongCoreEnum.UltraGroupChannelType channelType,
                String[] userGroupIds) {
        }

        @Override
        public void userGroupUnbindFrom(
                ConversationIdentifier identifier,
                IRongCoreEnum.UltraGroupChannelType channelType,
                String[] userGroupIds) {

        }
    }
);
```

IMLib SDK 在 `IRongCoreListener.UserGroupStatusListener` 接口类中提供了与用户组变更相关的回调方法。您可以从返回的 `ConversationIdentifier` 中获取超级群 ID（`targetId`）、频道 ID（`channelId`）数据。

```java
interface UserGroupStatusListener {
    /**
        * 当前用户收到超级群下的用户组中解散通知
        *
        * @param identifier 会话标识
        * @param userGroupIds 用户组ID列表
        */
    void userGroupDisbandFrom(ConversationIdentifier identifier, String[] userGroupIds);

    /**
        * 当前用户被添加到超级群下的用户组
        *
        * @param identifier 会话标识
        * @param userGroupIds 用户组ID列表
        */
    void userAddedTo(ConversationIdentifier identifier, String[] userGroupIds);

    /**
        * 当前用户从到超级群下的用户组中被移除
        *
        * @param identifier 会话标识
        * @param userGroupIds 用户组ID列表
        */
    void userRemovedFrom(ConversationIdentifier identifier, String[] userGroupIds);

    /**
        * 频道中绑定用户组回调
        *
        * @param identifier 频道标识
        * @param channelType 频道类型
        * @param userGroupIds 用户组ID列表
        */
    void userGroupBindTo(
            ConversationIdentifier identifier,
            IRongCoreEnum.UltraGroupChannelType channelType,
            String[] userGroupIds);

    /**
        * 频道解绑用户组回调
        *
        * @param identifier 频道标识
        * @param channelType 频道类型
        * @param userGroupIds 用户组ID列表
        */
    void userGroupUnbindFrom(
            ConversationIdentifier identifier,
            IRongCoreEnum.UltraGroupChannelType channelType,
            String[] userGroupIds);
}
```




## 关于更新 UI 的提示

考虑到同一用户既在私有频道成员列表中，又在私有频道绑定的（多个）用户组中，App 可以在收到回调时向 App 自身业务服务端查询当前用户是否仍可继续访问该私有频道，并根据该结果刷新 UI。

对于未在通知范围内的用户，可以在用户进入相应页面时向 App 自身业务服务端查询数据，并决定是否需要刷新 UI。

