# LinkedList

## LinkedList的属性
```java
public class LinkedList<E>
    extends AbstractSequentialList<E>
    implements List<E>, Deque<E>, Cloneable, java.io.Serializable {
    transient int size = 0;
    transient Node<E> first;
    transient Node<E> last;

...
    // 双向链表结点
    private static class Node<E> {
        E item;
        Node<E> next;
        Node<E> prev;

        Node(Node<E> prev, E element, Node<E> next) {
            this.item = element;
            this.next = next;
            this.prev = prev;
        }
    }

}
```




## 源码
[ArrayList](https://github.com/pallcard/learn-java/blob/master/src/main/resources/jdk/jdk1_8/java/util/LinkedList.java "LinkedList")

