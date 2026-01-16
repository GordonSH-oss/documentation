---
title: 自定义消息类型
sidebar_position: 110
---

# 自定义消息类型

> **重要**
>
> 本文适用于 SDK 5.6.7 及之后版本。如果您的 SDK 版本 \< 5.6.7，请使用[自定义消息（旧版）](./customize-deprecated.md)。

:::tip

 SDK 5.6.7 版本开始，`MessageContent.java` 中封装了 `user`（用户信息），`mentionedInfo`（@人信息） ，`extra`（附加信息）字段的编解码及序列化方法，实现自定义消息时可直接调用父类的解析方法，无需开发者自行解析。新旧版本自定义消息可兼容互通。已使用旧版自定义消息客户，如需新增自定义消息类型，推荐使用新版自定义消息。
:::

除了使用 IMLib SDK 内置消息外，还可以创建自定义消息类型。

您需要根据业务需求来选择自定义消息类型继承的消息基类：

- [MessageContent]：即普通类型消息内容。例如在 SDK 内置消息类型中的文本消息和位置消息。
- [MediaMessageContent]：即媒体类型消息。媒体类型消息内容继承自 [MessageContent]，并在其基础上增加了对多媒体文件的处理逻辑。在发送和接收消息时，SDK 会判断消息类型是否为多媒体类型消息，如果是多媒体类型，则会触发上传或下载多媒体文件流程。

关于消息实体类与消息内容的更多介绍，可参见[消息介绍](./introduction.md)。


:::tip

 自定义消息的类型、消息结构需要确保多端一致，否则将出现无法互通的问题。
:::


## 创建自定义消息

SDK 不负责定义和解析自定义消息的具体内容，您需要自行实现。

自定义消息类型的 `@MessageTag`（[MessageTag]）决定该消息类型的唯一标识（`objectname`），以及是否存储、是否展示、是否计入消息未读数等属性。

### 自定义消息示例代码：普通消息

以下为自定义普通消息的完整示例代码。

```java
// 1. 自定义消息实现 MessageTag 注解
@MessageTag(value = "app:txtcontent", flag = MessageTag.ISCOUNTED)
public class MyTextContent extends MessageContent {
    // <editor-fold desc="* 2. 自身内部变量，可以有多个">
    private static final String TAG = "appTextContent";
    private String content;
    // </editor-fold>

    // <editor-fold desc="* 3. 对外构造方法">
    private MyTextContent() {}

    // 快速构建消息对象方法
    public static MyTextContent obtain(String content) {
        MyTextContent msg = new MyTextContent();
        msg.content = content;
        return msg;
    }
    // </editor-fold>

    // <editor-fold desc="* 4. 二进制 Encode & decode 编解码方法">

    /**
     * 将本地消息对象序列化为消息数据。
     *
     * @return 消息数据。
     */
    @Override
    public byte[] encode() {
        //此处以需要携带 “用户信息” 或者 “@ 人” 信息为例
        JSONObject jsonObj = super.getBaseJsonObject();
        try {
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
            //此处以需要携带 “用户信息” 或者 “@ 人” 信息为例
            super.parseBaseJsonObject(jsonObj);
            // 将所有自定义变量从收到的 json 解析并赋值
            if (jsonObj.has("content")) {
                content = jsonObj.optString("content");
            }
        } catch (JSONException e) {
            Log.e(TAG, "JSONException " + e.getMessage());
        }
    }

    // </editor-fold>

    // <editor-fold desc="* 5. Parcel 的序列化方法">

    @Override
    public void writeToParcel(Parcel dest, int i) {
        // 对消息属性进行序列化，将类的数据写入外部提供的 Parcel 中，此处以需要序列化"用户信息"，"@信息" 等为例
        super.writeToBaseInfoParcel(dest);
        ParcelUtils.writeToParcel(dest, content);
    }
    /**
     * 构造函数。
     *
     * @param in 初始化传入的 Parcel。
     */
    public MyTextContent(Parcel in) {
//      此处以需要序列化"用户信息"，"@信息" 等为例
        super.readFromBaseInfoParcel(in);
        setContent(ParcelUtils.readFromParcel(in));
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
    @Override
    public int describeContents() {
        return 0;
    }
    // </editor-fold>

    // <editor-fold desc="* 6. get & set 方法">
    /**
     * 设置文字消息的内容。
     *
     * @param content 文字消息的内容。
     */
    public void setContent(String content) {
        this.content = content;
    }

    public String getContent() {
        return content;
    }
    // </editor-fold>

}
```

