# ReentrantLock

## ReentrantLock流程

### 测试代码

main方法中通过`new ReentrantLock();`创建了一个可重入锁，然后通过线程池创建了5个线程，在各个线程中通过`rlock.lock();`进行加锁，然后执行逻辑，最后通过`rlock.unlock();`进行解锁。代码如下：

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest {

  public static volatile int count = 0;

  public static void main(String[] args) throws InterruptedException {
    // ReentrantLock rlock = new ReentrantLock(true);
    ReentrantLock rlock = new ReentrantLock();
    ExecutorService executorService = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 5; i++) {
      executorService.execute(()->{
        rlock.lock();
        System.out.println(Thread.currentThread().getName()+ " start");
        for (int j = 0; j < 10; j++) {
          System.out.println(Thread.currentThread().getName()+ " " +count++);
        }
        System.out.println(Thread.currentThread().getName()+ " end");
        rlock.unlock();
      });
    }

    Thread.sleep(5000);
    System.out.println(count);

  }
}
```

### 上锁流程

![title](https://raw.githubusercontent.com/pallcard/noteImg/master/noteImg/2020/04/04/1585987486634-1585987486636.png)





















