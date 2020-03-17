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






