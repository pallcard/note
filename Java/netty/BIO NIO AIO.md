# BIO NIO AIO
## BIO
网络编程基本模型是Client/Server模型，两个进程之间进行相互通信，其中服务端提供位置信息（IP，端口），客户端通过连接操作向服务器监听的地址发起连接请求，通过三次握手建立连接，然后通过网络套接字进行通信。

### BIO 

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE2020316234123-1584373317560.png)

采用BIO模型的服务端通常都是一个独立的Acceptor线程负责监听客户端连接，他接受到客户端连接请求之后为每个客户端创建一个新的线程进行处理。











