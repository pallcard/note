# synchronized原理

## 底层原理
Java 虚拟机中同步（synchronized）是基于进入和退出Monitor对象实现的，无论显示（同步代码块）还是隐示都是如此。同步方法是由调用指令读取运行时常量池中方法的ACC_SYNCHRO