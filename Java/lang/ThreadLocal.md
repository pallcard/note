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

    }

	private Entry[] table;
	private int size = 0;
	private int threshold;
	private static final int INITIAL_CAPACITY = 16;
	...

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
5. 
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
        return setInitialValue();
    }

    private T setInitialValue() {
        T value = initialValue();
        Thread t = Thread.currentThread();
        // 通过当前Threadlocal获取entry
        ThreadLocalMap map = getMap(t);
        if (map != null)
            map.set(this, value);
        else
            createMap(t, value);
        return value;
    }




```


参考
https://juejin.im/post/5a5efb1b518825732b19dca4