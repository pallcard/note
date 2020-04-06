# ReentrantLock 中断

## 示例代码
```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest2 {

  public static Lock lock = new ReentrantLock();

  public static volatile int test = 0;

  public static void main(String[] args) {

    Thread t1 = new Thread(() -> {

      lock.lock();

      System.out.println(Thread.currentThread().getName()+"start");
      try {
        Thread.sleep(10000);
      } catch (InterruptedException e) {
        e.printStackTrace();
      }
      while (test<=0) {
        System.out.println("111");
      }
      System.out.println(Thread.currentThread().getName()+".....");
      System.out.println(Thread.currentThread().getName()+"end");

      lock.unlock();

    });

    Thread t2 = new Thread(() -> {
      try {
        lock.lockInterruptibly();
//        lock.lock();
      } catch (InterruptedException e) {
        System.out.println(Thread.currentThread().getName()+"interrupt");
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName()+"start");
      try {
//        Thread.sleep(10000);
        System.out.println(Thread.currentThread().getName()+".....");
        System.out.println(Thread.currentThread().getName()+"end");
      } finally {
        test++;
        lock.unlock();
      }
    });

    t1.start();
    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    t2.start();

    try {
      Thread.sleep(1000);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    t2.interrupt();
  }
}
```