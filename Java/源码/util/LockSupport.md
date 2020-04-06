# LockSupport

## 重要方法
### park
```
    public static void park(Object blocker) {
        Thread t = Thread.currentThread();
        setBlocker(t, blocker); // 使用Unsafe设置线程Blocker, 设置当前线程阻塞的原因，可以方便调试（线程在哪个对象上阻塞了）
        UNSAFE.park(false, 0L);
        setBlocker(t, null); // 设置Blocker为null
    }

    

```
