# I/O模型
## UNIX下的五种#### 四级标题I/O模型

linux的内核将所有外部设备看做是一个文件来操作，对一个文件的读写操作会调用提供的系统命令，返回一个fd(file descriptor),描述符是一个指向内核中的一个**结构体**数字。**以套接字为例讲解**。

### 1. 阻塞I/O
用户进程调用recvfrom，其系统调用直到数据包到达且被复制到应用的缓冲区或发送错误才返回，此期间一直等待。进程从调用recvfrom到返回一直被阻塞。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/15/Popo%E6%88%AA%E5%9B%BE2020315234928-1584287378785.png?token=AHBYBJ63AXOUK4ZEHTY6F226NZHNC)

### 2.非阻塞I/O
用户进程调用recvfrom后，若该缓冲区中无数据，就直接返回一个EWOULDBLOCK错误，然后进行轮循调用。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE20203160058-1584288230703.png)

### 3.I/O复用
Linux提供select/poll，进程通过将一个和多个fd传递给select/poll系统调用，阻塞在select操作上，select/poll可以检测多个fd是否就绪。select/poll是顺序检测且fd数量有限。linux提供epoll系统调用，基于事件驱动方式代替顺序扫描。当fd就绪，立即回调callball。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE20203160941-1584288618794.png)

### 4. 信号驱动I/O
开启套接字信号驱动功能，系统调用sigaction执行信号处理函数，（系统调用立即返回，进程继续执行），当数据准备就绪，为该进程生成一个SIGIO信号，通过信号回调通知应用程序去调用recvform来获取数据。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE202031601437-1584288909338.png)


### 5.异步I/O
告知内核启动某个操作，内核执行完所有操作后通知我们。信号量I/O是由内核告诉我们何时I/O，异步I/O告诉我们何时I/O完成。

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/03/16/Popo%E6%88%AA%E5%9B%BE20203160192-1584289151326.png)














