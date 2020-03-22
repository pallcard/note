# Java CAS

例子如下：JMMDemo类的属性cnt为AtomicInteger类型，成员方法add(),调用了cnt.incrementAndGet()，然后在看incrementAndGet()。
```java
public class JMMDemo {
  private AtomicInteger cnt = new AtomicInteger();

  public void add() {
    cnt.incrementAndGet();
  }

  public int get() {
    return cnt.get();
  }
}
```
AtomicInteger中incrementAndGet调用了unsafe.getAndAddInt方法。
```java
// AtomicInteger.java

    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
```

Unsafe中getAndAddInt()如下，首先通过getIntVolatile获取原来的值（expect），然后调用cas（compareAndSwapInt(obj, offset, expect, update)），判断obj的值和expect值是否相等，相等则更新

```java
// var1 对象， var2 偏移量， 增加的值
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

```java
public class AtomicInteger extends Number implements java.io.Serializable {
    private static final long serialVersionUID = 6214790243416807050L;

    // setup to use Unsafe.compareAndSwapInt for updates
    private static final Unsafe unsafe = Unsafe.getUnsafe();
    private static final long valueOffset;

    static {
        try {
            valueOffset = unsafe.objectFieldOffset
                (AtomicInteger.class.getDeclaredField("value"));
        } catch (Exception ex) { throw new Error(ex); }
    }

    private volatile int value;

    public final int incrementAndGet() {
        return unsafe.getAndAddInt(this, valueOffset, 1) + 1;
    }
    ... ...
}
```


CAS可以保证一次的读-改-写操作是原子操作，在单处理器上该操作容易实现，CPU提供了两种方法来实现多处理器的原子操作：总线加锁或者缓存加锁。
* 总线加锁
    总线加锁就是就是使用处理器提供的一个LOCK#信号，当一个处理器在总线上输出此信号时，其他处理器的请求将被阻塞住,那么该处理器可以独占使用共享内存。
* 缓存加锁



