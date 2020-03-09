# hashCode和equal
## hashCode的作用
对于一个HashSet要保证元素不重复，当对有新元素要add时，若只用equal方法进比较，则要遍历所有的元素，当整个Set集合很大时，这样做会带来很大的开销，故引入了hashCode，当需要添加一个元素时，先比较hashCode值，若不存在，在实用equal方法，