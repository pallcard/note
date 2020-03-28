# Vector

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

    protected Object[] elementData;
    // 集合当前大小
    protected int elementCount;
    // 扩容时，增长的大小，若此值<=0，按2倍增长
    protected int capacityIncrement;

...
}

```

## add
```
    public synchronized boolean add(E e) {
        modCount++;
        ensureCapacityHelper(elementCount + 1);
        elementData[elementCount++] = e;
        return true;
    }
```



## 源码
![ArrayList](https://github.com/pallcard/learn-java/blob/master/src/main/resources/jdk/jdk1_8/java/util/ArrayList.java "ArrayList")