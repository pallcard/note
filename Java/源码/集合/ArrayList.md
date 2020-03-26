# ArrayList

```
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
    // 默认大小 10
    private static final int DEFAULT_CAPACITY = 10;
    transient Object[] elementData;
    // 此属性存在于 AbstractList中
    protected transient int modCount = 0;
    ......
}

```

* modCount 作用：
    在使用迭代器遍历的时候，用来检查列表中的元素是否发生结构性变化（列表元素数量发生改变）了，主要在多线程环境下需要使用，防止一个线程正在迭代遍历，另一个线程修改了这个列表的结构。

* RandomAccess 作用
ArrayList用for循环遍历比iterator迭代器遍历快，LinkedList用iterator迭代器遍历比for循环遍历快，这时就需要用instanceof来判断List集合子类是否实现RandomAccess接口。
```java
    @SuppressWarnings("unchecked")
    public static <T> int binarySearch(List<? extends T> list, T key, Comparator<? super T> c) {
        if (c==null)
            return binarySearch((List<? extends Comparable<? super T>>) list, key);
	// 判断根据是否实现RandomAccess接口，来调用不同的方法	
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key, c);
        else
            return Collections.iteratorBinarySearch(list, key, c);
    }
```









