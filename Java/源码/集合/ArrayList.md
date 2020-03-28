# ArrayList

## ArrayList的属性

```java
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
	// 根据是否实现RandomAccess接口，判断使用索引还是iterator	
        if (list instanceof RandomAccess || list.size()<BINARYSEARCH_THRESHOLD)
            return Collections.indexedBinarySearch(list, key, c);
        else
            return Collections.iteratorBinarySearch(list, key, c);
    }
```

## ArrayList的几个方法

### add
```java
    public boolean add(E e) {
        // 确保list的容量够添加元素
        ensureCapacityInternal(size + 1);  // Increments modCount!!
        elementData[size++] = e;
        return true;
    }
```
add函数先通过ensureCapacityInternal来确保数组的容量足够，
```java
    private void ensureCapacityInternal(int minCapacity) {
        // 增加修改次数 并扩大数组容量(可能，只有当最新的容量大于当前数组的容量时进行扩容)
        ensureExplicitCapacity(calculateCapacity(elementData, minCapacity));
    }

    // elementData为空则返回最新需要的容量和默认容量的较大值
    private static int calculateCapacity(Object[] elementData, int minCapacity) {
        if (elementData == DEFAULTCAPACITY_EMPTY_ELEMENTDATA) {
            return Math.max(DEFAULT_CAPACITY, minCapacity);
        }
        return minCapacity;
    }

    private void ensureExplicitCapacity(int minCapacity) {
        modCount++;
        // overflow-conscious code
        if (minCapacity - elementData.length > 0)
            grow(minCapacity);
    }

    private void grow(int minCapacity) {
        // overflow-conscious code
        int oldCapacity = elementData.length;
        int newCapacity = oldCapacity + (oldCapacity >> 1); // 1.5倍
        if (newCapacity - minCapacity < 0)
            newCapacity = minCapacity;
        // MAX_ARRAY_SIZE = Integer.MAX_VALUE - 8;
        if (newCapacity - MAX_ARRAY_SIZE > 0)
            newCapacity = hugeCapacity(minCapacity);
        // minCapacity is usually close to size, so this is a win:
        // 创建新数组并进行元素复制
        elementData = Arrays.copyOf(elementData, newCapacity);
    }
```


## ArrayList迭代器

Itr是ArrayList的一个内部类，cursor下一个元素下标， lastRet上一次返回值下标， expectedModCount用户判断在迭代过程中list是否改变，若改变抛异常。
ListItr也是ArrayList的一个内部类，增加了设置cursor的构造器和previous() 向前移动cursor的方法。


```java
public class ArrayList<E> extends AbstractList<E>
        implements List<E>, RandomAccess, Cloneable, java.io.Serializable
{
	private class Itr implements Iterator<E> {
        	int cursor;       // index of next element to return
        	int lastRet = -1; // index of last element returned; -1 if no such
        	int expectedModCount = modCount;
	...
	}

 	private class ListItr extends Itr implements ListIterator<E> {
        	ListItr(int index) {
            		super();
           		cursor = index;
        	}

		public E previous() {
            		checkForComodification();
            		int i = cursor - 1;
            		if (i < 0)
                		throw new NoSuchElementException();
            		Object[] elementData = ArrayList.this.elementData;
            		if (i >= elementData.length)
                		throw new ConcurrentModificationException();
            		cursor = i;
            		return (E) elementData[lastRet = i];
        	}

        	public void add(E e) {
            		checkForComodification();

            		try {
                		int i = cursor;
                		ArrayList.this.add(i, e);
                		cursor = i + 1;
                		lastRet = -1;
                		expectedModCount = modCount;
            		} catch (IndexOutOfBoundsException ex) {
                		throw new ConcurrentModificationException();
            		}
        	}

	...
	｝

...
}

```

## SubList

实际上还是对于原list在进行操作。部分代码如下：

```java
 private class SubList extends AbstractList<E> implements RandomAccess {
        private final AbstractList<E> parent;
        private final int parentOffset;
        private final int offset;
        int size;


        public void add(int index, E e) {
            rangeCheckForAdd(index);
            checkForComodification();
            parent.add(parentOffset + index, e);
            this.modCount = parent.modCount;
            this.size++;
        }

...
}


```


## 总结
ArrayList底层使用数组实现，故可以随机访问，实现了








