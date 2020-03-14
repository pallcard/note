# hashCode和equal

## haseCode

## hashCode的作用

对于一个HashSet要保证元素不重复，当对有新元素要add时，若只用equal方法进比较，则要遍历所有的元素，当整个Set集合很大时，这样做会带来很大的开销，故引入了hashCode，当需要添加一个元素时，先比较hashCode值，若不存在则直接add，若存在则通过equal方法进行比较。这样添加一个元素最多也只用调用一次equal方法，也不用遍历整个Set。

## hashCode和equal

在 Java 中 hashCode 的实现总是伴随着 equals，他们是紧密配合的，你要是自己设计了其中一个，就要设计另外一个。

equals，我们必须遵循如下规则：

1. 对称性：如果 x.equals(y) 返回是“true”，那么 y.equals(x) 也应该返回是“true”。

2. 反射性：x.equals(x) 必须返回是“true”。

3. 类推性：如果 x.equals(y) 返回是“true”，而且 y.equals(z) 返回是“true”，那么 z.equals(x) 也应该返回是“true”。

4. 一致性：如果 x.equals(y) 返回是“true”，只要x和y内容一直不变，不管你重复 x.equals(y) 多少次，返回都是“true”。

5. 任何情况下，x.equals(null)，永远返回是“false”；x.equals(和x不同类型的对象)永远返回是“false”。

对于 hashCode，我们应该遵循如下规则：

1. 在一个应用程序执行期间，如果一个对象的 equals 方法做比较所用到的信息没有被修改的话，则对该对象调用 hashCode 方法多次，它必须始终如一地返回同一个整数。

2. 如果两个对象根据 equals(Object o) 方法是相等的，则调用这两个对象中任一对象的 hashCode 方法必须产生相同的整数结果。

3. 如果两个对象根据 equals(Object o) 方法是不相等的，则调用这两个对象中任一个对象的 hashCode 方法，不要求产生不同的整数结果。但如果能不同，则可能提高散列表的性能。

## 关系

* 如果 x.equals(y) 返回“true”，那么 x 和 y 的 hashCode() 必须相等。

* 如果 x.equals(y) 返回“false”，那么 x 和 y 的 hashCode() 有可能相等，也有可能不等。


