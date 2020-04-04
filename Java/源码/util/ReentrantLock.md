# ReentrantLock

## ReentrantLock流程

### ReentrantLockTest

main方法中通过`new ReentrantLock();`创建了一个可重入锁，然后通过线程池创建了5个线程，

```
import java.util.concurrent.ExecutorService;
import java.util.concurrent.Executors;
import java.util.concurrent.locks.ReentrantLock;

public class ReentrantLockTest {

  public static volatile int count = 0;

  public static void main(String[] args) throws InterruptedException {
    // ReentrantLock lock = new ReentrantLock(true);
    ReentrantLock lock = new ReentrantLock();
    ExecutorService executorService = Executors.newFixedThreadPool(5);
    for (int i = 0; i < 5; i++) {
      executorService.execute(()->{
        lock.lock();
        System.out.println(Thread.currentThread().getName()+ " start");
        for (int j = 0; j < 10; j++) {
          count++;
        }
        System.out.println(Thread.currentThread().getName()+ " end");
        lock.unlock();
      });
    }

    Thread.sleep(5000);
    System.out.println(count);

  }
}
```