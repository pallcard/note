# TCP与UDP

## TCP
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
3. TIME_WAIT：表示收到FIN，处于等待
