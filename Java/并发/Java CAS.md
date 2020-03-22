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

Unsafe中getAndAddInt()如下，首先通过getIntVolatile获取原来的值，然后调用cas（compareAndSwapInt），判断当前值和

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



