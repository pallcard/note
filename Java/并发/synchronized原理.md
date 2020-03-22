# synchronized原理

## 底层原理
Java 虚拟机中同步（synchronized）是基于进入和退出Monitor对象实现的，无论显式（同步代码块）还是隐式都是如此。同步方法是由调用指令读取运行时常量池中方法的ACC_SYNCHRONIZED标志隐式实现的。

### 对象头
在JVM中，对象在内存分布三块区域：对象头、实例数据和对齐填充。
* 实例变量：存放类（包括父类）属性信息，数组长度，4字节对齐。
* 对齐填充：虚拟机要求对象起始地址必须是8字节的整数倍
* 对象头：Mark Word， Class Metadata Address（指向类元信息）

### Mark Word

中Mark Word在默认情况下存储着对象的HashCode、分代年龄、锁标记位等以下是32位JVM的Mark Word默认存储结构。对象头的信息是与对象自身定义的数据没有关系的额外存储成本，因此考虑到JVM的空间效率，Mark Word 被设计成为一个非固定的数据结构。

|锁状态|25bit|4bit|1bit是否是偏向锁|2bit 锁标志位|
|-|-|-|-|-|
|无锁状态|对象HashCode|对象分代年龄|0|01|

### 重量级锁

重量级锁也就是通常说synchronized的对象锁，锁标识位为10，其中指针指向的是monitor对象（也称为管程或监视器锁）的起始地址，在Java虚拟机(HotSpot)中，monitor是由ObjectMonitor实现的。

ObjectMonitor中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表( 每个等待锁的线程都会被封装成ObjectWaiter对象)，(**思考:_EntryList和_WaitSet设计的原因，可能原因是synchronized是可重人锁，如_EntryList会出现重复的线程，带是拿锁执行只有改线程拿到就可以执行**)，_owner指向持有ObjectMonitor对象的线程，当多个线程同时访问一段同步代码时，首先会进入 _EntryList 集合，当线程获取到对象的monitor 后进入 _Owner 区域并把monitor中的owner变量设置为当前线程同时monitor中的计数器count加1，若线程调用 wait() 方法，将释放当前持有的monitor，owner变量恢复为null，count自减1，同时该线程进入 WaitSe t集合中等待被唤醒。若当前线程执行完毕也将释放monitor(锁)并复位变量的值，以便其他线程进入获取monitor(锁)。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/22/1584848963527-1584848963736.png)

### synchronized代码块、方法原理

*  代码块
    同步语句块的实现使用的是monitorenter 和 monitorexit 指令，其中monitorenter指令指向同步代码块的开始位置，monitorexit指令则指明同步代码块的结束位置，当执行monitorenter指令时，当前线程将试图获取 objectref(即对象锁) 所对应的 monitor 的持有权，当 objectref 的 monitor 的进入计数器为 0，那线程可以成功取得 monitor，并将计数器值设置为 1，取锁成功。如果当前线程已经拥有 objectref 的 monitor 的持有权，那它可以重入这个 monitor (关于重入性稍后会分析)，重入时计数器的值也会加 1。倘若其他线程已经拥有 objectref 的 monitor 的所有权，那当前线程将被阻塞，直到正在执行线程执行完毕，即monitorexit指令被执行，执行线程将释放 monitor(锁)并设置计数器值为0 ，其他线程将有机会持有 monitor 

*  方法
    方法级的同步是隐式，即无需通过字节码指令来控制的，它实现在方法调用和返回操作之中。JVM可以从方法常量池中的方法表结构(method_info Structure) 中的 ACC_SYNCHRONIZED 访问标志区分一个方法是否同步方法。当方法调用时，调用指令将会 检查方法的 ACC_SYNCHRONIZED 访问标志是否被设置，如果设置了，执行线程将先持有monitor（虚拟机规范中用的是管程一词）， 然后再执行方法，最后再方法完成(无论是正常完成还是非正常完成)时释放monitor。

* 总结
    1. 同步代码块：先执行monitorenter获取monitor，monitorexit 释放monitor。
    2. 同步方法：先查看ACC_SYNCHRONIZED访问标准，若干设置了，先获取monitor，然后释放monitor。

## 参考
https://blog.csdn.net/javazejian/article/details/72828483

https://blog.csdn.net/lengxiao1993/article/details/52296220





