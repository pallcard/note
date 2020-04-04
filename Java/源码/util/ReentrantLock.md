# ReentrantLock

## ReentrantLock流程

### 测试代码

main方法中通过`new ReentrantLock();`创建了一个可重入锁，然后通过线程池创建了5个线程，在各个线程中通过`rlock.lock();`进行加锁，然后执行逻辑，最后通过`rlock.unlock();`进行解锁。代码如下：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest {

  public static volatile int count = 0;

  public static void main(String[] args) throws InterruptedException {
    // ReentrantLock rlock = new ReentrantLock(true);
    ReentrantLock rlock = new ReentrantLock();
    ExecutorService executorService = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 5; i++) {
      executorService.execute(()->{
        rlock.lock();
        System.out.println(Thread.currentThread().getName()+ " start");
        for (int j = 0; j < 10; j++) {
          System.out.println(Thread.currentThread().getName()+ " " +count++);
        }
        System.out.println(Thread.currentThread().getName()+ " end");
        rlock.unlock();
      });
    }

    Thread.sleep(5000);
    System.out.println(count);

  }
}
```

### 上锁流程

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585988828882-1585988828890.png)

说明：ReentrantLock 调用lock时，若为非公平锁先直接通过cas设置state的状态，若设置失败，在调用acquire，acquire会先调用tryAcquire(AQS子类实现)，线程会尝试通过cas设置状态，若失败，将thread封装成Node加入到队列（CLH队列）中，然后调用acquireQueued，acquireQueued中会自旋，判断前一个结点是否为head，若为则通过tryAcquire获取锁，若失败去修改前一个结点的waitstatus，挂起当前线程。


![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585988196704-1585988196707.png)

说明：ReentrantLock 调用unlock时，然后调用release，而release通过tryRelease来设置state状态，并设置线程为null，最后unpark后面的结点。



### 代码说明

```java
// ReentrantLock#NonfairSync
        final void lock() {
            if (compareAndSetState(0, 1))//若通过CAS设置变量State（同步状态）成功，也就是获取锁成功，则将当前线程设置为独占线程
                setExclusiveOwnerThread(Thread.currentThread());
            else
                acquire(1);// 存在某种排队等候机制，线程继续等待，仍然保留获取锁的可能，获取锁流程仍在继续
        }

// ReentrantLock#FairSync
        final void lock() {
            acquire(1);
        }


    // AbstractQueuedSynchronizer
    public final void acquire(int arg) {
        //去尝试获取锁，获取成功则设置锁状态并返回true，否则返回false。tryAcquire尝试获取锁，子类实现
        if (!tryAcquire(arg) && 
            //如果tryAcquire返回FALSE（获取同步状态失败），则调用该方法将当前线程加入到CLH同步队列尾部。
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg)) 
            //acquireQueued：当前线程会根据公平性原则来进行阻塞等待（自旋）,直到获取锁为止；并且返回当前线程在等待过程中有没有中断过。
            selfInterrupt(); 
    }



```












