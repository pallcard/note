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

### 锁优化
锁的状态总共有四种，无锁状态、偏向锁、轻量级锁和重量级锁。随着锁的竞争，锁可以从偏向锁升级到轻量级锁，再升级的重量级锁，但是锁的升级是单向的。

同一线程申请相同的锁   =>  偏向锁
线程交替执行同步块     =>  轻量级锁
线程持有锁的时间不会太长，直接挂起消耗性能大   => 自旋锁
若干次自旋之后        => 重量级锁

1. 偏向锁
    在大多数情况下，锁不仅不存在多线程竞争，而总是一个线程多次获取。偏向锁的核心思想是，如果一个线程获得了锁，那么锁就进入偏向模式，此时Mark Word 的结构也变为偏向锁结构，当这个线程再次请求锁时，无需再做任何同步操作，即获取锁的过程，这样就省去了大量有关锁申请的操作。对于锁竞争比较激烈的场合，偏向锁就失效了。
2. 轻量级锁
    对绝大部分的锁，在整个同步周期内都不存在竞争，轻量级锁所适应的场景是线程交替执行同步块的场合，如果存在同一时间访问同一锁的场合，就会导致轻量级锁膨胀为重量级锁。
3. 自旋锁
    轻量级锁失败后，虚拟机为了**避免线程真实地在操作系统层面挂起**，还会进行一项称为自旋锁的优化手段。
4. 锁消除
    Java虚拟机在JIT编译时(可以简单理解为当某段代码即将第一次被执行时进行编译，又称即时编译)，通过对运行上下文的扫描，去除不可能存在共享资源竞争的锁，通过这种方式消除没有必要的锁，可以节省毫无意义的请求锁时间。
```java
public class StringBufferRemoveSync {

    public void add(String str1, String str2) {
        //StringBuffer是线程安全,由于sb只会在append方法中使用,不可能被其他线程引用
        //因此sb属于不可能共享的资源,JVM会自动消除内部的锁
        StringBuffer sb = new StringBuffer();
        sb.append(str1).append(str2);
    }

    public static void main(String[] args) {
        StringBufferRemoveSync rmsync = new StringBufferRemoveSync();
        for (int i = 0; i < 10000000; i++) {
            rmsync.add("abc", "123");
        }
    }

}
```

### 线程中断
当一个线程处于**被阻塞状态**或者试图执行一个阻塞操作时，使用`Thread.interrupt()`方式中断该线程，注意此时将会抛出一个InterruptedException的异常(该异常必须捕捉无法向外抛出)，同时中断状态将会被复位(由中断状态改为非中断状态)

处于**运行期且非阻塞**的状态的线程，这种情况下，直接调用`Thread.interrupt()`中断线程是不会得到任响应的，需要我们手动（`Thread.isInterrupted()`）进行中断检测并结束程序

### notify/notifyAll和wait

[WaitNotifyDemo](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/concurrent/WaitNotifyDemo.java "WaitNotifyDemo")

* wait() 
    总是在一个循环中被调用，挂起当前线程来等待一个条件的成立。 Wait调用会一直等到其他线程调用notifyAll()时才返回。当一个线程在执行synchronized 的方法内部，调用了wait()后， 该线程会释放该对象的锁， 然后该线程会被添加到该对象的等待队列中（waiting queue）, 只要该线程在等待队列中， 就会一直处于闲置状态， 不会被调度执行。 要注意wait()方法会强迫线程先进行释放锁操作，所以在调用wait()时， 该线程必须已经获得锁，否则会抛出异常。

* notify/notifyAll
    当一个线程调用一个对象的notify()方法时， 调度器会从所有处于该对象等待队列（_WaitSet）的线程中取出任意一个线程， 将其添加到入口队列(_EntryList) 中. 然后在入口队列中的多个线程就会竞争对象的锁， 得到锁的线程就可以继续执行。 如果等待队列中（_WaitSet）没有线程， notify()方法不会产生任何作用。notifyAll() 和notify()工作机制一样， 区别在于notifyAll()会将等待队列（_WaitSet）中所有的线程都添加到入口队列中（_EntryList）

### await和signal/signalAll

[AwaitSignalDemo](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/concurrent/AwaitSignalDemo.java "AwaitSignalDemo")

java.util.concurrent 类库中提供了 Condition 类来实现线程之间的协调，可以在 Condition 上调用 await() 方法使线程等待，其它线程调用 signal() 或 signalAll() 方法唤醒等待的线程。

相比于 wait() 这种等待方式，await() 可以指定等待的条件，因此更加灵活。




## 参考
https://blog.csdn.net/javazejian/article/details/72828483

https://blog.csdn.net/lengxiao1993/article/details/52296220





