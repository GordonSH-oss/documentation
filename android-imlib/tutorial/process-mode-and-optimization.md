---
title: 进程模式与性能优化指南
sidebar_position: 4
---


IMLib SDK 支持单进程和多进程两种模式，默认使用多进程模式。单进程模式下，IMLib SDK 和 App 运行在同一个进程中；多进程模式下，IMLib SDK 运行在独立的进程中。

**单进程模式的特点：**
- 所有组件在同一进程中，对象可以直接引用，无需序列化/反序列化，便于调试和跟踪。
- 无进程间通信开销，组件间直接调用，延迟低。
- 出现崩溃、内存泄漏以及 ANR 时会影响全局，内存上限受限于单进程。

**多进程模式的特点：**
- 组件间调用需要序列化/反序列化，延迟高，调试和跟踪不便。
- 组件间相互独立，内存隔离，崩溃、内存泄漏以及 ANR 影响局部。
- 可以突破单进程内存上限限制，充分利用多核 CPU 资源。

Tips：目前大多数手机设备的内存分配对于一般 App 的使用场景基本够用，在单进程下运行通常不会出现内存不足的问题。

### 内存使用对比

| 指标 | 单进程 | 多进程 |
|------|--------|--------|
| 基础内存 | 低 | 高 |
| 内存隔离 | 无 | 有 |
| 内存泄漏影响 | 全局 | 局部 |
| 内存上限 | 512MB-1GB | 可突破限制 |

### 性能对比

| 指标 | 单进程 | 多进程 |
|------|--------|--------|
| 启动速度 | 快 | 慢 |
| 响应延迟 | 低 | 中等 |
| 并发处理 | 差 | 好 |
| 资源利用 | 中等 | 高 |

### 稳定性对比

| 指标 | 单进程 | 多进程      |
|------|--------|----------|
| 崩溃影响 | 全局 | 局部，不影响主进程 |
| ANR影响 | 全局 | 局部 |
| 内存泄漏 | 全局 | 局部 |
| 进程恢复 | 简单 | 复杂       |


## App 在后台时 SDK 的限制问题

### 连接状态限制
- 网络连接: 后台时网络连接可能被系统限制或断开。
- 心跳机制: 心跳包发送频率受限，可能导致连接超时。
- 重连机制: 自动重连任务可能被系统阻止或延迟。

### 电池优化限制
- Doze 模式: 深度睡眠模式限制网络和CPU
- App Standby: 应用待机模式限制后台活动

### 通知和推送

当 App 退到后台时，系统针对后台还没有开始受到限制 Socket 也都属于是连接状态，如果收到消息会走融云的在线通知。 正常 30s 到 2 分钟之间 App 的各方面都会受到系统限制（部分机型限制后台运行后，退到后台会立刻限制进程运行或者网络出访，导致 Socket 会出现退到后台立马断开的情况），也会把耗电的 Socket 以及相关任务线程都给停掉，属于挂起状态。直到 App 在后台后进程被杀掉，才会收到远程推送。

综上所述，在后台时一共有三个状态：
- 1.刚进入后台（来消息时会收到融云的在线通知） 。
- 2.进程挂起（来消息时走离线推送） 。
- 3.进程 kill（来消息时走离线推送）。

## 调用 IM SDK 接口出现的内存泄露问题解决方案

使用静态内部类时不会持有外部类引用，WeakReference 允许垃圾回收器在 Activity 销毁后回收对象，避免内存泄露问题。

### 示例代码

```java
public class TestActivity extends AppCompatActivity {
    
    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_test);

        // 使用 WeakReference 避免内存泄露
        WeakReference<AddFriendActivity> activityRef = new WeakReference<>(this);
        RongCoreClient.getInstance().getConversation(Conversation.ConversationType.PRIVATE, "TargetId", new TestConversationCallback(activityRef));
    }


    /**
     * 静态内部类回调，避免持有外部类引用
     */
    private static class TestConversationCallback extends IRongCoreCallback.ResultCallback<Conversation> {
        private final WeakReference<AddFriendActivity> activityRef;

        public ConversationCallback(WeakReference<AddFriendActivity> activityRef) {
            this.activityRef = activityRef;
        }

        @Override
        public void onSuccess(Conversation conversation) {
            AddFriendActivity activity = activityRef.get();
            if (activity != null && !activity.isFinishing() && !activity.isDestroyed()) {
                // 处理成功回调
                // 可以在这里添加具体的业务逻辑
            }
        }

        @Override
        public void onError(IRongCoreEnum.CoreErrorCode coreErrorCode) {
            AddFriendActivity activity = activityRef.get();
            if (activity != null && !activity.isFinishing() && !activity.isDestroyed()) {
                // 处理错误回调
                // 可以在这里添加具体的错误处理逻辑
            }
        }
    }
}
```
















