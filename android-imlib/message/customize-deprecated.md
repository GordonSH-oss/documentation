---
title: 自定义消息类型（旧版）
sidebar_position: 120
---

# 自定义消息类型（旧版）

> **重要**
>
> 本文适用于 SDK \< 5.6.7 版本。如果您的 SDK 版本 ≧ 5.6.7，推荐使用新版[自定义消息](./customize.md)。

除了使用 IMLib SDK 内置消息外，还可以创建自定义消息类型。
您需要根据业务需求来选择自定义消息类型继承的消息基类：

- [MessageContent]：即普通类型消息内容。例如在 SDK 内置消息类型中的文本消息和位置消息。
- [MediaMessageContent]：即媒体类型消息。媒体类型消息内容继承自 [MessageContent]，并在其基础上增加了对多媒体文件的处理逻辑。在发送和接收消息时，SDK 会判断消息类型是否为多媒体类型消息，如果是多媒体类型，则会触发上传或下载多媒体文件流程。

:::tip

 自定义消息的类型、消息结构需要确保多端一致，否则将出现无法互通的问题。
:::


## 创建自定义消息

SDK 不负责定义和解析自定义消息的具体内容，您需要自行实现。

自定义消息类型的 `@MessageTag`（[MessageTag]） 决定该消息类型的唯一标识（`objectname`），以及是否存储、是否展示、是否计入消息未读数等属性。

