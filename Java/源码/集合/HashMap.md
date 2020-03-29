# HashMap
```java
public class HashMap<K,V> extends AbstractMap<K,V>
    implements Map<K,V>, Cloneable, Serializable {
    // 默认大小 16
    static final int DEFAULT_INITIAL_CAPACITY = 1 << 4; // aka 16
    // 最大容量 =  2的30次方
    static final int MAXIMUM_CAPACITY = 1 << 30;
    // 默认装载因子 0.75
    static final float DEFAULT_LOAD_FACTOR = 0.75f;
    // 桶的树化阈值：即 链表转成红黑树的阈值，在存储数据时，当链表长度 > 该值时，则将链表转换成红黑树
    static final int TREEIFY_THRESHOLD = 8;
    // 桶的链表还原阈值：即 红黑树转为链表的阈值，当在扩容（resize（））时（此时HashMap的数据存储位置会重新计算），
    //在重新计算存储位置后，当原有的红黑树内数量 < 6时，则将 红黑树转换成链表
    static final int UNTREEIFY_THRESHOLD = 6;


}

```


https://blog.csdn.net/carson_ho/article/details/79373134