---
title: 本地通知
sidebar_position: 1
---

IMLib 未实现本地通知功能，需要您的 App 自行实现本地通知。

:::tip

 - 当 App 刚进入后台时， App 处于后台活跃状态，SDK 通过长连接通道接收消息，此时 SDK 收到消息会触发消息接收监听。您可以在收到消息监听通知时弹出本地通知。关于消息监听器的说明，详见[接收消息](../message/receive.md)。
 - App 进入后台如果被系统杀死，SDK 会在断开连接后通过推送通道推送通知。

:::

<!--public-cloud-only start-->
由于超级群产品本身的业务特性，App 可能需要配合**免打扰级别**（详见[超级群免打扰功能概述]）功能，实现精细化的本地通知控制策略。当前 IMLib SDK 获取免打扰的 API 是异步的，每次都从数据库中获取，可能会造成一定延迟，无法满足部分 App 对本地通知处理的要求。如果您希望实现从内存中获取免打扰级别数据，可以参考融云 IMKit SDK 中的实现方案。<!--public-cloud-only end-->

如果您需要参考 IMKit 的本地通知实现逻辑，请查看 [IMKit 本地通知文档](/android-imkit/notification/customize-notification-local#imkit-本地通知实现逻辑)。

<!-- api -->
[超级群免打扰功能概述]: ../ultragroup/do-not-disturb-about.md

