# Vector

```java
public class Vector<E>
    extends AbstractList<E>
    implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{

    protected Object[] elementData;
    protected int elementCount;
    protected int capacityIncrement;

...
}

```

## 源码
![ArrayList](https://github.com/pallcard/learn-java/blob/master/src/main/resources/jdk/jdk1_8/java/util/ArrayList.java "ArrayList")