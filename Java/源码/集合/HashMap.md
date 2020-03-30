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
    // 最小树形化容量阈值：即 当哈希表中的容量 > 该值时，才允许树形化链表 （即 将链表 转换成红黑树）
    // 否则，若桶内元素太多时，则直接扩容，而不是树形化
    // 为了避免进行扩容、树形化选择的冲突，这个值不能小于 4 * TREEIFY_THRESHOLD
    static final int MIN_TREEIFY_CAPACITY = 64;

    // 每个位置相当于桶
    transient Node<K,V>[] table;
    transient int size;
    transient int modCount;
    // 扩容阈值（threshold）：当哈希表的大小 ≥ 扩容阈值时，就会扩容哈希表（即扩充HashMap的容量）
    // a. 扩容 = 对哈希表进行resize操作（即重建内部数据结构），从而哈希表将具有大约两倍的桶数
    // b. 扩容阈值 = 容量 x 加载因子
    int threshold;
    // 装载因子
    final float loadFactor;

}

```

## put
```java
    public V put(K key, V value) {
        return putVal(hash(key), key, value, false, true);
    }
```

```java
    final V putVal(int hash, K key, V value, boolean onlyIfAbsent,
                   boolean evict) {
        Node<K,V>[] tab; Node<K,V> p; int n, i;
        // 若哈希表的数组tab为空，则 通过resize() 创建， 延迟初始化
        if ((tab = table) == null || (n = tab.length) == 0)
            n = (tab = resize()).length;
        // (n - 1) & hash 按需取 哈希码一定数量的低位 作为存储的数组下标位置
        // 无冲突，则直接创建新结点
        if ((p = tab[i = (n - 1) & hash]) == null)
            tab[i] = newNode(hash, key, value, null);
        // 有冲突
        else {
            Node<K,V> e; K k;
            // 若key存在，则覆盖value值
            if (p.hash == hash &&
                ((k = p.key) == key || (key != null && key.equals(k))))
                e = p;
            // 若是红黑树，则直接在树中插入 or 更新键值对
            else if (p instanceof TreeNode)
                e = ((TreeNode<K,V>)p).putTreeVal(this, tab, hash, key, value);
            // 链表
            else {
                for (int binCount = 0; ; ++binCount) {
                    if ((e = p.next) == null) {
                        // 到链表尾部，尾插
                        p.next = newNode(hash, key, value, null);
                        // 链表长度是否>8（8 = 桶的树化阈值），则把链表转换为红黑树
                        if (binCount >= TREEIFY_THRESHOLD - 1) // -1 for 1st
                            treeifyBin(tab, hash);
                        break;
                    }
                    // 存在key，跳出循环
                    if (e.hash == hash &&
                        ((k = e.key) == key || (key != null && key.equals(k))))
                        break;
                    p = e;
                }
            }
            // 若存在，则替换旧值
            if (e != null) { // existing mapping for key
                V oldValue = e.value;
                if (!onlyIfAbsent || oldValue == null)
                    e.value = value;
                afterNodeAccess(e);
                return oldValue;
            }
        }
        ++modCount;
        // 插入成功，实际存在的键值对数量size > 最大容量threshold
        if (++size > threshold)
            resize();
        afterNodeInsertion(evict);
        return null;
    }
```
### 

https://blog.csdn.net/carson_ho/article/details/79373134