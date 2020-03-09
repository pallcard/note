# hashCode和equal

## hashCode的作用

对于一个HashSet要保证元素不重复，当对有新元素要add时，若只用equal方法进比较，则要遍历所有的元素，当整个Set集合很大时，这样做会带来很大的开销，故引入了hashCode，当需要添加一个元素时，先比较hashCode值，若不存在则直接add，若存在则通过equal方法进行比较。这样添加一个元素最多也只用调用一次equal方法，也不用遍历整个Set。

## hashCode和equal

在 Java 中 hashCode 的实现总是伴随着 equals，他们是紧密配合的，你要是自己设计了其中一个，就要设计另外一个。



