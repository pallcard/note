# LockSupport

## 重要方法
### park
```java
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker); // 使用Unsafe设置线程Blocker, 设置当前线程阻塞的原因，可以方便调试（线程在哪个对象上阻塞了）
        UNSAFE.park(false, 0L);
        setBlocker(t, null); // 设置Blocker为null
    }

    // 阻塞线程,并且该线程在下列情况发生之前都会被阻塞：
    // ① 调用unpark函数，释放该线程的许可。
    // ② 该线程被中断。
    // ③ 设置的时间到了。
    public native void park(boolean var1, long var2);
```
### unpark
```java
    public static void unpark(Thread thread) {
        if (thread != null)
            UNSAFE.unpark(thread);
    }
```
说明：此函数表示如果给定线程的许可尚不可用，则使其可用。如果线程在 park 上受阻塞，则它将解除其阻塞状态。否则，保证下一次调用 park 不会受阻塞。如果给定线程尚未启动，则无法保证此操作有任何效果。


## 参考



https://www.cnblogs.com/leesf456/p/5347293.html
