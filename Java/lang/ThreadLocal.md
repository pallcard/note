# ThreadLocal

ThreadLocal的作用是提供线程内的局部变量，就是在各线程内部创建一个变量的副本，相比于使用各种锁机制访问变量，ThreadLocal的思想就是用空间换时间。

## 概览
1.get:获取与当前线程关联的ThreadLocal值，若无需要实现initialValue，获取初始值（默认为null）
2.set:设置当前线程ThreadLocal值
3.remove：删除ThreadLocal绑定值


```


```



参考
https://juejin.im/post/5a5efb1b518825732b19dca4