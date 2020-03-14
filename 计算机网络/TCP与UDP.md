# TCP与UDP

## TCP
### 三次握手
![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/14/e92d0ebc-7d46-413b-aec1-34a39602f787-1584172028486.png?token=AHBYBJ5DXSVBS24EB35IC226NSGD4)

* 第一次： 客户端发送SYN给服务器，表示请求建立连接
* 第二次： 服务器发送SYN、ACK给客户端，表示服务器是通的，并同意建立连接
* 第三次： 客户端发送ACK给服务器，表示客户端是通的，可以相互通信了。

3次握手原因：保证