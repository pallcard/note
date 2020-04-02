# ThreadLocal

ThreadLocal的作用是提供线程内的局部变量，就是在各线程内部创建一个变量的副本，相比于使用各种锁机制访问变量，ThreadLocal的思想就是用空间换时间。

## 概览
1.get:获取与当前线程关联的ThreadLocal值，若无需要实现initialValue，获取初始值（默认为null）
2.set:设置当前线程ThreadLocal值
3.remove：删除ThreadLocal绑定值

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/02/1585795156229-1585795156291.png)

说明：Thread 中有`ThreadLocal.ThreadLocalMap threadLocals = null;`,ThreadLocal中定义了静态内部类ThreadLocalMap，ThreadLocalMap中有一个hash表（table数组，使用开发地址法），其中每个值为Entry,Entry为kv的形式，K为一个ThreadLocal的虚引用(下一次GC回收)，


## 源码

### 数据定义

```java
// ThreadLocal.java
public class ThreadLocal<T> {
    private final int threadLocalHashCode = nextHashCode(); // key的hashcode,通过AtomicInteger实现，每次给通过给nextHashCode增加一个固定值
    private static AtomicInteger nextHashCode =
        new AtomicInteger(); // 此值为static变量，类共享，保证所有实例共享
    static class ThreadLocalMap {
        // WeakReference下一次GC回收
	static class Entry extends WeakReference<ThreadLocal<?>> { 
            /** The value associated with this ThreadLocal. */
            Object value;

            Entry(ThreadLocal<?> k, Object v) {
                 super(k);
                 value = v;
            }
        }
	// 向后移动
        private static int nextIndex(int i, int len) {
            return ((i + 1 < len) ? i + 1 : 0);
        }
	// 向前移动
        private static int prevIndex(int i, int len) {
            return ((i - 1 >= 0) ? i - 1 : len - 1);
        }

	private Entry[] table;
	private int size = 0;
	private int threshold;
	private static final int INITIAL_CAPACITY = 16;
	...

    }

}

// Thread.java
public class Thread implements Runnable {
	ThreadLocal.ThreadLocalMap threadLocals = null;
...
}
```
说明：
1. 初始容量 16
2. 主要数据结构是Entry数组
3. size记录实际个数
4. threshold扩容阈值，初始 len*2/3
5. nextIndex和prevIndex则是为了安全的移动索引
6. key hash的实现是通过一个类变量AtomicInteger，每个key都在这个变量上增加一个值（cas）实现

### 重要方法

#### get

```java
    public T get() {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);  //通过getMap获取Thread内的ThreadLocalMap
        if (map != null) {
            ThreadLocalMap.Entry e = map.getEntry(this);
            if (e != null) {
                @SuppressWarnings("unchecked")
                T result = (T)e.value;
                return result;
            }
        }
        return setInitialValue(); // 延迟初始化
    }

    private T setInitialValue() {
        T value = initialValue(); // 默认返回null
        Thread t = Thread.currentThread();
        // 通过当前Threadlocal获取entry
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value); // new ThreadLocalMap并set第一个value
        return value; 
    }


        private Entry getEntry(ThreadLocal<?> key) {
            int i = key.threadLocalHashCode & (table.length - 1); // 通过hash值找到下标
            Entry e = table[i];
            if (e != null && e.get() == key)
                return e;
            else
                return getEntryAfterMiss(key, i, e); //开地址法查找
        }

	private Entry getEntryAfterMiss(ThreadLocal<?> key, int i, Entry e) {
            Entry[] tab = table;
            int len = tab.length;

            while (e != null) {
                ThreadLocal<?> k = e.get();
                if (k == key)
                    return e;
                if (k == null)
                    expungeStaleEntry(i); // 证明这个Entry中key已经为null,那么这个Entry就是一个过期对象，这里调用expungeStaleEntry清理该Entry，详见remove
                else
                    i = nextIndex(i, len); // ThreadLocal采用的是开放地址法，即有冲突后，把要插入的元素放在要插入的位置后面为null的地方，
                e = tab[i];
            }
            return null;
        }

```

#### set
```java
    public void set(T value) {
        Thread t = Thread.currentThread();
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
    }


```

#### remove
```java
     public void remove() {
         ThreadLocalMap m = getMap(Thread.currentThread());
         if (m != null)
             m.remove(this);
     }

     private void remove(ThreadLocal<?> key) {
          Entry[] tab = table;
          int len = tab.length;
          int i = key.threadLocalHashCode & (len-1);
          for (Entry e = tab[i];
              e != null;
              e = tab[i = nextIndex(i, len)]) {
              if (e.get() == key) {
                  e.clear();
                  expungeStaleEntry(i);
                  return;
              }
          }
     }

    private int expungeStaleEntry(int staleSlot) {
            Entry[] tab = table;
            int len = tab.length;

            // expunge entry at staleSlot 置null
            tab[staleSlot].value = null;
            tab[staleSlot] = null;
            size--;

            // Rehash until we encounter null 由于使用开放地址法，故i值之后的元素可以和之前元素的hash值相同，这些元素需要清理
            Entry e;
            int i;
            for (i = nextIndex(staleSlot, len);
                 (e = tab[i]) != null;  // 需要处理 staleSlot -> 下一个tab值不为null的元素
                 i = nextIndex(i, len)) {
                ThreadLocal<?> k = e.get();
                if (k == null) { // 清理一下value、entry值
                    e.value = null;
                    tab[i] = null;
                    size--;
                } else {
                    int h = k.threadLocalHashCode & (len - 1);
                    if (h != i) {   // rehash 将i放置到正确的位置（h）
                        tab[i] = null;

                        // Unlike Knuth 6.4 Algorithm R, we must scan until
                        // null because multiple entries could have been stale.
                        while (tab[h] != null)  // h位置已占用，往后找
                            h = nextIndex(h, len);
                        tab[h] = e;
                    }
                }
            }
            return i;
        }
```

## 测试代码

```
public class ThreadLocalTest {
  private static final ThreadLocal<Integer> threadLocal = new ThreadLocal<Integer>(){
    @Override
    protected Integer initialValue() {
      return Integer.valueOf(1);
    }
  };

  public static void main(String[] args) {
    Thread[] threads = new Thread[2];
    for (int i=0;i<2;i++) {
      threads[i] = new Thread(() -> {
        System.out.println(Thread.currentThread() +"'s initial value: " + threadLocal.get());
        for (int j=0;j<10;j++) {
          threadLocal.set(threadLocal.get() + j);
        }
        System.out.println(Thread.currentThread() +"'s last value: " + threadLocal.get());
      });
    }

    for (Thread t: threads)
      t.start();
  }
}
```




## 总结

1. 在线程里面放一个Map表, Map定义在ThreadLocal中，Map表中每个元素是Entry，key为ThreadLocal，value为存储对象。

2. 采用延迟初始化，当get/set时，会为当前线程创建map对象并赋值。

3. hash表采用的开放地址法，hash值是通过AtomicInteger（静态变量）存储，每次key都是通过cas操作对类中的这个值进行增加。


## 参考
https://juejin.im/post/5a5efb1b518825732b19dca4