### 详解示例代码

以下讲解自定义普通消息的示例代码。创建一个 `MessageContent` 的子类。

1. 自定义消息类型，必须使用 `@MessageTag` 添加消息注解。

    `@MessageTag` 中需要指定以下参数：

    | 参数           | 类型                              | 描述                                                                                                                                                                            |
    |:---------------|:----------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | value          | String                            | （必要参数）消息类型唯一标识，例如 `app:txtcontent`。为尽量减少对消息体积的影响，建议控制在 16 字符以内。**注意**，请不要以 `RC:` 开头（`RC:` 为官方保留前缀），否则会和融云内置消息冲突。 |
    | flag           | int                               | （必要参数）消息的存储与计数属性。传入值详见下方 **Flag 说明**。**注意**，客户端与服务端的存储行为均会受该属性的影响。                                                                |
    | messageHandler | `Class<? extends MessageHandler>` | （可选参数）默认使用 SDK 默认的 `messageHandler`。在自定义媒体消息类型的情况下，如果 `encode` 后的大小超过 128 KB，您需要使用自定义的 `messageHandler`。                              |

    **`flag` 参数**影响客户端与服务端的存储等行为，具体说明如下：

    | flag 取值                | 适用场景                                                                                                                                                                                                                                |
    |:-------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
    | `MessageTag.NONE`        | 此类消息不会保存到客户端本地消息数据库，也不会记录未读数。支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制。一般用作命令消息等不需要展示的消息。例如，运营平台向终端发送的指令消息，通知端上执行一个动作。            |
    | `MessageTag.ISPERSISTED` | 此类消息会保存到客户端本地消息数据库，但是不记录未读数。支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制，且存入服务端历史消息。常用于小灰条类型的消息，需要 UI 展示，但不需要增加未读数。                            |
    | `MessageTag.ISCOUNTED`   | 此类消息会保存到客户端本地消息数据库，并且增加未读数。支持离线消息<sup><a href="/guides/glossary/imglossary#offline">?</a></sup>机制，且存入服务端历史消息。如文本，图片等消息均为此类。                                                           |
    | `MessageTag.STATUS`      | 状态消息在客户端与服务端均不会存储，也不会记录未读数，仅用于传递即时状态的消息，例如发送输入状态。对方在线能收到该消息；对方不在线，服务器会直接丢弃该消息，不会进行推送。因此如果接收方不在线，则无法再收到该状态消息；卸载重装后也不会重新下发。 |

2. 按需增加内部变量，可以有多个。示例代码中的 `content` 字段即为自定义消息的内部变量，用于存放消息内容。

    ```java
    private String content;
    ```

3. 实现对外构造方法，提供给其他类调用。

    ```java
    // 快速构建消息对象方法
    public static MyTextContent obtain(String content) {
        MyTextContent msg = new MyTextContent();
        msg.content = content;
        return msg;
    }
    ```

