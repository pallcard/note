# Java CAS

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

```java
public final int getAndIncrement() {
    return unsafe.getAndAddInt(this, valueOffset, 1);
}
```

```java
public final int getAndAddInt(Object var1, long var2, int var4) {
    int var5;
    do {
        var5 = this.getIntVolatile(var1, var2);
    } while(!this.compareAndSwapInt(var1, var2, var5, var5 + var4));

    return var5;
}
```

```java


```



