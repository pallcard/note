# JDK 并发工具包
## 引言

JDK并发包中提供了CountDownLatch、CyclicBarrier、Semaphore工具类来实现并发流程的控制。
Exchanger提供了线程间交换数据的方法。

## join()方法

需求：实现主程序要等待其他线程完成后，在继续执行。很容易想到使用join()方法来实现。

```java
public class JoinCountDownLatchTest {
    public static void main(String[] args) throws InterruptedException {
        Thread thread1 = new Thread(() -> {
            System.out.println("执行1");
        });
        Thread thread2 = new Thread(() -> {
            System.out.println("执行2");
        });

        thread1.start();
        thread2.start();

        thread1.join();
        thread2.join();
        System.out.println("主线程");
    }
}

/* 输出结果：
执行1
执行2
主线程
*/
```

join()原理是不停的检查join线程是否存活，直到join线程都中止，线程才会this.notifyAll()。


## CountDownLatch类
```java
public class CountDownLatchTest {

    public static void main(String[] args) throws InterruptedException {

        CountDownLatch countDownLatch = new CountDownLatch(2);
        Thread thread1 = new Thread(() -> {
            System.out.println("执行1");
            countDownLatch.countDown();
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("执行2");
            countDownLatch.countDown();  
        });
        thread1.start();
        thread2.start();

        countDownLatch.await(10, TimeUnit.MILLISECONDS);

        System.out.println("主线程");
    }
}
```
说明：

CountDownLatch类的构造函数需要传入数字N，表示你想等待N个点（N个线程或N个步骤）。上述代码中，
传入的N为2，则表示主线程要等待两个点后，才会继续执行主程序，在此之前程序会一直阻塞在countDownLatch.await()方法这，
（上述程序的await方法带一个指定时间，只会阻塞到指定时间）。其中调用countDown()函数时，N会减1，当N=0时，
结束阻塞。

若将上述类中的方法该成如下代码，则执行1，执行2会在主线程之前打印，但是等待i不一定会在主线程之前打印，
想要实现，需要将构造中的N改为3，在for循环后面调用countDown()方法。

```java
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(2);
        Thread thread1 = new Thread(() -> {
            System.out.println("执行1");
            countDownLatch.countDown();
        });

        Thread thread2 = new Thread(() -> {
            System.out.println("执行2");
            countDownLatch.countDown();
            for(int i = 0; i < 1000; i++) {
                System.out.println("等待"+ (i+1));
            }
//            countDownLatch.countDown();
        });
        thread1.start();
        thread2.start();

        countDownLatch.await();

        System.out.println("主线程");
    }
```

## CyclicBarrier类

```java
public class CyclicBarrierTest {
    public static void main(String[] args) throws InterruptedException {

        CyclicBarrier cyclicBarrier = new CyclicBarrier(3);
        // 若将3改为4，程序会一直等待，没有4个线程到达屏障
//        CyclicBarrier cyclicBarrier = new CyclicBarrier(4);

        // 若将3改为4，程序会一直等待，没有4个线程到达屏障, 优先执行当任务也不会执行
        CyclicBarrier cyclicBarrier2 = new CyclicBarrier(3, new Runnable() {
            @Override
            public void run() {
                System.out.println("优先执行");
            }
        });

        Thread thread1 = new Thread(() -> {
            for (int i = 0; i < 10; i++) {
                System.out.println("到达屏障前");
            }
            try {
                cyclicBarrier.await();
                cyclicBarrier2.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("执行1");
        });

        Thread thread2 = new Thread(() -> {
            try {
                cyclicBarrier.await();
                cyclicBarrier2.await();
            } catch (Exception e) {
                e.printStackTrace();
            }
            System.out.println("执行2");
        });

        thread1.start();
        thread2.start();

        try {
            cyclicBarrier.await();
            cyclicBarrier2.await();
        } catch (Exception e) {
            e.printStackTrace();
        }
        Thread.sleep(1000);

        System.out.println("主线程");
    }
}
```
CyclicBarrier让一组线程到达一个屏障时被阻塞，例如，上述代码若将屏障数设置为4，则只会打印到达屏障前，
然后就会3个线程都会被阻塞。在CyclicBarrier的构造函数中也可以添加一个任务，到线程数达到屏障要求时，线程
继续执行，并且构造函数中的会优先执行。

## 实例

计算银行的账单流水，使用多个线程进行计算，然后进行汇总。

```java
public class BankWaterService implements Runnable{

    // 4个屏障处理完成，执行当前类当run方法
    private CyclicBarrier cyclicBarrier = new CyclicBarrier(4, this);

    private Executor executor = Executors.newFixedThreadPool(4);

    private ConcurrentHashMap<String, Integer> sheetBankWaterCount = new ConcurrentHashMap<>();

    @Override
    public void run() {
        int result = 0;
        for(Map.Entry<String, Integer> sheet : sheetBankWaterCount.entrySet()) {
            result += sheet.getValue();
        }

        sheetBankWaterCount.put("result", result);
        System.out.println(result);
    }

    private void count(){
        for (int i = 0; i < 4; i++) {
            executor.execute(()->{
                sheetBankWaterCount.put(Thread.currentThread().getName(), 1);
                System.out.println(Thread.currentThread().getName() + 1);
                try {
                    cyclicBarrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            });
        }

    }

    public static void main(String[] args) {
        BankWaterService bankWaterService = new BankWaterService();
        bankWaterService.count();
    }
}
```
说明：

首先屏障设置为4，优先执行的任务传入this，表示当4个线程到达后会优先执行类中的run()方法。对于4个线程
到计算完自己的流水后到达屏障阻塞，到4个线程都计算完毕后，会优先执行本类中的run()方法，从而完成了对于
流水的统计。

**注：**

CyclicBarrier的计数器可以使用reset()方法重置，这正好解释了Cyclic循环的意思。

### Semaphore类

```java
public class SemaphoreTest {

    private static final int THREAD_COUNT = 30;

    private static ExecutorService threadPool = Executors.newFixedThreadPool(THREAD_COUNT);

    private static Semaphore s = new Semaphore(10);

    public static void main(String[] args) {
        for (int i = 0; i < THREAD_COUNT; i++) {
            threadPool.execute(()->{
                try {
                    s.acquire();
                    System.out.println("save data");
                    s.release();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }
            });
        }
        threadPool.shutdown();
    }
}
```
Semaphore类可以控制同时访问特定资源的线程数量。例如上述代码有30个线程，但是只能有10个线程来访问数据库（打印save data）。

### Exchanger类
```java
public class ExchangerTest {

    private static final Exchanger<String> exgr = new Exchanger<>();

    private static ExecutorService threadPool = Executors.newFixedThreadPool(2);

    public static void main(String[] args) {

        threadPool.execute(()->{
            String A = "流水A";
            try {
                String B = exgr.exchange(A + Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName()+A.equals(B)+ " A: " + A + " B: " + B);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        threadPool.execute(()->{
            String B = "流水B";
            try {
                String A = exgr.exchange(B+ Thread.currentThread().getName());
                System.out.println(Thread.currentThread().getName()+A.equals(B)+ " A: " + A + " B: " + B);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        });

        threadPool.shutdown();
    }
}
```

可以控制两个线程交换彼此数据。如果两个线程有一个exchange()方法没有执行，则会一直等待。exchange()方法里也可以设置最大等待时间。

参考：java并发编程艺术