4. 实现二进制编解码方法，用于二进制数据和消息对象的相互转换，一共有两个方法（Encode 和 Decode）。

    :::tip

    父类 `MessageContent.java` 中封装了 `user`（用户信息），`mentionedInfo`（@人信息） ，`extra`（附加信息）字段的编解码及序列化方法。自定义消息如需携带以上信息，可参考示例代码注释调用父类方法进行解析。
    :::


    1. 重写父类的 `encode` 方法，将消息对象转为 `byte[]`。

        具体流程 ：消息对象 => JSONObject => JSON String  => byte[]

        ```java
        /**
             * 将本地消息对象序列化为消息数据。
             *
             * @return 消息数据。
             */
            @Override
            public byte[] encode() {

                // 调用父类接口将基础字段转为 JSONObject
                JSONObject jsonObj = super.getBaseJsonObject();
                try {
                    // 将所有自定义消息的内部变量，都序列化至 json 对象中
                    jsonObj.put("content", this.content);
                } catch (JSONException e) {
                    Log.e(TAG, "JSONException " + e.getMessage());
                }

                try {
                    // 将 jsonObj 转为二进制
                    return jsonObj.toString().getBytes("UTF-8");
                } catch (UnsupportedEncodingException e) {
                    Log.e(TAG, "UnsupportedEncodingException ", e);
                }
                return null;
            }
        ```

    2. 重写父类的 `MessageContent(byte[] data)` 构造方法，以实现对自定义消息内容的解析。

        解析流程（Decode）与编码流程完全相反，具体流程为：byte[] => JSON String => JSONObject => 消息对象

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

                //需要调用父类的 parseBaseJsonObjec() 方法，将基础字段解析出来
                super.parseBaseJsonObject(jsonObj);

                // 在将所有自定义变量从 json 解析并赋值
                if (jsonObj.has("content")) {
                    content = jsonObj.optString("content");
                }
            } catch (JSONException e) {
                Log.e(TAG, "JSONException " + e.getMessage());
            }
        }
        ```

5. 实现 Parcel 的序列化方法，这一组方法用于消息对象跨进程传输，总共有四个方法。

    使用 SDK 的 ParcelUtils 工具类实现 MyTextContent 的序列化与反序列化。

    > **重要**
    >
    > - 父类 `MessageContent.java` 中封装了 `user`（用户信息），`mentionedInfo`（@人信息） ，`extra`（附加信息）字段的编解码及序列化方法。自定义消息如需携带以上信息，可参考示例代码注释调用父类方法进行序列化。
    > - 序列化中的 Parcel 的读写个数和顺序一定要一一对应。

    ```java

        /**
         * 将类的数据写入外部提供的 Parcel 中。
         *
         * @param dest 对象被写入的 Parcel。
         * @param flags 对象如何被写入的附加标志，可能是 0 或 PARCELABLE_WRITE_RETURN_VALUE。
         */
        @Override
        public void writeToParcel(Parcel dest, int flags) {
            // 将父类变量写入
            super.writeToBaseInfoParcel(dest);
            // 将内部变量写入
            ParcelUtils.writeToParcel(dest, content);
        }

        /**
         * 构造函数。
         *
         * @param in 初始化传入的 Parcel。
         */
        public MyTextContent(Parcel in) {
            // 读取父类变量
            super.readFromBaseInfoParcel(in);
            // 读取内部变量
            setContent(ParcelUtils.readFromParcel(in));
        }

        /**
         * 描述了包含在 Parcelable 对象排列信息中的特殊对象的类型。
         *
         * @return 一个标志位，表明 Parcelable 对象特殊对象类型集合的排列。
         */
        public int describeContents() {
            return 0;
        }

       /** 读取接口，目的是要从 Parcel 中构造一个实现了 Parcelable 的类的实例处理。 */
        public static final Creator<MyTextContent> CREATOR =
                new Creator<MyTextContent>() {

                    @Override
                    public TextMessage createFromParcel(Parcel source) {
                        return new MyTextContent(source);
                    }

                    @Override
                    public TextMessage[] newArray(int size) {
                        return new MyTextContent[size];
                    }
                };
    ```

6. 实现自定义内部变量的 Getter 和 Setter 方法。

    ```java
        /**
         * 设置文字消息的内容。
         *
         * @param content 文字消息的内容。
         */
        public void setContent(String content) {
            this.content = content;
        }

        public String getContent() {
            return content;
        }
    ```

### 自定义消息示例代码：媒体消息

:::tip

 除非不使用 SDK 内置上传逻辑，否则 JSON 的 `localPath` 属性必须有值。
:::


```java
// 1. 自定义消息实现 MessageTag 注解
@MessageTag(value = "app:mediacontent", flag = MessageTag.ISCOUNTED)
public class MyMediaMessageContent extends MediaMessageContent {
    // <editor-fold desc="* 2. 自身内部变量，可以有多个，媒体类消息可以直接用 MediaMessageContent 的 localPath ">

    // </editor-fold>
    // <editor-fold desc="* 3. 对外构造方法">
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
    // </editor-fold>

    // <editor-fold desc="* 4. 二进制 Encode & decode 编解码方法">

    @Override
    public byte[] encode() {
        //此处以需要携带 “用户信息” 或者 “@ 人” 信息为例
        JSONObject jsonObj = super.getBaseJsonObject();

        try {

            if (getLocalUri() != null) {
                /** 除非不使用 SDK 内置上传逻辑，否则 JSON 的 `localPath` 属性必须有值。 */
                jsonObj.put("localPath", getLocalUri().toString());
            }
        } catch (JSONException e) {
            Log.e("JSONException", e.getMessage());
        }
        return jsonObj.toString().getBytes();
    }

