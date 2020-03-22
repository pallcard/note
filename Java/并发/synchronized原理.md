# synchronized原理

## 底层原理
Java 虚拟机中同步（synchronized）是基于进入和退出Monitor对象实现的，无论显示（同步代码）还是隐示都是如此。