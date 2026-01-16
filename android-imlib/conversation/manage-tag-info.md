---
title: 管理标签信息数据
sidebar_position: 70
---

# 管理标签信息数据

> SDK 从 5.1.1 版本开始支持创建标签。

本文描述如何使用 [RongCoreClient](https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/index.html) 下的接口创建和管理标签信息数据。IMLib SDK 支持用户创建标签信息（[TagInfo]），用于对会话进行标记分组。每个用户最多可以创建 20 个标签。客户端创建的标签信息数据会同步融云服务端。

标签信息（[TagInfo]）参数说明：

| 参数 | 类型 | 说明 |
:-----|:-----|:----|
| tagId | String | 标签唯一标识，字符型，长度不超过 10 个字。 |
| tagName | String | 长度不超过 15 个字，标签名称可以重复。 |
| count | int | 匹配的会话个数。 |
| timestamp | long | 时间戳由 SDK 内部协议栈提供。 |

:::tip

 本文仅描述如何管理标签信息数据。关于如何为会话设置标签、以及如何按标签获取会话数据，请参见[设置与使用会话标签]。
:::


## 创建标签信息

创建标签，每个用户最多可以创建 20 个标签。


#### 示例代码

```java
RongCoreClient.getInstance().addTag(tagInfo, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 移除标签信息

移除标签。移除标签信息时只需要传入 [TagInfo] 中的 `tagId`。

#### 示例代码

```java
RongCoreClient.getInstance().removeTag(tagId, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 编辑标签信息

您可以修改  [TagInfo] 中的标签名称（`tagName`）字段来更新标签信息。

#### 示例代码

```java
RongCoreClient.getInstance().updateTag(tagInfo, new IRongCoreCallback.OperationCallback() {

    @Override
    public void onSuccess() {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 获取标签信息列表

获取当前用户已创建的标签信息。成功回调中会返回 [TagInfo] 列表。

#### 示例代码

```java
RongCoreClient.getInstance().getTags(new IRongCoreCallback.ResultCallback<List<TagInfo>>() {

    /**
        * 成功时回调
        * @param messages 获取的消息列表
        */
    @Override
    public void onSuccess(List<TagInfo> tagInfos) {

    }

    @Override
    public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {

    }
});
```

## 多端同步标签信息修改

融云支持同一用户账号在多端登录。设置标签信息更改监听器 [IRongCoreListener.TagListener] 后可在当前设备上接收到来自其他设备的标签信息修改通知。收到通知后需调用 `getTags` 方法从融云服务端获取最新标签信息。

:::tip

 - 请在初始化之后，连接之前调用该方法。
 - 在当前设备上修改标签信息不会触发该回调方法。服务端仅会通知 SDK 在同一用户账号登录的其他设备上触发回调。
:::

当用户在其它端添加、移除、编辑标签时，都会触发 `TagListener` 的 `onTagChanged` 回调。请在收到通知后调用 `getTags` 从融云服务端获取最新标签信息。


```java
RongCoreClient.getInstance().setTagListener(new IRongCoreListener.TagListener{
            @Override
            public void onTagChanged() {
            }
});
```


<!-- links -->
[设置与使用会话标签]: ./tag.md
<!-- api links -->
[TagInfo]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-tag-info/
[IRongCoreListener.TagListener]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-i-rong-core-listener/-tag-listener/index.html

