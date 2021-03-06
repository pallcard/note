# BIO NIO AIO
## BIO
网络编程基本模型是Client/Server模型，两个进程之间进行相互通信，其中服务端提供位置信息（IP，端口），客户端通过连接操作向服务器监听的地址发起连接请求，通过三次握手建立连接，然后通过网络套接字进行通信。

### BIO

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE2020316234123-1584373317560.png)

采用BIO模型的服务端通常都是一个独立的Acceptor线程负责监听客户端连接，他接受到客户端连接请求之后为每个客户端创建一个新的线程进行处理。这样导致**服务端的线程个数和客户端并发访问数呈1：1**，当并发访问过大，系统会出现线程堆栈溢出、创建失败等问题。

[示例代码](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/net/netty/bio "bio")

### BIO优化

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE2020316235452-1584374147782.png)

BIO需要为每一个客户端都创建一个线程去处理，故可以使用线程池进行优化，从而形成M个客户端，线程池最大线程数N，其中M远大于N，通过线程池可以灵活地调配线程资源。

[示例代码](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/net/netty/bio2 "bio2")

### BIO弊端
尽管BIO可以通过线程池进行优化，但是由于数据传输使用的inputStream、outputStream。
* inputStream的read函数在进行读取操作时，会一直阻塞下去，直到（1）有数据可读，（2）可用数据已经读取完毕，（3）发生空指针异常或I/O异常。这意味着当发送方发送消息缓慢或网络传输慢时，读取输入流一方会长时间阻塞。
* outputStream的write函数，在写输入流的时候会被阻塞，直到所有要发送的字节全部写入完毕或发生异常。当消息的接受者处理缓慢时，不能及时从TCP缓冲区读取数据，Tcp 的window size就会不断减小，发送方就不能发送数据，从而一直阻塞。

## NIO

### 缓冲区 Buffer
NIO所有数据都是用缓冲区处理的，缓冲区实质就是一个数组，缓冲区提供了对数据结构化访问以及维护读写位置limit等信息。
![title]
(https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/17/Popo%E6%88%AA%E5%9B%BE2020317233514-1584459592878.png)

* capacity：最大容量
* position：下一个被读或写的下标
* limit：第一个不被读或写的下标

1. 新建8字节缓冲区，此时capcity = 8, position = 0, limit = 8
2. 读入5个字节，   此时capcity = 8, position = 5, limit = 8
3. flip()         此时capcity = 8, position = 0, limit = 5
4. clear()        此时capcity = 8, position = 0, limit = 8

### 通道 Channel
Channel是一个通道，网络数据通过Channel读取和写入，通道与流的不同之处在于通道是全双工的，可读可写。
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/17/1584459841491-1584459841500.png)

### 多路复用器 Selector
多路复用器提供选择已经就绪的任务的能力，简单来讲，Selector会不断地轮询注册在其上的Channel,如果Channel上发生了读或写事件，这个Channel就处于就绪状态，会被Selector轮询出来，通过SelectorKey获取就绪Channel的集合，然后进行I/O操作。

[示例代码](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/net/netty/nio "nio")

## AIO
NIO 2.0的异步套套接字通道是真正的异步非阻塞I/O，对应UNIX网络编程中事件驱动I/O。

[示例代码](https://github.com/pallcard/learn-java/blob/master/src/main/java/com/wishhust/net/netty/aio "aio")

## 概念&比较

### 异步非阻塞I/O
NIO 的Selector是基于epool实现的，只能算非阻塞I/O,

AIO 是真正的异步I/O，在异步I/O操作时传递信号变量，当操作完成之后会回调相关方法。


|column1|BIO|BIO2|NIO|AIO|
|-|-|-|-|-|
|客户端个数：I/O线程|1：1|M:N|M:1|M：0|
|I/O类型（阻塞）|阻塞I/O|阻塞I/O|非阻塞I/O|非阻塞I/O|
|I/O类型（同步）|同步I/O|同步I/O|同步I/O|异步I/O|
|API使用难度|简单|简单|非常复杂|复杂|
|调试难度|简单|简单|复杂|复杂|
|可靠性|非常差|差|高|高|
|吞吐量|低|中|高|高|

## 其他
JDK NIO的epoll bug，他会导致Selector空轮询，最终导致cpu 100%。