1. 创建一个 [MessageContent] 的子类。如果自定义媒体消息，需要继承 [MediaMessageContent]。以下示例中创建了一个 `MyTextContent` 类：

    ```java
    @MessageTag(value = "app:txtcontent", flag = MessageTag.ISCOUNTED)
    public class MyTextContent extends MessageContent {

        private static final String TAG = "MyTextContent";
        // 自定义消息变量，可以有多个
        private String content;

        private MyTextContent() {}

        /**
        * 设置文字消息的内容。
        *
        * @param content 文字消息的内容。
        */
        public void setContent(String content) {
            this.content = content;
        }

    }
    ```

    继承自 [MessageContent] 或 [MediaMessageContent] 的子类都必须使用 `@MessageTag` 添加消息注解。以上是一个非媒体消息的示例，其中的 `MessageTag` 指定了消息类型的唯一标识为 `app:txtcontent`、需要在客户端存入数据库、且需要计入未读消息数。

    `MessageTag` 字段说明与详细用法参见下文[如何添加消息注解](#messagetag)。

1. 重写父类的 `encode` 方法，将 `MyTextContent` 的属性写入 JSON，转为 JSON 字符串，最后编码为字节序列（Bytes 数组）。

    ```java
        /**
         * 将本地消息对象序列化为消息数据。
         *
         * @return 消息数据。
         */
        @Override
        public byte[] encode() {
            JSONObject jsonObj = new JSONObject();
            try {
                // 消息携带用户信息时, 自定义消息需添加下面代码
                if (getJSONUserInfo() != null) {
                    jsonObj.putOpt("user", getJSONUserInfo());
                }
                // 用于群组聊天, 消息携带 @ 人信息时, 自定义消息需添加下面代码
                if (getJsonMentionInfo() != null) {
                    jsonObj.putOpt("mentionedInfo", getJsonMentionInfo());
                }
                //  将所有自定义消息的内容，都序列化至 json 对象中
                jsonObj.put("content", this.content);
            } catch (JSONException e) {
                Log.e(TAG, "JSONException " + e.getMessage());
            }

            try {
                return jsonObj.toString().getBytes("UTF-8");
            } catch (UnsupportedEncodingException e) {
                Log.e(TAG, "UnsupportedEncodingException ", e);
            }
            return null;
        }
    ```

1. 重写父类的 `MessageContent(byte[] data)` 构造方法，以实现对自定义消息内容的解析。先由字节序列（Bytes）解码成 JSON 字符串，再构造 JSON，并将内容取出赋给 `MyTextContent` 的属性。

    ```java
    /** 创建 MyMyTextContent(byte[] data) 带有 byte[] 的构造方法用于解析消息内容. */
    public MyTextContent(byte[] data) {
        if (data == null) {
            return;
        }
        String jsonStr = null;
        try {
            jsonStr = new String(data, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            Log.e(TAG, "UnsupportedEncodingException ", e);
        }
        if (jsonStr == null) {
            Log.e(TAG, "jsonStr is null ");
            return;
        }

        try {
            JSONObject jsonObj = new JSONObject(jsonStr);
            // 消息携带用户信息时, 自定义消息需添加下面代码
            if (jsonObj.has("user")) {
                setUserInfo(parseJsonToUserInfo(jsonObj.getJSONObject("user")));
            }
            // 用于群组聊天, 消息携带 @ 人信息时, 自定义消息需添加下面代码
            if (jsonObj.has("mentionedInfo")) {
                setMentionedInfo(parseJsonToMentionInfo(jsonObj.getJSONObject("mentionedInfo")));
            }
            // 将所有自定义变量从收到的 json 解析并赋值
            if (jsonObj.has("content")) {
                content = jsonObj.optString("content");
            }
        } catch (JSONException e) {
            Log.e(TAG, "JSONException " + e.getMessage());
        }
    }
    ```

1. 使用 `ParcelUtils` 工具类实现 `MyTextContent` 的序列化与反序列化。

    :::tip

    序列化中的 Parcel 的读写个数和顺序一定要一一对应。
    :::


```java
       /**
         * 描述了包含在 Parcelable 对象排列信息中的特殊对象的类型。
         *
         * @return 一个标志位，表明 Parcelable 对象特殊对象类型集合的排列。
         */
        public int describeContents() {
            return 0;
        }

        /**
         * 将类的数据写入外部提供的 Parcel 中。
         *
         * @param dest 对象被写入的 Parcel。
         * @param flags 对象如何被写入的附加标志，可能是 0 或 PARCELABLE_WRITE_RETURN_VALUE。
         */
        @Override
        public void writeToParcel(Parcel dest, int flags) {
            ParcelUtils.writeToParcel(dest, getExtra());
            ParcelUtils.writeToParcel(dest, content);
        }

        /**
         * 构造函数。
         *
         * @param in 初始化传入的 Parcel。
         */
        public MyTextContent(Parcel in) {
            setExtra(ParcelUtils.readFromParcel(in));
            setContent(ParcelUtils.readFromParcel(in));
        }
    ```

您已成功创建自定义消息类型。后续步骤如下：

1. 完善消息注解 `@MessageTag`。如果需要实现对消息内容的自定义处理，您需要创建自定义的 `messageHandler`。
1. 注册自定义消息类型，SDK 才能识别并收发该类消息。

### 如何添加消息注解 {#messagetag}

SDK 内置消息类型默认已带有 [MessageTag]。如果创建自定义消息类型，必须使用 `@MessageTag` 添加消息注解。

`@MessageTag` 中需要指定以下参数：

| 参数           | 类型                              | 描述                                                                                                                                                                             |
|:---------------|:----------------------------------|:-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| value          | String                            | （必要参数）消息类型唯一标识，例如 `"app:txtcontent`。为尽量减少对消息体积的影响，建议控制在 16 字符以内。**注意**，请不要以 `RC:` 开头（`RC:` 为官方保留前缀），否则会和融云内置消息冲突。 |
| flag           | int                               | （必要参数）消息的存储与计数属性。传入值详见下方 **Flag 说明**。**注意**，客户端与服务端的存储行为均会受该属性的影响。                                                                 |
| messageHandler | `Class<? extends MessageHandler>` | （可选参数）默认使用 SDK 默认的 `messageHandler`。在自定义媒体消息类型的情况下，如果 `encode` 后的大小超过 128 KB，您需要使用自定义的 `messageHandler`。                               |

- **`flag` 参数说明**：

    | 存储计数属性           | 客户端是否存储                | 服务端是否存储                                                                                 | 适用场景                                            |
    |:-----------------------|:-----------------------|:-----------------------------------------------------------------------------------------|:----------------------------------------------------|
    | MessageTag.NONE        | 客户端不存储、不计入未读消息数 | 支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制                      | 不需要展示的消息。例如，运营平台向终端发送的指令消息。 |
    | MessageTag.ISCOUNTED   | 客户端存储、计入未读消息数     | 支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制，且存入服务端历史消息 | ---                                                 |
    | MessageTag.ISPERSISTED | 客户端存储，不计入未读消息数   | 支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制，且存入服务端历史消息 | ---                                                 |
    | MessageTag.STATUS      | 客户端不存储，不计入未读消息数 | 服务端不存储                                                                                   | 用于传递即时状态的消息，无法支持消息推送。            |

    :::tip

    - `MessageTag.NONE` 一般用于需要确保收到，但不需要展示的消息，例如运营平台向终端发送的指令信息。如果消息接收方不在线，再次上线时可通过离线消息收到。
    - `MessageTag.STATUS` 用于状态消息。状态消息表示的是即时的状态，例如输入状态。因为状态消息在客户端与服务端均不会存储，如果接收方不在线，则无法再收到该状态消息。
    :::


- **代码示例**

    ```java
    @MessageTag(value = "appx:MyTextContent", flag = MessageTag.ISCOUNTED, )
    class MyTextContent extend MessageContent {

    }
    ```

### 自定义媒体消息类型的处理方式

`MessageTag` 中定义了 `messageHandler`，支持自定义处理消息内容，例如文件的压缩等操作。

- 如果不指定 `messageHandler`，SDK 会默认使用 `DefaultMessageHandler`。一般情况下，您不需要指定  `messageHandler`，
- 如果希望自定义处理消息内容，您需要创建 `messageHandler` 并继承 [MessageHandler]。在自定义媒体消息类型的情况下，如果 `encode` 后的大小超过 128 KB，您需要使用自定义的 `messageHandler`。

以下示例中自定义的 `MyMediaHandler` 继承了 [MessageHandler]。其中 `MyMediaMessageContent` 为自定义的媒体消息内容。

```java
public class MyMediaHandler extends MessageHandler<MyMediaMessageContent> {

    public MyMediaHandler(Context context) {
        super(context);
    }

    /**
     * 解码 MessageContent 到 Message 中。
     *
     * @param message 用于存放 MessageContent 的消息实体。
     * @param content 将要被解码的 MessageContent。
     */
    @Override
    public void decodeMessage(Message message, MyMediaMessageContent model) {

    }

    /**
     * 对 Message 编码。
     *
     * @param message 将要被编码的 Message 实体。
     */
    @Override
    public void encodeMessage(Message message) {

    }
}
```

## 注册自定义消息

您需要在建立 IM 连接之前调用 [registerMessageType] 注册自定义消息类型，SDK 才能识别该类消息。否则消息将无法识别，SDK 会按照 `UnknownMessage` 处理。


以注册自定义消息 `MyTextContent.class`、`MyMessage.class` 为例：

```java
ArrayList<Class<? extends MessageContent>> myMessages = new ArrayList<>();
myMessages.add(MyTextContent.class);
myMessages.add(MyMessage.class);
RongIMClient.registerMessageType(myMessages);
```

| 参数                    | 类型                                     | 说明                |
|:------------------------|:-----------------------------------------|:------------------|
| messageContentClassList | List\<Class\<? extends MessageContent\>> | 自定义消息的类列表。 |

## 发送自定义消息

自定义消息类型可直接使用发送内置消息类型的发送方法进行消息的发送。

- 如果自定义消息类型继承 `MessageContent`，请使用发送普通消息的接口发送。
- 如果自定义消息类型继承 `MediaMessageContent`，请使用发送媒体消息的接口发送。

如果自定义消息类型需要支持推送，必须在发送自定义消息时额外指定推送内容（`pushContent`）。推送内容在接收方收到推送时显示在通知栏中。

- 在发送消息时，可直接通过 `pushContent` 参数指定推送内容。
- 您也可以通过设置 [Message] 的 `MessagePushConfig` 中的 `pushContent` 及其他字段，对消息的推送进行个性化配置。优先使用 `MessagePushConfig` 中的配置。

发送消息的具体方法与配置方式，请参考以下文档：

- **App 仅集成 IMLib SDK**：[发送消息][imlib-发送消息]（单聊、群聊、聊天室）、[收发消息]（超级群）
- **App 集成 IMKit SDK**：[发送消息][imkit-发送消息]（单聊、群聊、聊天室）

:::tip

 - 如果融云服务端无法获取自定义消息的 `pushContent`，则无法触发消息推送。例如，在接收方在离线等情况无法收到消息推送通知。
 - 如果自定义的消息类型为**状态消息**（见[如何添加消息注解](#messagetag)），则无法支持推送，不需要额外指定推送内容。
:::


## 代码示例

### 示例：自定义普通消息类型

```java
@MessageTag(value = "app:txtcontent", flag = MessageTag.ISCOUNTED)
public class MyTextContent extends MessageContent {

    private static final String TAG = "MyTextContent";
    // 自定义消息变量，可以有多个
    private String content;

    private MyTextContent() {}

    /**
    * 设置文字消息的内容。
    *
    * @param content 文字消息的内容。
    */
    public void setContent(String content) {
        this.content = content;
    }

    /**
        * 构造函数。
        *
        * @param in 初始化传入的 Parcel。
        */
    public MyTextContent(Parcel in) {
        setExtra(ParcelUtils.readFromParcel(in));
        setContent(ParcelUtils.readFromParcel(in));
    }

    // 快速构建消息对象方法
    public static MyTextContent obtain(String content) {
        MyTextContent msg = new MyTextContent();
        msg.content = content;
        return msg;
    }

    /** 创建 MyTextContent(byte[] data) 带有 byte[] 的构造方法用于解析消息内容. */
    public MyTextContent(byte[] data) {
        if (data == null) {
            return;
        }
        String jsonStr = null;
        try {
            jsonStr = new String(data, "UTF-8");
        } catch (UnsupportedEncodingException e) {
            Log.e(TAG, "UnsupportedEncodingException ", e);
        }
        if (jsonStr == null) {
            Log.e(TAG, "jsonStr is null ");
            return;
        }

        try {
            JSONObject jsonObj = new JSONObject(jsonStr);
            // 消息携带用户信息时, 自定义消息需添加下面代码
            if (jsonObj.has("user")) {
                setUserInfo(parseJsonToUserInfo(jsonObj.getJSONObject("user")));
            }
            // 用于群组聊天, 消息携带 @ 人信息时, 自定义消息需添加下面代码
            if (jsonObj.has("mentionedInfo")) {
                setMentionedInfo(parseJsonToMentionInfo(jsonObj.getJSONObject("mentionedInfo")));
            }
            // 将所有自定义变量从收到的 json 解析并赋值
            if (jsonObj.has("content")) {
                content = jsonObj.optString("content");
            }
        } catch (JSONException e) {
            Log.e(TAG, "JSONException " + e.getMessage());
        }
    }

    public String getContent() {
        return content;
    }

    /**
     * 将本地消息对象序列化为消息数据。
     *
     * @return 消息数据。
     */
    @Override
    public byte[] encode() {
        JSONObject jsonObj = new JSONObject();
        try {
            // 消息携带用户信息时, 自定义消息需添加下面代码
            if (getJSONUserInfo() != null) {
                jsonObj.putOpt("user", getJSONUserInfo());
            }
            // 用于群组聊天, 消息携带 @ 人信息时, 自定义消息需添加下面代码
            if (getJsonMentionInfo() != null) {
                jsonObj.putOpt("mentionedInfo", getJsonMentionInfo());
            }
            //  将所有自定义消息的内容，都序列化至 json 对象中
            jsonObj.put("content", this.content);
        } catch (JSONException e) {
            Log.e(TAG, "JSONException " + e.getMessage());
        }

        try {
            return jsonObj.toString().getBytes("UTF-8");
        } catch (UnsupportedEncodingException e) {
            Log.e(TAG, "UnsupportedEncodingException ", e);
        }
        return null;
    }

    @Override
    public int describeContents() {
        return 0;
    }

    @Override
    public void writeToParcel(Parcel dest, int i) {
        // 对消息属性进行序列化，将类的数据写入外部提供的 Parcel 中
        ParcelUtils.writeToParcel(dest, getExtra());
        ParcelUtils.writeToParcel(dest, content);
    }

    public static final Creator<MyTextContent> CREATOR =
            new Creator<MyTextContent>() {
                public MyTextContent createFromParcel(Parcel source) {
                    return new MyTextContent(source);
                }

                public MyTextContent[] newArray(int size) {
                    return new MyTextContent[size];
                }
            };
}
```

### 示例：自定义媒体消息类型

:::tip

 除非不使用 SDK 内置上传逻辑，否则 JSON 的 `localPath` 属性必须有值。
:::


```java
@MessageTag(value = "app:mediacontent", flag = MessageTag.ISCOUNTED)
public class MyMediaMessageContent extends MediaMessageContent {

    /** 读取接口，目的是要从 Parcel 中构造一个实现了 Parcelable 的类的实例处理。 */
    public static final Creator<MyMedia> CREATOR =
            new Creator<MyMediaMessage>() {

                @Override
                public MyMediaMessageContent createFromParcel(Parcel source) {
                    return new MyMediaMessageContent(source);
                }

                @Override
                public MyMediaMessageContent[] newArray(int size) {
                    return new MyMediaMessageContent[size];
                }
            };


    public MyMediaMessageContent(byte[] data) {
        String jsonStr = new String(data);

        try {
            JSONObject jsonObj = new JSONObject(jsonStr);

            if (jsonObj.has("localPath")) {
                setLocalPath(Uri.parse(jsonObj.optString("localPath")));
            }

        } catch (JSONException e) {
            Log.e("JSONException", e.getMessage());
        }
    }


    /**
     * 构造函数。
     *
     * @param in 初始化传入的 Parcel。
     */
    public MyMediaMessageContent(Parcel in) {
        setLocalPath(ParcelUtils.readFromParcel(in, Uri.class));
    }


    public MyMediaMessageContent(Uri localUri) {
        setLocalPath(localUri);
    }

    /**
     * 生成 MyMediaMessageContent 对象。
     *
     * @param localUri 媒体文件地址。
     * @return MyMediaMessageContent 对象实例。
     */
    public static MyMediaMessageContent obtain(Uri localUri) {
        return new MyMediaMessageContent(localUri);
    }

    @Override
    public byte[] encode() {
        JSONObject jsonObj = new JSONObject();

        try {

            if (getLocalUri() != null) {
                /** 除非不使用 SDK 内置上传逻辑，否则 JSON 的 `localPath` 属性必须有值。 */
                jsonObj.put("localPath", getLocalUri().toString());
            }
        } catch (JSONException e) {
            RLog.e("JSONException", e.getMessage());
        }
        return jsonObj.toString().getBytes();
    }

    /**
     * 获取本地图片地址（file:///）。
     *
     * @return 本地图片地址（file:///）。
     */
    public Uri getLocalUri() {
        return getLocalPath();
    }

    /**
     * 设置本地图片地址（file:///）。
     *
     * @param localUri 本地图片地址（file:///）.
     */
    public void setLocalUri(Uri localUri) {
        setLocalPath(localUri);
    }

    /**
     * 描述了包含在 Parcelable 对象排列信息中的特殊对象的类型。
     *
     * @return 一个标志位，表明Parcelable对象特殊对象类型集合的排列。
     */
    @Override
    public int describeContents() {
        return 0;
    }

    /**
     * 将类的数据写入外部提供的 Parcel 中。
     *
     * @param dest 对象被写入的 Parcel。
     * @param flags 对象如何被写入的附加标志，可能是 0 或 PARCELABLE_WRITE_RETURN_VALUE。
     */
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        ParcelUtils.writeToParcel(dest, getLocalPath());
    }
}
```

<!-- links -->
[MessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message-content/
[MediaMessageContent]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/MediaMessageContent.html
[TextMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-text-message/
[ImageMessage]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/-image-message/
[LocationMessage]: https://www.rongcloud.cn/docs/api/android/im_v5/latest/location/io/rong/imlib/location/message/LocationMessage.html
[MessageTag]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-message-tag/index.html
[Message]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib.model/-message/index.html
[MessageHandler]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.message/MessageHandler.html
[imlib-发送消息]: ../message/send.md
[收发消息]: ../ultragroup/chat.md
[imkit-发送消息]: /android-imkit/customization/message-send
[内置消息类型]: /platform-chat-api/message-about/objectname
[registerMessageType]: https://doc.rongcloud.cn/apidoc/imlibcore-android/latest/zh_CN/html/-android--i-m-lib-core--s-d-k/io.rong.imlib/-rong-core-client/register-message-type.html

