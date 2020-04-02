# AQS
AbstractQueuedSynchronizer（AQS）是同步器实现框架，它是构建锁或者其他同步组件的基础框架（如ReentrantLock、ReentrantReadWriteLock、Semaphore等），JUC并发包的作者（Doug Lea）期望它能够成为实现大部分同步需求的基础。

AQS的主要使用方式是继承，子类通过继承同步器并实现它的抽象方法来管理同步状态。

## 源码

### 定义
```
public abstract class AbstractQueuedSynchronizer
    extends AbstractOwnableSynchronizer
    implements java.io.Serializable {
}
```

#### Node
```
 static final class Node {
        // 共享模式，表示线程要获取的是共享锁，即一个锁可以被不同的线程拥有
        static final Node SHARED = new Node();  
        /** Marker to indicate a node is waiting in exclusive mode */
        static final Node EXCLUSIVE = null;  // 独占模式，表示线程要获取的独占锁，即一个锁只能被一个线程拥有

        /** waitStatus value to indicate thread has cancelled */
        static final int CANCELLED =  1;  //表示当前节点的线程因为超时或中断被取消了
        /** waitStatus value to indicate successor's thread needs unparking */
        static final int SIGNAL    = -1;  // 表示当前节点的后续节点中的线程通过 park 被阻塞了，需要通过unpark解除它的阻塞
        /** waitStatus value to indicate thread is waiting on condition */
        static final int CONDITION = -2;  // 表示当前节点在 condition 队列中
        /**
         * waitStatus value to indicate the next acquireShared should
         * unconditionally propagate
         */
        static final int PROPAGATE = -3; // 共享模式的头结点可能处于此状态，表示无条件往下传播,引入此状态是为了优化锁竞争，使队列中线程有序地一个一个唤醒


```


![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/02/1585838251595-1585838251650.png)