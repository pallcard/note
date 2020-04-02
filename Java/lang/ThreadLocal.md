# ThreadLocal

ThreadLocal的作用是提供线程内的局部变量，就是在各线程内部创建一个变量的副本，相比于使用各种锁机制访问变量，ThreadLocal的思想就是用空间换时间。

## 概览
1.get:获取与当前线程关联的ThreadLocal值，若无需要实现initialValue，获取初始值（默认为null）
2.set:设置当前线程ThreadLocal值
3.remove：删除ThreadLocal绑定值

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/02/1585795156229-1585795156291.png)

说明：Thread 中有`ThreadLocal.ThreadLocalMap threadLocals = null;`,ThreadLocal中定义了静态内部类ThreadLocalMap，ThreadLocalMap中有一个hash表（table数组，使用开发地址法），其中每个值为Entry,Entry为kv的形式，K为一个ThreadLocal的虚引用(下一次GC回收)，



## 源码

```
// ThreadLocal.java
public class ThreadLocal<T> {
	static class ThreadLocalMap {
		// 下一次GC回收
		static class Entry extends WeakReference<ThreadLocal<?>> { 
            		/** The value associated with this ThreadLocal. */
            		Object value;

            		Entry(ThreadLocal<?> k, Object v) {
                		super(k);
                		value = v;
            		}
        	}

	}

	private Entry[] table;
	private int size = 0;
	private int threshold;

	...
}

// Thread.java
public class Thread implements Runnable {
	ThreadLocal.ThreadLocalMap threadLocals = null;
...
}
```



参考
https://juejin.im/post/5a5efb1b518825732b19dca4