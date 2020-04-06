# Thread

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/05/1586097355299-1586097355309.png)

## 源码

### 定义
```java
public class Thread implements Runnable {
    private volatile String name;      // 线程名称
    private int            priority;   // 优先级，1~10
    private Thread         threadQ;    // 实际被执行的对象
    private long           eetop;

    private boolean     daemon = false; // 是否为守护线程

    // Java thread status for tools,initialized to indicate thread 'not yet started'
    private volatile int threadStatus = 0;

    /* What will be run. */
    private Runnable target;   // 实际被执行的对象

    /* The group of this thread */
    private ThreadGroup group;

    /* ThreadLocal values pertaining to this thread. This map is maintained
     * by the ThreadLocal class. */
    ThreadLocal.ThreadLocalMap threadLocals = null;   // 当前线程的数据

    /* Whether or not to single_step this thread. */
    private boolean     single_step;

    /* JVM state */
    private boolean     stillborn = false;

    /* The context ClassLoader for this thread */
    private ClassLoader contextClassLoader;// 这个线程的上下文类加载器

    /* The inherited AccessControlContext of this thread */
    private AccessControlContext inheritedAccessControlContext;

    /*
     * InheritableThreadLocal values pertaining to this thread. This map is
     * maintained by the InheritableThreadLocal class.
     */
    ThreadLocal.ThreadLocalMap inheritableThreadLocals = null;

    /*
     * The requested stack size for this thread, or 0 if the creator did
     * not specify a stack size.  It is up to the VM to do whatever it
     * likes with this number; some VMs will ignore it.
     */
    private long stackSize;

    /*
     * JVM-private state that persists after native thread termination.
     */
    private long nativeParkEventPointer;

    /*
     * Thread ID
     */
    private long tid; // 线程id

    /* For generating thread ID */
    private static long threadSeqNumber;

    volatile Object parkBlocker; // 线程的阻塞对象，（线程阻塞在那个对象上了）

    /* The object in which this thread is blocked in an interruptible I/O
     * operation, if any.  The blocker's interrupt method should be invoked
     * after setting this thread's interrupt status.
     */
    private volatile Interruptible blocker;

...

}
```

### 重要方法

#### start
```java
    public synchronized void start() {
        /**
         * This method is not invoked for the main method thread or "system"
         * group threads created/set up by the VM. Any new functionality added
         * to this method in the future may have to also be added to the VM.
         *
         * A zero status value corresponds to state "NEW".
         */
        if (threadStatus != 0)
            throw new IllegalThreadStateException();

        /* Notify the group that this thread is about to be started
         * so that it can be added to the group's list of threads
         * and the group's unstarted count can be decremented. */
        group.add(this); // 通知group， 这个线程要执行了，所以可以添加进group中，并且这个线程组中没有没有启动的数量将减少。

        boolean started = false;
        try {
            start0();  // native方法
            started = true;
        } finally {
            try {
                if (!started) {// 通知gorop启动失败
                    group.threadStartFailed(this);
                }
            } catch (Throwable ignore) {
                /* do nothing. If start0 threw a Throwable then
                  it will be passed up the call stack */
            }
        }
    }

    private native void start0();
```




## 参考
https://segmentfault.com/a/1190000021433836#item-7

https://wangchangchung.github.io/2016/12/05/Java%E5%B8%B8%E7%94%A8%E7%B1%BB%E6%BA%90%E7%A0%81%E2%80%94%E2%80%94Thread%E6%BA%90%E7%A0%81%E8%A7%A3%E6%9E%90/

http://ifeve.com/java-concurrency-constructs/

https://blog.csdn.net/chenkaibsw/article/details/80912878?depth_1-utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3&utm_source=distribute.pc_relevant.none-task-blog-BlogCommendFromBaidu-3