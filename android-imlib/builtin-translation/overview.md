---
title: 智能文本翻译
sidebar_position: 1
---

从 5.24.0 版本起，IMLib SDK（融云即时通讯能力库）支持对文本消息和纯文本内容进行翻译。您可以设置用户级别的翻译语言和自动翻译状态，也可为不同会话单独配置翻译策略。

IMLib SDK 提供了两个批量翻译接口：
- [`translateMessagesWithParams`]：用于翻译文本消息（`TextMessage`）内容。
- [`translateTextsWithParams`]：用于翻译任意文字内容。

:::tip
翻译功能需网络连接，请确保设备网络通畅。批量翻译内容可能耗时较长，接口不会直接返回翻译结果。请通过注册 `IRongCoreListener.TranslationListener` 监听器获取翻译内容。
:::

## 开启翻译服务

请前往[融云控制台]开启翻译功能。

## 用户级全局设置

全局设置对同一用户的所有登录设备生效。一端设置后，其他设备可通过 `IRongCoreListener.TranslationListener` 事件监听器获得变更通知。

### 设置翻译语言

通过 [`setTranslationLanguage`] 接口可设置用户级别的翻译目标语言。支持的语种请参考[翻译语言代码列表](supported-languages-and-codes.md) 。未设置时，默认目标语言为中文（`zh`）。

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `language` | String | 翻译目标语言的[语言代码](supported-languages-and-codes.md) |
| `callback` | `IRongCoreCallback.OperationCallback` | 结果回调 |

#### 示例代码

```java
RongCoreClient.getInstance().setTranslationLanguage("zh", 
    new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {
            // 设置翻译语言成功
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 设置翻译语言失败
        }
    });
```

### 获取翻译语言

通过 [`getTranslationLanguage`] 接口可获取当前用户级别的翻译目标语言。

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `callback` | `IRongCoreCallback.ResultCallback<String>` | 结果回调 |

#### 示例：获取翻译语言

```java
RongCoreClient.getInstance().getTranslationLanguage(
    new IRongCoreCallback.ResultCallback<String>() {
        @Override
        public void onSuccess(String language) {
            // 获取翻译语言成功
            Log.d(TAG, "当前翻译语言: " + language);
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 获取翻译语言失败
            Log.e(TAG, "getTranslationLanguage error: " + errorCode);
        }
    });
```

### 标记自动翻译状态

:::important 注意
IMLib SDK 本身不包含自动翻译功能，仅用于向应用的业务逻辑提供开关存储和多端状态同步。
:::

通过 [`setAutoTranslateEnable`] 接口可标记用户级别的自动翻译状态。

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `enable` | boolean | 是否标记为自动翻译 |
| `callback` | `IRongCoreCallback.OperationCallback` | 结果回调 |

#### 示例代码

```java
RongCoreClient.getInstance().setAutoTranslateEnable(true,
    new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {
            // 标记自动翻译状态成功
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 标记自动翻译状态失败
        }
    });
```

### 获取自动翻译状态

通过 [`getAutoTranslateEnabled`] 接口可获取当前用户级别的自动翻译状态。

#### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `callback` | `IRongCoreCallback.ResultCallback<Boolean>` | 结果回调 |

#### 示例代码

```java
RongCoreClient.getInstance().getAutoTranslateEnabled(
    new IRongCoreCallback.ResultCallback<Boolean>() {
        @Override
        public void onSuccess(Boolean isEnable) {
            // 获取自动翻译状态成功
            Log.d(TAG, "自动翻译状态: " + isEnable);
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 获取自动翻译状态失败
            Log.e(TAG, "getAutoTranslateEnabled error: " + errorCode);
        }
    });
```

### 监听全局配置变更

