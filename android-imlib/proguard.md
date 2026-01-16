---
title: 混淆
sidebar_position: 140
---

# 混淆

## 混淆脚本

在项目目录下的 `proguard-rules.pro` 文件中添加混淆配置。

```properties
-keepattributes Exceptions,InnerClasses

-keepattributes Signature

-keep class io.rong.** {*;}
-keep class cn.rongcloud.** {*;}
-keep class * implements io.rong.imlib.model.MessageContent {*;}
-dontwarn io.rong.push.**
-dontnote com.xiaomi.**
-dontnote com.google.android.gms.gcm.**
-dontnote io.rong.**

-ignorewarnings
```

当代码中有继承 `PushMessageReceiver` 的子类时，需 keep 所创建的子类广播。

示例：

```properties
-keep class io.rong.app.DemoNotificationReceiver {*;} 
```

把 `io.rong.app.DemoNotificationReceiver` 改成子类广播的完整类路径即可。

