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

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/02/1585838251595-1585838251650.png)