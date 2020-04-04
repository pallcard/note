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

#### 上锁相关代码

```java

// ReentrantLock
    public void lock() {
        sync.lock();
    }

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

    // ReentrantLock#FairSync
    protected final boolean tryAcquire(int acquires) {
        final Thread current = Thread.currentThread();
        int c = getState();
        if (c == 0) {
	    // 若队列中无有效结点才进行cas操作拿锁，非公平锁无需判断hasQueuedPredecessors
            if (!hasQueuedPredecessors() &&
                compareAndSetState(0, acquires)) {
                setExclusiveOwnerThread(current);
                return true;
            }
        }
        else if (current == getExclusiveOwnerThread()) {
            int nextc = c + acquires;
            if (nextc < 0)
                throw new Error("Maximum lock count exceeded");
            setState(nextc);
            return true;
        }
        return false;
    }

// 是公平锁加锁时判断等待队列中是否存在有效节点的方法。如果返回False，说明当前线程可以争取共享资源；如果返回True，说明队列中存在有效节点，当前线程必须加入到等待队列中。
    public final boolean hasQueuedPredecessors() {
        Node t = tail; 
        Node h = head;
        Node s;
        // 双向链表中，第一个节点为虚节点，其实并不存储任何信息，只是占位。真正的第一个有数据的节点，是在第二个节点开始的。
	// 当h != t时： 如果(s = h.next) == null，等待队列正在有线程进行初始化，但只是进行到了Tail指向Head，没有将Head指向Tail，此时队列中有元素，需要返回True。 
	// 如果(s = h.next) != null，说明此时队列中至少有一个有效节点。如果此时s.thread == Thread.currentThread()，说明等待队列的第一个有效节点中的线程与当前线程相同，那么当前线程是可以获取资源的；
	// 如果s.thread != Thread.currentThread()，说明等待队列的第一个有效节点线程与当前线程不同，当前线程必须加入进等待队列。
        return h != t &&
            ((s = h.next) == null || s.thread != Thread.currentThread());

    }

```
#### 解锁相关代码

```
    // ReentrantLock
    public void unlock() {
        sync.release(1);
    }

    // AbstractQueuedSynchronizer
    public final boolean release(int arg) {
        // 释放锁成功后则执行后面的唤醒后续节点的逻辑了
        if (tryRelease(arg)) { 
            Node h = head;
            // addWaiter 方法默认的节点状态为 0，此时节点还没有进入就绪状态
            if (h != null && h.waitStatus != 0) 
                unparkSuccessor(h);
            return true;
        }
        return false;
    }

    // ReentrantLock#Sync
    protected final boolean tryRelease(int releases) {
            int c = getState() - releases;
            if (Thread.currentThread() != getExclusiveOwnerThread())
                throw new IllegalMonitorStateException();
            boolean free = false;
            if (c == 0) {
                free = true;
                setExclusiveOwnerThread(null);
            }
            setState(c);
            return free;
        }

    private void unparkSuccessor(Node node) {
        /*
         * If status is negative (i.e., possibly needing signal) try
         * to clear in anticipation of signalling.  It is OK if this
         * fails or if status is changed by waiting thread.
         */
        int ws = node.waitStatus;
        // 将头节点的状态设置为0, 尝试清除头节点的状态，改为初始状态
        if (ws < 0)
            compareAndSetWaitStatus(node, ws, 0);

        /*
         * Thread to unpark is held in successor, which is normally
         * just the next node.  But if cancelled or apparently null,
         * traverse backwards from tail to find the actual
         * non-cancelled successor.
         */
        // 后继节点
        Node s = node.next; 
        // 如果后继节点为null，或者已经被取消了
        if (s == null || s.waitStatus > 0) {
            s = null;
            // for循环从队列尾部一直往前找可以唤醒的节点
            for (Node t = tail; t != null && t != node; t = t.prev)
                if (t.waitStatus <= 0)
                    s = t;
        }
        if (s != null)
	    // 唤醒后继节点
            LockSupport.unpark(s.thread); 
    }
```

#### 关于FairSync和NonfairSync中tryAcquire
```
// FairSync
        protected final boolean tryAcquire(int acquires) {
            return nonfairTryAcquire(acquires);
        }
	// 
        final boolean nonfairTryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {
                int nextc = c + acquires;
                if (nextc < 0) // overflow
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }

    // FairSync
    protected final boolean tryAcquire(int acquires) {
            final Thread current = Thread.currentThread();
            int c = getState();
            if (c == 0) {
                if (!hasQueuedPredecessors() && // 若队列中无有效结点才进行cas操作拿锁
                    compareAndSetState(0, acquires)) {
                    setExclusiveOwnerThread(current);
                    return true;
                }
            }
            else if (current == getExclusiveOwnerThread()) {  // 可重入锁
                int nextc = c + acquires;
                if (nextc < 0)
                    throw new Error("Maximum lock count exceeded");
                setState(nextc);
                return true;
            }
            return false;
        }
```



### 例子说明

开头例子启动了5个线程(t1,...t5)，启动后他们会去争抢锁，假设t1最先lock，他通过cas操作拿到了锁（假设长时间占用），然后t2(可能是其他线程)在取lock时会失败，然后调用acquire，会再次去获取锁，又失败了，就会将t2封装成一个Node对象，将其加入到队列中，然后在调用acquireQueued，此时t2为第一个结点，会再次去获取锁，若失败，修改前一个结点（head）的waitstatus=-1，自旋再次调用一遍，此时前一个结点的waitstaus=-1，故阻塞改线程。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585997115494-1585997115496.png)

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585997180105-1585997180109.png)

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585997141559-1585997141562.png)


参考：

https://mp.weixin.qq.com/s/jCBrHSVK647bdVIPvJHxOg

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html