    public MyMediaMessageContent(byte[] data) {
        String jsonStr = new String(data);

        try {
            JSONObject jsonObj = new JSONObject(jsonStr);
            //此处以需要携带 “用户信息” 或者 “@ 人” 信息为例
            super.parseBaseJsonObject(jsonObj);
            if (jsonObj.has("localPath")) {
                setLocalPath(Uri.parse(jsonObj.optString("localPath")));
            }

        } catch (JSONException e) {
            Log.e("JSONException", e.getMessage());
        }
    }
    // </editor-fold>

    // <editor-fold desc="* 5. Parcel 的序列化方法">

    /**
     * 将类的数据写入外部提供的 Parcel 中。
     *
     * @param dest 对象被写入的 Parcel。
     * @param flags 对象如何被写入的附加标志，可能是 0 或 PARCELABLE_WRITE_RETURN_VALUE。
     */
    @Override
    public void writeToParcel(Parcel dest, int flags) {
        // 对消息属性进行序列化，将类的数据写入外部提供的 Parcel 中，此处以需要序列化"用户信息"，"@信息" 等为例
        super.writeToBaseInfoParcel(dest);
        ParcelUtils.writeToParcel(dest, getLocalPath());
    }

    /**
     * 构造函数。
     *
     * @param in 初始化传入的 Parcel。
     */
    public MyMediaMessageContent(Parcel in) {
        // 此处以需要序列化"用户信息"，"@信息" 等为例
        super.readFromBaseInfoParcel(in);
        setLocalPath(ParcelUtils.readFromParcel(in, Uri.class));
    }

    /** 读取接口，目的是要从 Parcel 中构造一个实现了 Parcelable 的类的实例处理。 */
    public static final Creator<MyMediaMessageContent> CREATOR =
            new Creator<MyMediaMessageContent>() {

                @Override
                public MyMediaMessageContent createFromParcel(Parcel source) {
                    return new MyMediaMessageContent(source);
                }

                @Override
                public MyMediaMessageContent[] newArray(int size) {
                    return new MyMediaMessageContent[size];
                }
            };

    /**
     * 描述了包含在 Parcelable 对象排列信息中的特殊对象的类型。
     *
     * @return 一个标志位，表明Parcelable对象特殊对象类型集合的排列。
     */
    @Override
    public int describeContents() {
        return 0;
    }

    // </editor-fold>

    // <editor-fold desc="* 6. get & set 方法">

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
    // </editor-fold>
}

```


## 注册自定义消息

您需要在建立 IM 连接之前调用 [registerMessageType] 注册自定义消息类型，SDK 才能识别该类消息。否则消息将无法识别，SDK 会按照 `UnknownMessage` 处理。

:::tip

 必须在连接之前注册自定义消息。建议在应用生命周期内调用。
:::


以下以注册自定义消息 `MyTextContent.class`、`MyMessage.class` 为例：

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

自定义消息类型可直接使用发送内置消息类型的方法。请注意根据当前使用的 SDK、业务、消息类型选择合适的核心类与方法：

- 如果自定义消息类型继承 `MessageContent`，请使用发送普通消息的接口发送。
- 如果自定义消息类型继承 `MediaMessageContent`，请使用发送媒体消息的接口发送。

如果自定义消息类型需要支持推送，必须在发送自定义消息时额外指定推送内容（`pushContent`）。推送内容在接收方收到推送时显示在通知栏中。

- 在发送消息时，可直接通过 `pushContent` 参数指定推送内容。
- 您也可以通过设置 [Message] 的 `MessagePushConfig` 中的 `pushContent` 及其他字段，对消息的推送进行个性化配置。优先使用 `MessagePushConfig` 中的配置。

发送消息的具体方法与配置方式，请参考以下文档：

- **App 仅集成 IMLib SDK**：[发送消息][imlib-发送消息]（单聊、群聊<!--public-cloud-only start-->、聊天室）、[收发消息]（超级群<!--public-cloud-only end-->）
- **App 集成 IMKit SDK**：[发送消息][imkit-发送消息]（单聊、群聊<!--public-cloud-only start-->、聊天室<!--public-cloud-only end-->）

:::tip

 - 如果融云服务端无法获取自定义消息的 `pushContent`，则无法触发消息推送。例如，在接收方在离线等情况无法收到消息推送通知。
 - 如果自定义的消息类型为**状态消息**，则无法支持推送，不需要额外指定推送内容。
:::


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

