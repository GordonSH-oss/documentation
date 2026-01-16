---
title: 监听数据库升级状态
sidebar_position: 60
---

从 5.10.4 版本开始，SDK 增加数据库升级状态监听接口。当 SDK 升级涉及到数据库模块变化时，覆盖安装 SDK 会触发数据库升级，数据库升级期间可能会造成连接状态的不稳定，故增加数据库升级状态监听接口，供您在收到数据库升级事件时增加升级提示或进度页面，缓解用户等待焦虑。

:::tip
数据库升级由 SDK 自动触发，不会造成用户数据丢失，升级时间与手机性能、数据库版本及大小有关，具体时间无法预估。
:::

`RongIMClient` 中增加了以下接口，用于增加和移除数据库升级状态监听器：

- `addDatabaseStatusListener`
- `removeDatabaseStatusListener`

**示例代码**

```java
// 设置数据库升级状态监听器
public void addDatabaseStatusListener(IRongCoreListener.DatabaseUpgradeStatusListener listener);

// 移除数据库升级状态监听器
public void removeDatabaseStatusListener(IRongCoreListener.DatabaseUpgradeStatusListener listener);
```

您可以自定义自己的数据库升级状态监听器，来实现 `DatabaseUpgradeStatusListener` 接口的 3 个函数。

- `databaseUpgradeWillStart`：数据库升级开始。
- `databaseIsUpgrading`：数据库升级进行中。
- `databaseUpgradeDidComplete`：数据库升级完成。


**示例代码**

```java
public interface DatabaseUpgradeStatusListener {
    /**
     * 数据库升级开始
     */
    public abstract void databaseUpgradeWillStart();

    /**
     * 数据库升级进行中回调
     *
     * @param progress 取值范围 0~100
     */
    public abstract void databaseIsUpgrading(int progress);

    /**
     * 数据库升级完成回调
     *
     * @param code 升级成功返回 {IRongCoreEnum.CoreErrorCode.SUCCESS}，升级失败返回。
     *     {IRongCoreEnum.CoreErrorCode.RC_DB_UPGRADE_FAILED}
     */
    public abstract void databaseUpgradeDidComplete(IRongCoreEnum.CoreErrorCode code);
}
```
