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

## 示例
```
import java.util.concurrent.locks.LockSupport;

public class LockSupportTest {

  public static void main(String[] args) {
//    可以看到，在主线程调用park阻塞后，在myThread线程中发出了中断信号，
//   此时主线程会继续运行，也就是说明此时interrupt起到的作用与unpark一样。
//    MyThread myThread = new MyThread(Thread.currentThread());
    MyThread2 myThread = new MyThread2(Thread.currentThread());
    myThread.start();
    System.out.println("before park");
    // 获取许可
    LockSupport.park("ParkAndUnparkDemo");
    System.out.println("after park");
  }
}


class MyThread extends Thread {
  private Object object;

  public MyThread(Object object) {
    this.object = object;
  }

  public void run() {
    System.out.println("before interrupt");
    try {
      // 休眠3s
      Thread.sleep(3000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    Thread thread = (Thread) object;
    // 中断线程
    thread.interrupt();
    System.out.println("after interrupt");
  }
}


class MyThread2 extends Thread {
  private Object object;

  public MyThread2(Object object) {
    this.object = object;
  }

  public void run() {
    System.out.println("before unpark");
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    // 获取blocker
    System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));
    // 释放许可
    LockSupport.unpark((Thread) object);
    // 休眠500ms，保证先执行park中的setBlocker(t, null);
    try {
      Thread.sleep(500);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    // 再次获取blocker
    System.out.println("Blocker info " + LockSupport.getBlocker((Thread) object));

    System.out.println("after unpark");
  }
}
```
1. 
2. 


## 参考

https://www.cnblogs.com/leesf456/p/5347293.html
