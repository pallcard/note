# AQS
AbstractQueuedSynchronizer（AQS）是同步器实现框架，它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

## 源码

### 定义
```
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {

    private transient volatile Node head;  // 头结点
    private transient volatile Node tail;  // 尾结点
    private volatile int state; //state 字段为同步状态，其中 state > 0 为有锁状态，每次加锁就在原有 state 基础上加 1，即代表当前持有锁的线程加了 state 次锁，反之解锁时每次减一，当 statte = 0 为无锁状态；


}
```

#### Node
```
 static final class Node {
        // 共享模式，表示线程要获取的是共享锁，即一个锁可以被不同的线程拥有
        static final Node SHARED = new Node();  
        // 独占模式，表示线程要获取的独占锁，即一个锁只能被一个线程拥有
        static final Node EXCLUSIVE = null;  

        //表示当前节点的线程因为超时或中断被取消了
        static final int CANCELLED =  1;  
        // 表示当前节点的后续节点中的线程通过 park 被阻塞了，需要通过unpark解除它的阻塞
        static final int SIGNAL    = -1; 
        // 表示当前节点在 condition 队列中
        static final int CONDITION = -2;  
        // 共享模式的头结点可能处于此状态，表示无条件往下传播,引入此状态是为了优化锁竞争，使队列中线程有序地一个一个唤醒
        static final int PROPAGATE = -3;

        volatile int waitStatus;
        volatile Node prev;
        volatile Node next;
        volatile Thread thread;
        Node nextWaiter;

}
```


### 重要方法

#### acquire
```
public final void acquire(int arg) {
        if (!tryAcquire(arg) &&
            acquireQueued(addWaiter(Node.EXCLUSIVE), arg))
            selfInterrupt();
    }

```


#### addWaiter
```
private Node addWaiter(Node mode) {
        Node node = new Node(Thread.currentThread(), mode);
        // Try the fast path of enq; backup to full enq on failure
        Node pred = tail;
        if (pred != null) {  // 尝试快速将该节点加入到队列的尾部
            node.prev = pred;
            if (compareAndSetTail(pred, node)) { // 当tail=pred时，更新tail为node
                pred.next = node;
                return node;
            }
        }
        enq(node);
        return node;
    }

    private Node enq(final Node node) {
        for (;;) {   // 自旋直至成功
            Node t = tail;
            if (t == null) { // Must initialize   等待队列为空
                if (compareAndSetHead(new Node()))
                    tail = head;
            } else {
                node.prev = t;
                if (compareAndSetTail(t, node)) {
                    t.next = node;
                    return t;
                }
            }
        }
    }
```


![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/02/1585839797495-1585839797498.png)


说明：初始CLH队列为空，当调用`addWaiter()`时，`pred=tail`此时为空，直接调用`enq(node)`,在enq方法中，`t=tail`此时为空，故通过cas设置head为new Node();设置成功令`tail=head`,否则自旋；此时`t=tail`不空，设置node的prev指针，cas设置tail，设置t的next指针，然后返回t，addWaiter返回node。
当再次调用`addWaiter()`时，`pred=tail`此时不为空，将node加入尾部，若失败，则使用`enq(node)`添加。




## 参考
https://juejin.im/entry/5ae02a7c6fb9a07ac76e7b70

https://www.jianshu.com/p/c806dd7f60bc

http://objcoding.com/2019/05/05/aqs-exclusive-lock/#aqs-%E7%BB%93%E6%9E%84%E5%89%96%E6%9E%90

https://tech.meituan.com/2019/12/05/aqs-theory-and-apply.html
