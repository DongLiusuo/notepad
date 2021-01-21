# 公平和非公平锁

并发中的ReentrantLock创建可以指定构造器的boolean类型来得到公平锁和非公平锁，默认是非公平锁。

### 是什么

##### 公平锁

是指多个线程按照申请锁的顺序来获取锁，先来后到。

##### 非公平锁

是指多个线程按照获取锁的顺序并不是按照申请锁的顺序，有可能后申请的线程比先申请的线程优先获取锁。在高并发的情况下，有可能会造成优先级反转或饥饿现象。

### 两者区别

**公平锁**：Threads acquire  a fair lock in the order in which they request it

>  公平锁，就是很公平，在并发环境中，每个线程在获取锁时会先查看次锁维护的等待队列，如果为空，或者当前线程是等待队列的第一个，就占有锁，否则就会加入到等待队列中，以后会按照FIFO的规则从队列中取到自己。

**非公平锁**：a nonfair lock permits barging: threads requesting a lock can jump ahead of the queue of waiting threads if the lock happens to be available when it is requested.

> 非公平锁比较粗鲁，上来就直接占有锁，如果尝试失败，就再采用类似公平锁那种方式。

### 其他

Java ReetrantLock而言，通过构造器参数指定该锁是否是公平锁，默认是非公平锁。非公平锁的优点在于吞吐量比公平锁大

synchronized而言，也是一种非公平锁。

# 可重入锁（递归锁）

### 是什么

可重入锁，又叫递归锁。

指的是同一线程外层函数获得锁之后，内层递归函数仍能获取该锁的代码。

在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁

也就是说，线程可以进入任何一个他 已经拥有的锁同步着的代码块。

### ReentrantLock/synchronized就是一个典型的可重入锁

可重入锁最大的作用是避免死锁

```java

class Phone implements Runnable {

    synchronized public void sendSms() {
        System.out.println(Thread.currentThread().getId()+"\t invoked sendSms method");
        sendEmail();
    }
    synchronized public void sendEmail() {
        System.out.println(Thread.currentThread().getId()+"\t invoked sendEmail..... method");
    }

    //=================================================

    Lock lock = new ReentrantLock();

    @Override
    public void run() {
        get();
    }

    private void get() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getId()+"\t invoked get method");
            set();
        }finally {
            lock.unlock();
        }
    }

    private void set() {
        lock.lock();
        try {
            System.out.println(Thread.currentThread().getId()+"\t invoked set... method");
        }finally {
            lock.unlock();
        }
    }

}

/**
 * 可重入锁（也叫递归锁）
 * 指的是同一线程外层函数获取锁之后，内层递归函数仍然能获取该锁的代码，
 * 在同一个线程在外层方法获取锁的时候，在进入内层方法会自动获取锁
 *
 * 也就是说，线程可以进入任何一个他已经拥有的锁所同步着的代码块
 */
public class ReentrantLockDemo {
    public static void main(String[] args) throws Exception{
        Phone phone = new Phone();
        new Thread(() -> phone.sendSms(), "t1").start();
        new Thread(() -> phone.sendSms(), "t2").start();

        Thread.sleep(1000);
        System.out.println();
        System.out.println();
        System.out.println();

        new Thread(phone,"t10").start();
        new Thread(phone,"t20").start();
    }
}
```



# 自旋锁

### 是什么

自旋锁（spinlock）是指尝试获取锁的线程不会立即阻塞，而是采用循环的方式获取锁，这样的好处是减少线程上下文切换的消耗，缺点是循环会消耗CPU 

### 好处

循环比较获取直到成功为止，没有类似wait的阻塞

### 手写一个自旋锁

```java

import java.util.concurrent.atomic.AtomicReference;

/**
 * 请手写一个自旋锁
 *
 * 通过CAS操作完成自旋锁，
 * A线程先进来调用myLock方法持有锁5秒钟，
 * B随后进来发现当前已经有线程持有锁，不是null，所以只能通过自旋等待，直到A释放锁后B随后抢到
 */

public class SpinLockDemo {

    // 原子引用线程
    AtomicReference<Thread> atomicReference = new AtomicReference<>();

    public void myLock() {
        Thread thread = Thread.currentThread();
        while (!atomicReference.compareAndSet(null, thread)) {

        }
        System.out.println(thread.getName() + "\t invoked myLock");
    }

    public void myUnLock() {
        Thread thread = Thread.currentThread();
        atomicReference.compareAndSet(thread, null);
        System.out.println(thread.getName() + "\t invoked myUnLock");
    }

    public static void main(String[] args) {
        SpinLockDemo spinLockDemo = new SpinLockDemo();
        new Thread(()->{
            spinLockDemo.myLock();
            try {
                Thread.sleep(5000);
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
            spinLockDemo.myUnLock();
        },"AA").start();

        try {
            Thread.sleep(1000);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }

        new Thread(()->{
            spinLockDemo.myLock();
            spinLockDemo.myUnLock();
        },"BB").start();

    }
}

```

# 独占锁（写锁）、共享锁（读锁）、互斥锁

### 独占锁

独占锁指该锁只能被一个线程所持有。对ReentrantLock和synchronized而言都是独占锁

### 共享锁

指该锁可被多个线程所持有。对ReentrantReadLock其读锁是共享锁，其写锁是独占锁，读锁是共享锁。

读锁可保证并发读是非常高效的，读写、写读、写写都是互斥的。

```java
import java.util.HashMap;
import java.util.Map;
import java.util.concurrent.locks.ReentrantReadWriteLock;

/**
 * 多个线程同时读一个资源类没有任何问题，所以为了满足并发量，读取共享资源应该可以同时进行。
 * 但是如果有一个线程去写共享资源，就不应该再有其他线程可以对该资源进行读或写
 *
 * 写操作：原子性、独占性。
 */
public class ReentrantReadWriteLockDemo {
    public static void main(String[] args) {
        MyCache myCache = new MyCache();
        // 5个线程写
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            new Thread(()->{
                myCache.put(finalI + "", finalI + "");
            },i+1+"线程").start();
        }
        // 5个线程读
        for (int i = 0; i < 5; i++) {
            int finalI = i;
            int finalI1 = i;
            new Thread(()->{
                Object rt = myCache.get(finalI1 + "");
            },i+1+"线程").start();
        }
    }
}

// 模拟简单的分布式缓存
class MyCache{
    private final Map<String, Object> map = new HashMap<>();
    private final ReentrantReadWriteLock rwLock = new ReentrantReadWriteLock();


    public void put(String key, Object value) {
        rwLock.writeLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t正在写入：" + key);
            // 模拟网络拥堵
            try {
                Thread.sleep(1000);
            } catch (Exception e) {
                e.printStackTrace();
            }
            map.put(key, value);
            System.out.println(Thread.currentThread().getName() + "\t写入完成。" + key);
        }finally {
            rwLock.writeLock().unlock();
        }
    }
    public Object get(String key) {
        rwLock.readLock().lock();
        try {
            System.out.println(Thread.currentThread().getName() + "\t正在读取：" );
            Object rt = map.get(key);
            System.out.println(Thread.currentThread().getName() + "\t读取完成。" + rt);
            return rt;
        }finally {
            rwLock.readLock().unlock();
        }
    }

    public void clearCache() {
        map.clear();
    }
}

```

