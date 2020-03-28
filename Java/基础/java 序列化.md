# java 序列化

## 序列化

序列化（Serialization）是将对象的状态信息转化为可以存储或者传输的形式的过程，**一般将一个对象存储到一个储存媒介**，例如档案或记忆体缓冲等，在网络传输过程中，可以是字节或者XML等格式；而字节或者XML格式的可以还原成完全相等的对象，这个相反的过程又称为反序列化。

## java序列化

对象序列化机制（object serialization）是java语言内建的一种对象持久化方式，通过对象序列化，可以将对象的状态信息保存于字节数组，并且可以在有需要的时候将这个字节数组通过反序列化的方式转换成对象。

## 接口

1. java.io.Serializable
标识接口，Java类通过实现java.io.Serialization接口来启用序列化功能，未实现此接口的类将无法将其任何状态或者信息进行序列化或者反序列化。

2. java.io.Externalizable

Externalizable继承了Serializable，该接口中定义了两个抽象方法：writeExternal()与readExternal()。当使用Externalizable接口来进行序列化与反序列化的时候需要开发人员重写writeExternal()与readExternal()方法。在使用Externalizable进行序列化的时候，在读取对象时，会调用被序列化类的无参构造器去创建一个新的对象，然后再将被保存对象的字段的值分别填充到新对象中。所以，实现Externalizable接口的类必须要提供一个public的无参的构造器。

## serialVersionUID

Java的序列化机制是通过判断类的serialVersionUID来验证版本一致性的。在进行反序列化时，JVM会把传来的字节流中的serialVersionUID与本地相应实体类的serialVersionUID进行比较，如果相同就认为是一致的，可以进行反序列化，否则就会出现序列化版本不一致的异常，即是InvalidCastException。

## transient 关键字
transient 关键字的作用是控制变量的序列化，在变量声明前加上该关键字，可以阻止该变量被序列化到文件中，在被反序列化后，transient 变量的值被设为初始值，如 int 型的是 0，对象型的是 null。

## 静态变量
序列化时，并不保存静态变量，这其实比较容易理解，序列化保存的是对象的状态，静态变量属于类的状态，因此 序列化并不保存静态变量。

## 代码
[代码](https://github.com/pallcard/learn-java/blob/master/src/main/resources/jdk/jdk1_8/java/util/ArrayList.java "ArrayList")