全局设置（翻译语言、自动翻译状态）的多端同步，通过 `IRongCoreListener.TranslationListener` 的 `onTranslationLanguageDidChange` 和 `onAutoTranslateStateDidChange` 回调。详见[翻译事件监听](#翻译事件监听)。

## 会话级翻译策略

:::important 注意
IMLib SDK 本身不包含自动翻译功能，仅用于向应用的业务逻辑提供开关存储和多端状态同步。
:::

IMLib SDK 支持会话级配置，可针对不同会话单独设置自动翻译策略。会话级配置对同一用户的所有设备生效，变更后其他设备可通过 `IRongCoreListener.ConversationStatusListener` 获取通知。

### 设置会话翻译策略

通过 [`batchSetConversationTranslateStrategy`] 接口可批量设置会话翻译策略。

#### 翻译策略枚举

`TranslateStrategy` 枚举：
- `DEFAULT`：默认策略，跟随用户级别自动翻译设置。
- `AUTO_ON`：自动翻译，会话内消息自动翻译。
- `AUTO_OFF`：手动翻译，会话内消息不自动翻译。

#### 示例代码

```java
List<ConversationIdentifier> identifiers = new ArrayList<>();
ConversationIdentifier identifier = new ConversationIdentifier();
identifier.setTargetId("targetId");
identifier.setType(Conversation.ConversationType.PRIVATE);
identifiers.add(identifier);

RongCoreClient.getInstance().batchSetConversationTranslateStrategy(
    identifiers, 
    TranslateStrategy.AUTO_ON,
    new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {
            // 设置会话翻译策略成功
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 设置会话翻译策略失败
        }
    });
```

### 查询会话翻译策略

获取会话对象 `Conversation` 后，可通过其 `translateStrategy` 属性读取当前策略。

#### 示例：获取会话翻译策略

```java
// 检查会话的翻译策略
TranslateStrategy strategy = conversation.getTranslateStrategy();
switch (strategy) {
    case DEFAULT:
        // 使用默认策略，跟随用户级别设置
        break;
    case AUTO_ON:
        // 该会话开启自动翻译
        break;
    case AUTO_OFF:
        // 该会话关闭自动翻译
        break;
}
```

### 监听会话状态变更

多端登录时，设置结果会通过 `IRongCoreListener.ConversationStatusListener` 监听器中的 `onStatusChanged(ConversationStatus[] conversationStatus)` 方法同步到其他端。`ConversationStatus` 的翻译状态键 `TRANSLATION_KEY` 将被设置为对应的翻译策略值。

#### 示例代码
```java
// 设置会话状态监听器
RongCoreClient.getInstance().addConversationStatusListener(new IRongCoreListener.ConversationStatusListener() {
    @Override
    public void onStatusChanged(ConversationStatus[] conversationStatus) {
        // 会话状态变更回调，包含翻译策略变更
        for (ConversationStatus status : conversationStatus) {
            IRongCoreEnum.TranslateStrategy translateStrategy = status.getTranslateStrategy();
            if (translateStrategy != null) {
                // 处理翻译策略变更
                Log.d(TAG, "会话翻译策略变更: " + translateStrategy);
            }
        }
    }
});
```

## 批量翻译文本消息

通过 [`translateMessagesWithParams`] 接口可批量翻译文本消息。

:::info
- 单次翻译请求的消息数量限制为 1-10 条
- 翻译结果通过异步回调返回，需注册翻译监听器
:::

### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `params` | `TranslateMessagesParams` | 翻译的消息参数 |
| `callback` | `IRongCoreCallback.OperationCallback` | 结果回调 |

#### TranslateMessagesParams 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `list` | `List<TranslateMessageParam>` | 翻译消息参数列表，长度限制 [1, 10]。 |
| `force` | `boolean` | 是否忽略缓存结果，强制重新翻译，默认为 `false`。 |
| `mode` | `TranslateMode` | 翻译模式，目前仅支持高速翻译模式（`TranslateMode.MECHANICAL`）。 |

#### TranslateMessageParam 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `messageUId` | `String` | 消息的唯一标识符（必须）。 |
| `sourceLanguage` | `String` | 源语言。为空时，则自动检测。 |
| `targetLanguage` | `String` | 目标语言，为空时使用配置的全局语言。 |

### 示例代码

```java
List<TranslateMessageParam> messageParams = new ArrayList<>();

TranslateMessageParam param = new TranslateMessageParam("messageUId");
// 翻译的语言，为空时，使用配置的全局语言
param.setTargetLanguage("zh");
// 消息内容的语言，为空时，会自动识别
param.setSourceLanguage("en");
messageParams.add(param);

TranslateMessagesParams params = new TranslateMessagesParams(messageParams);
// 翻译模式，默认为 TranslateMode.MECHANICAL
params.setMode(TranslateMode.MECHANICAL);
// 是否强制重新翻译，默认为 false
params.setForce(true);

RongCoreClient.getInstance().translateMessagesWithParams(params, 
    new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {
            // 翻译请求发送成功
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 翻译请求发送失败
        }
    });
```

## 批量翻译文本内容

通过 [`translateTextsWithParams`] 接口可批量翻译任意文本内容。

:::info
- 单次翻译请求的文本数量限制为 1-10 条
- 翻译结果通过异步回调返回，需注册翻译监听器
:::

### 参数说明

| 参数 | 类型 | 说明 |
| :--- | :--- | :--- |
| `params` | `TranslateTextsParams` | 翻译文本参数 |
| `callback` | `IRongCoreCallback.OperationCallback` | 结果回调 |

#### TranslateTextsParams 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `list` | `List<TranslateTextParam>` | 翻译文本参数列表，长度限制 [1, 10] |
| `mode` | `TranslateMode` | 翻译模式，目前仅支持高速翻译模式（`TranslateMode.MECHANICAL`）。 |

#### TranslateTextParam 属性说明

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `text` | `String` | 待翻译的文本内容（必须） |
| `sourceLanguage` | `String` | 源语言，为空时，则自动检测。 |
| `targetLanguage` | `String` | 目标语言，为空时使用配置的全局语言。 |

#### 示例代码

```java
List<TranslateTextParam> textParams = new ArrayList<>();

TranslateTextParam param = new TranslateTextParam("Hello, World!");
// 翻译的语言，为空时，使用配置的全局语言
param.setTargetLanguage("zh");
// 消息内容的语言，为空时，会自动识别
param.setSourceLanguage("en");
textParams.add(param);

TranslateTextsParams params = new TranslateTextsParams(textParams);
// 翻译模式，默认为 TranslateMode.MECHANICAL
params.setMode(TranslateMode.MECHANICAL);

RongCoreClient.getInstance().translateTextsWithParams(params,
    new IRongCoreCallback.OperationCallback() {
        @Override
        public void onSuccess() {
            // 翻译请求发送成功
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode errorCode) {
            // 翻译请求发送失败
        }
    });
```

## 监听翻译结果

翻译结果通过 `IRongCoreListener.TranslationListener` 的 `onTranslationDidFinished(List<TranslateItem> translateItems)` 方法异步返回。同时，消息对象 `TextMessage` 的 `translateInfo` 属性也会更新。

### 翻译结果数据结构

#### TranslateItem

`onTranslationDidFinished` 回调返回 `TranslateItem` 列表。

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `messageUId` | `String` | 消息的唯一标识符，用于匹配消息。 |
| `code` | `CoreErrorCode` | 翻译结果的状态码。 |
| `translateInfo` | `TranslateInfo` | 翻译结果信息。 |

#### TranslateInfo

`TextMessage` 的 `translateInfo` 属性以及 `TranslateItem` 中的 `translateInfo` 属性，都使用 `TranslateInfo` 对象存储翻译结果。

| 属性 | 类型 | 说明 |
| :--- | :--- | :--- |
| `translatedText` | `String` | 翻译后的内容 |
| `status` | `TranslateStatus` | 翻译状态（无、翻译中、成功、失败） |
| `targetLanguage` | `String` | 翻译目标语言 |

#### 翻译状态枚举

`TranslateInfo.TranslateStatus` 枚举，定义了翻译的如下状态：
- `NONE`：无翻译状态
- `TRANSLATING`：翻译中
- `SUCCESS`：翻译成功
- `FAILED`：翻译失败

## 翻译事件监听

:::info
- 翻译相关的事件回调都通过 `IRongCoreListener.TranslationListener` 异步返回。
- 请在适当时机移除翻译监听器，避免内存泄漏。
:::

### 添加翻译事件监听器

调用 [`addTranslationListener`] 接口添加翻译事件监听器。

#### 示例代码

```java
// 添加翻译事件监听
RongCoreClient.getInstance().addTranslationListener(new IRongCoreListener.TranslationListener() {
    @Override
    public void onTranslationDidFinished(List<TranslateItem> translateItems) {
        // 批量翻译完成回调
        for (TranslateItem item : translateItems) {
            if (item.getCode() == IRongCoreEnum.CoreErrorCode.SUCCESS) {
                // 翻译成功，获取翻译结果
                TranslateInfo info = item.getTranslateInfo();
                String translatedText = info.getTranslatedText();
                String targetLanguage = info.getTargetLanguage();
                TranslateInfo.TranslateStatus status = info.getStatus();
            }
        }
    }

    @Override
    public void onTranslationLanguageDidChange(String language) {
        // 用户级翻译语言变更通知
    }

    @Override
    public void onAutoTranslateStateDidChange(boolean isEnable) {
        // 用户级自动翻译状态变更通知
    }
});
```

### 移除翻译事件监听器

调用 [`removeTranslationListener`] 接口可以移除翻译事件监听器。

#### 示例代码

```java
// 移除翻译事件监听
RongCoreClient.getInstance().removeTranslationListener(translationListener);
```

<!-- links -->
<!-- 控制台链接 -->
[融云控制台]:https://console.rongcloud.cn/agile/ai/purchase#1704

<!-- api文档链接 -->
[`TextMessage`]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/index.html?query=public%20class%20TextMessage
[`translateInfo`]:https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/index.html#365398094%2FProperties%2F-103216741
[`translateMessagesWithParams`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/translate-messages-with-params.html
[`translateTextsWithParams`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/translate-texts-with-params.html
[`setTranslationLanguage`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/set-translation-language.html
[`getTranslationLanguage`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-translation-language.html
[`setAutoTranslateEnable`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/set-auto-translate-enable.html
[`getAutoTranslateEnabled`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/get-auto-translate-enabled.html
[`batchSetConversationTranslateStrategy`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/batch-set-conversation-translate-strategy.html
[`setConversationStatusListener`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/set-conversation-status-listener.html
[`addTranslationListener`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/add-translation-listener.html
[`removeTranslationListener`]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/remove-translation-listener.html
