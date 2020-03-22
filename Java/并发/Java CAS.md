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

```