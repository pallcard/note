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

    // lock.lockInterruptibly();
    private void doAcquireInterruptibly(int arg)
        throws InterruptedException {
        final Node node = addWaiter(Node.EXCLUSIVE);
        boolean failed = true;
        try {
            for (;;) {
                final Node p = node.predecessor();
                if (p == head && tryAcquire(arg)) {
                    setHead(node);
                    p.next = null; // help GC
                    failed = false;
                    return;
                }
                if (shouldParkAfterFailedAcquire(p, node) &&
                    parkAndCheckInterrupt())
                    throw new InterruptedException();
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    // lock.lock();
    //当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；并且返回当前线程在等待过程中有没有中断过。
    final boolean acquireQueued(final Node node, int arg) {
        boolean failed = true;
        try {
            boolean interrupted = false; //中断标志
            for (;;) {  // 自旋
                final Node p = node.predecessor();  // 获取当前节点的 pred 节点
                if (p == head && tryAcquire(arg)) { //当前线程的前驱节点是头结点，且同步状态成功，head 节点代表当前持有锁的线程，那么如果当前节点的 pred 节点是 head 节点，说明当前节点在真实数据队列的首部，就尝试获取锁（头结点是虚节点）
                    setHead(node); // 获取锁成功，头指针移动到当前node
                    p.next = null; // help GC
                    failed = false;
                    return interrupted; // 不需要挂起，返回 false
                } // 说明p为头节点且当前没有获取到锁（可能是非公平锁被抢占了）或者是p不为头结点，这个时候就要判断当前node是否要被阻塞（被阻塞条件：前驱节点的waitStatus为-1），防止无限循环浪费资源。具体两个方法下面细细分析
                if (shouldParkAfterFailedAcquire(p, node) &&  // 获取锁失败，则进入挂起逻辑
                    parkAndCheckInterrupt()) //挂起当前线程，阻塞调用栈，返回当前线程的中断状态。
                    interrupted = true; //lock.lock();和lock.lockInterruptibly();区别
            }
        } finally {
            if (failed)
                cancelAcquire(node);
        }
    }

    private final boolean parkAndCheckInterrupt() {
        LockSupport.park(this); //LockSupport 是用来创建锁和其他同步类的基本线程阻塞原语。
        return Thread.interrupted();  // 该方法调用后会将中断标示位清除，即重新设置为false，主要用于lockInterruptibly，调用lock无用，但是由于此处会改变中断状态，之后需要还原
    }

```

说明：在main线程中新建了两个线程t1，t2；t1是先去拿锁，然后死循环打印（通过t2去改变状态来使得循环结束）；对于t2也是先拿锁，在解锁前改变flag状态使得t1循环结束。

流程：main启动t1、t2之后，t1首先会拿到锁，执行循环，然后t2通过`LockSupport.park(this);`阻塞，然后main中调用了`t2.interrupt();`，此时t2继续执行，由于中断doAcquireInterruptibly抛出异常，node被设置为取消，然后t2的继续执行。此时t1，t2未进行同步。

```java
import java.util.concurrent.locks.Lock;
import java.util.concurrent.locks.ReentrantLock;
public class ReentrantLockTest3 {

  public static Lock lock = new ReentrantLock();

  public static int count = 0;

  public static void main(String[] args) {

    Thread t1 = new Thread(() -> {

      lock.lock();

      System.out.println(Thread.currentThread().getName()+"start");
      for (int i = 0; i < 100000; i++) {
        System.out.println(Thread.currentThread().getName()+" :"+count++);
      }
      System.out.println(Thread.currentThread().getName()+".....");
      System.out.println(Thread.currentThread().getName()+"end");

      lock.unlock();

    });

    Thread t2 = new Thread(() -> {
      try {
        lock.lockInterruptibly();
	// lock.lock();
      } catch (InterruptedException e) {
        System.out.println(Thread.currentThread().getName()+"interrupt");
        e.printStackTrace();
      }
      System.out.println(Thread.currentThread().getName()+"start");

      for (int i = 0; i < 100000; i++) {
        System.out.println(Thread.currentThread().getName()+" :"+count++);
      }

      System.out.println(Thread.currentThread().getName()+"end");

      lock.unlock();
    });

    t1.start();
    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    t2.start();

    try {
      Thread.sleep(10);
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    t2.interrupt();
    try {
      t1.join();
      t2.join();
    } catch (InterruptedException e) {
      e.printStackTrace();
    }
    System.out.println(Thread.currentThread().getName() +" :"+count);
  }
}
```

说明：t1 和t2线程分别将count加100000，若使用lock.lock()进行加锁，lock不响应中断，故t1，t2会同步执行，最终结果是200000，但是使用lock.lockInterruptibly()进行加锁，中断会会抛出异常，最后导致t1，t2未同步，最终结果不确定。






