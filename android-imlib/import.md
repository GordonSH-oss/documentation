---
sidebar_position: 30
title: 导入 SDK
---

# 导入 SDK

利用 Android Studio 中的 Gradle 构建系统，您可以轻松地将融云即时通讯能力库（IMLib）作为依赖项添加到您的构建中。

您可以使用 Gradle 直接导入远程依赖项，或者以 Android 本地库模块导入本地依赖项。

## 环境要求

- （SDK ≧ 5.6.3）使用 Android 5.0（API 21）或更高版本
- （SDK \< 5.6.3）使用 Android 4.4（API 19）或更高版本

## 检查版本

在导入 SDK 前，您可以[前往融云官网 SDK 下载页面]确认当前最新版本号。

## Gradle {#gradle}

使用 Gradle，添加融云即时通讯能力库（IMLib）为远程依赖项。请注意使用 [融云的 Maven 仓库](http://maven.rongcloud.cn)。Android Studio 的配置在 Gradle 插件 7.0 以下版本、7.0 版本、和 7.1 及以上版本有所不同。请根据您当前的 Gradle 插件版本进行配置。本文以使用 Gradle 插件 7.0 以下版本为例。

1. 声明[融云的 Maven 代码库]，以使用 Gradle 插件 7.0 以下版本为例。打开根目录下的 `build.gradle`（**Project** 视图下）：

    ```groovy
    allprojects {
        repositories {
            ...
            //融云 maven 仓库地址
            maven {url "https://maven.rongcloud.cn/repository/maven-releases/"}
        }
    }
    ```

   如果使用 Gradle 插件的版本为 8.0 及以上，建议在根目录下的`settings.gradle`中配置。
   ```groovy
    dependencyResolutionManagement {
        repositories {
            ...
            //融云 maven 仓库地址
            maven {url "https://maven.rongcloud.cn/repository/maven-releases/"}
        }
    }
    ```

2. 在应用的 `build.gradle` 中，添加融云即时通讯能力库（IMLib）为远程依赖项。

    ```groovy
    dependencies {
        ...
        //此处以集成 IMLib 为例
        api 'cn.rongcloud.sdk:im_lib:x.y.z'
    }
    ```

    :::tip

    各个 SDK 的最新版本号可能不相同，还可能是 x.y.z.h，可前往 [融云官网 SDK 下载页面] 或 [融云的 Maven 代码库] 查询。
    :::


## Android 本地库 (Module) {#module}

在导入 SDK 前，您需要[前往融云官网 SDK 下载页面]，将 **即时通讯能力库 IMLib** 下载到本地。

1. 在 Android Studio 中打开工程后，依次点击 **File** > **New** > **Import Module**，找到下载的 Module 组件并导入。

1. 如果导入的内容中包含有插件的 aar 包，请移至 `app/libs` 目录下。

1. 打开根目录下的 `settings.gradle`（**Project** 视图下），添加 IMLib 本地库模块。

    ```groovy
    include ':IMLib'
    ...
    ```

1. 在应用的 `build.gradle` 中，添加 IMLib 为本地库模块依赖项。

    ```groovy
    dependencies {
        ...
        api project(':IMLib')
        ...
    }
    ```

1. (可选) 以 Android 本地库模块导入 SDK 时默认不带 Javadoc。建议自行从[融云的 Maven 代码库]下载 Javadoc 并导入，以便于在 Android Studio 中即时查看。

    如需指导，请参见以下知识库链接：

    https://help.rongcloud.cn/t/topic/727

<!-- 以下为参考链接 -->
[融云的 Maven 代码库]: http://maven.rongcloud.cn/#browse/browse:maven-releases
[前往融云官网 SDK 下载页面]:https://www.rongcloud.cn/devcenter?type=sdk
[融云官网 SDK 下载页面]:https://www.rongcloud.cn/devcenter?type=sdk

