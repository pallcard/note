# TCP与UDP

## TCP
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/1584177724544-1584177724548.png?token=AHBYBJYTVBCNWL2YHKXV7KS6NSRIE)

几个字段：
* 序号：对于字节流进行编号
* 确认好：期望收到的下一个报文段的序号
* 数据偏移：首部的长度
* 确认ACK：当 ACK=1 时确认号字段有效，否则无效
* 同步SYN：建立时用来同步序号
* 终止FIN：用来释放一个连接
* 窗口：窗口值作为接收方让发送方设置其发送窗口的依据
### 三次握手
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/e92d0ebc-7d46-413b-aec1-34a39602f787-1584172028486.png?token=AHBYBJ5DXSVBS24EB35IC226NSGD4)

* 第一次： 客户端发送**SYN**给服务器，表示请求建立连接
* 第二次： 服务器发送**SYN、ACK**给客户端，表示服务器是通的，并同意建立连接
* 第三次： 客户端发送**ACK**给服务器，表示客户端是通的，可以相互通信了。

3次握手原因：防止错误的连接请求（重传机制产生多个连接请求）使得服务器错误的打开连接，浪费资源。

### 四次挥手
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/f87afe72-c2df-4c12-ac03-9b8d581a8af8-1584172500269.jpg?token=AHBYBJY53R3V3SU4WTLRPN26NSHBI)

* 第一次:客户端发送FIN给服务器，表示客户端数据已经传完，要关闭连接
* 第二次：服务器发送ACK给客户端，表示服务器收到了请求，此时服务器需要把未传完的数据传输过去
* 第三次：服务器发送FIN给客户端，表示数据传输完毕，准备关闭
* 第四次：客户端发送ACK给服务器，表示收到，等待2MSL，然后关闭

四次挥手原因：
客户端发送关闭连接的请求后，客户端处于能接收数据的状态，服务上数据还未传输完毕，要保证服务器上所有数据都能被接收到。

time-wait
1. 需要让服务器知道，客户端收到FIN请求，若没收到在这个时间端，服务器会从新发送FIN给客户端
2. 保证服务器最后发送的数据都能被客户端接收到

### 状态说明
三次握手
1. LISTEN：表示socket处理listen状态
2. SYN_SENT：表示socket在建立连接时，已发送SYN请求，在等待确认报文
3. SYN_REVD：表示收到了SYN请求
4. ESTABLISHED：表示建立连接完成
四次挥手
1. FIN_WAIT_1：表示等待对方发送FIN请求
2. FIN_WAIT_2：表示收到了ACK，继续等待对方发送FIN请求
3. TIME_WAIT：表示收到FIN，需要等2MSL(Max Segment Lifetime)，网络不可靠，发送ACK后，服务器为收到ACK，则服务器重传FIN
4. CLOSE_WAIT：表示收到了FIN，并返回了ACK，等待关闭状态，需要等到数据发送完毕。
5. LAST_ACK：服务器发送FIN后等待ACK状态
6. CLOSED：连接已经断开
7. CLOSEING：表示发送完FIN，直接收到对方FIN，两端几乎同时关闭同一个socket的时候才会出现CLOSING状态

### 滑动窗口

发送窗口内的字节都允许被发送，接收窗口内的字节都允许被接收。接收窗口只会对窗口内最后一个**按序到达**的字节进行确认，例如接收窗口已经收到的字节为 {31, 34, 35}，其中 {31} 按序到达，而 {34, 35} 就不是，因此只对字节 31 进行确认。发送方得到一个字节的确认之后，就知道这个字节之前的所有字节都已经被接收。
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/a3253deb-8d21-40a1-aae4-7d178e4aa319-1584177434517.jpg?token=AHBYBJYL5W65J7BNFE7CV326NSQVU)


### 流量控制、拥塞控制
流量控制：通过控制发送方发送速率，使得接收方来得及接收
拥塞控制：（网络出现拥塞，出现分组丢失，触发超时重传，更拥塞），通过控制发送方发送速率，控制整个网络的拥塞程度。（慢开始，拥塞避免，快重传，快恢复）

## UDP
### UDP首部

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/1584177682155-1584177682163.png?token=AHBYBJ4V66QO27ZZFD6CHVK6NSRFQ)

## 比较

* 用户数据报协议 UDP（User Datagram Protocol）是无连接的，尽最大可能交付，没有拥塞控制，面向报文（对于应用程序传下来的报文不合并也不拆分，只是添加 UDP 首部），支持一对一、一对多、多对一和多对多的交互通信。

* 传输控制协议 TCP（Transmission Control Protocol）是面向连接的，提供可靠交付，有流量控制，拥塞控制，提供全双工通信，面向字节流（把应用层传下来的报文看成字节流，把字节流组织成大小不等的数据块），每一条 TCP 连接只能是点对点的（一对一）。


## TCP网络编程中connect()、listen()和accept()三者之间的关系

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/1584178293109-1584178293183.png?token=AHBYBJ4J4S7ZW577YXVNKO26NSSLK)

listen() 函数的主要作用就是将套接字( sockfd )变成被动的连接监听套接字（被动等待客户端的连接），至于参数 backlog 的作用是设置内核中连接队列的长度（这个长度有什么用，后面做详细的解释），TCP 三次握手也不是由这个函数完成，listen()的作用仅仅告诉内核一些信息。

当有一个客户端主动连接（connect()），Linux 内核就自动完成TCP 三次握手，将建立好的链接自动存储到队列中，如此重复。
