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




}

```


https://blog.csdn.net/carson_ho/article/details/79373134