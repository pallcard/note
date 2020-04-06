# interrupt

## 中断线程
线程的thread.interrupt()方法是中断线程，将会设置该线程的中断状态位，即设置为true，中断的结果线程是死亡、还是等待新的任务或是继续运行至下一步，就取决于这个程序本身。线程会不时地检测这个中断标示位，以判断线程是否应该被中断（中断标示值是否为true）。

## 判断线程是否被中断
* Thread.currentThread().isInterrupted()
    
* Thread.interrupted()
    该方法调用后会将中断标示位清除，即重新设置为false