# synchronized
synchronized 是 Java 中的关键字，是利用锁的机制来实现同步的。

锁机制有如下两种特性：
* 互斥性：即在同一时间只允许一个线程持有某个对象锁
* 可见性：必须确保在锁被释放之前，对共享变量所做的修改，对于随后获得该锁的另一个线程是可见的

## 对象锁和类锁

### 对象锁

在 Java 中，每个对象都会有一个 monitor 对象，这个对象其实就是 Java 对象的锁，通常会被称为“内置锁”或“对象锁”。类的对象可以有多个，所以每个对象有其独立的对象锁，互不干扰。

* synchronized(this|object) {}

* 修饰非静态方法

### 类锁

在 Java 中，针对每个类也有一个锁，可以称为“类锁”，类锁实际上是通过对象锁实现的，即类的 Class 对象锁。每个类只有一个 Class 对象，所以每个类只有一个类锁。

* synchronized(类.class) {}
* 修饰静态方法

## 总结
1. 多个线程获取同一个对象锁，同步部分(对象的同步非静态方法、通过代码块)同步执行；
2. 多个线程获取不同的对象锁，同一个类的不同对象的对象锁互不干扰；
3. 对象锁和类锁是独立的，湖北干扰，可以理解为类锁是类的class对象锁，和普通对象锁是不同的对象锁。
4. synchronized关键字不能继承。对于父类中的 synchronized 修饰方法，子类在覆盖该方法时，默认情况下不是同步的，必须显示的使用 synchronized 关键字修饰才行。
5. 接口不能使用synchronized
6. 构造方法不能使用synchronized，但是可以使用synchronized代码块

## 代码
[对象锁](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/concurrent/SynchronizedDemo.java "SynchronizedDemo")
[类锁](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/concurrent/SynchronizedDemo2.java "SynchronizedDemo2")
[对象锁和类锁](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/concurrent/SynchronizedDemo3.java "SynchronizedDemo3")

## 参考
https://juejin.im/post/594a24defe88c2006aa01f1c#heading-4






