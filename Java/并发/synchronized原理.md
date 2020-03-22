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

ObjectMonitor中有两个队列，_WaitSet 和 _EntryList，用来保存ObjectWaiter对象列表( 每个等待锁的线程都会被封装成ObjectWaiter对象)，(思考:_EntryList和_WaitSet设计的原因，可能原因是synchronized是可重人锁，如_EntryList会出现重复的线程，带是拿锁执行只有改线程拿到就可以执行)







