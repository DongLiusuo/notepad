# Class CountDownLatch

JDK文档的翻译：

> 一种同步帮助，它允许一个或多个线程等待，直到在其他线程中执行的一组操作完成为止。
> CountDownLatch用给定的count初始化。 由于countDown方法的调用，直到当前计数达到零为止， await方法将阻塞，此后所有释放的线程将被释放，并且任何随后的await调用将立即返回。 这是一种一次性现象-无法重置计数。 如果需要用于重置计数的版本，请考虑使用CyclicBarrier 。
> CountDownLatch是一种多功能的同步工具，可以用于多种用途。 以1的计数初始化的CountDownLatch用作简单的on / off锁存器或gate：所有调用await线程都在gate处等待，直到被countDown的线程打开countDown 。 初始化为N的CountDownLatch可以用于使一个线程等待，直到N个线程完成某个动作或某个动作已经完成N次。
> CountDownLatch一个有用属性是，它不需要调用countDown线程在继续进行操作之前就countDown等待计数达到零，它只是防止任何线程经过await状态直到所有线程都可以通过。

让一些线程阻塞直到另一些线程完成一系列操作后才被唤醒

CountDownLatch主要有两个方法，当一个或多个线程调用await方法时，调用线程会被阻塞。其他线程调用countDown方法会将计数器减1（调用counntDown方法的线程不会阻塞），当计数器变为0时，因调用await方法的线程会被唤醒，继续执行。

```java
import java.util.concurrent.CountDownLatch;

public class CountDownLatchDemo {
    public static void main(String[] args) throws InterruptedException {
        CountDownLatch countDownLatch = new CountDownLatch(6);
        for (int i = 0; i < 6; i++) {
            int finalI = i;
            new Thread(()->{
                System.out.println(finalI + "线程执行了");
                countDownLatch.countDown();
            },i+1+"").start();
        }
        countDownLatch.await();
        System.out.println("最后的线程--main执行了");
    }
}
```

# Class CyclicBarrier

CyclicBarrier的字面意思是可循环（Cyclic）使用的屏障（Barrier）。它要做的事情是，让一组线程到达一个屏障（也可以叫同步点）时被阻塞，直到最后一个线程到达屏障时，屏障才会被打开，所有被屏障拦截的线程才会继续执行，线程进入屏障通过await方法。

```java
import java.util.concurrent.CyclicBarrier;

public class CyclicBarrierDemo {

    public static void main(String[] args) {
        CyclicBarrier cyclicBarrier = new CyclicBarrier(7,()->{
            System.out.println(Thread.currentThread().getName()+"\t,最后执行的线程");
        });
        for (int i = 0; i < 7; i++) {
            int finalI = i;
            new Thread(()->{
                System.out.println(finalI + "线程执行了");
                try {
                    cyclicBarrier.await();
                } catch (Exception e) {
                    e.printStackTrace();
                }
            },i+1+"").start();
        }
    }
}

```

# Class Semaphore

信号量只要用于两个目的

* 多个共享资源的互斥使用
* 并发线程数的控制

```java

import java.util.concurrent.Semaphore;
import java.util.concurrent.TimeUnit;

public class SemaphoreDemo {

    public static void main(String[] args) {
        Semaphore semaphore = new Semaphore(3);// 模拟3个停车位
        // 模拟6部车
        for (int i = 0; i < 6; i++) {
            new Thread(()->{
                try {
                    semaphore.acquire();
                    System.out.println(Thread.currentThread().getName() + "\t抢到车位");
                    TimeUnit.SECONDS.sleep(3);// 暂停3秒  假设停车了3秒
                    System.out.println(Thread.currentThread().getName() + "\t离开车位...");
                } catch (InterruptedException e) {
                    e.printStackTrace();
                }finally {
                    semaphore.release();
                }
            },i+1+"").start();
        }
    }
}

```

