# 请谈谈你对volatile的理解

 ## 1. volatile是Java虚拟机提供的轻量级的同步机制

   - 保证可见性

   - 不保证原子性

   - 禁止指令重排

     volatile是Java虚拟机提供的轻量级的同步机制，他保证可见性，不保证原子性，禁止指令重排。
     
     ```java
     import java.util.concurrent.atomic.AtomicInteger;
     
     class MyData {
     
         volatile int number = 0;
     
         public void addTo60() {
             number = 60;
         }
     
         // number是有volatile修饰的
         public void addPlusPlus() {
             number++;
         }
     
         AtomicInteger atomicInteger = new AtomicInteger();
         public void addAtomic() {
             atomicInteger.getAndIncrement();
         }
     }
    
     /**
      * 1 验证volatile的可见性
      *   假设int number = 0,number变量没有添加volatile关键字修饰,即没有可见性
      *   添加了volatile，可以解决可见性的问题。
      * 2 验证volatile不保证原子性
      *   原子性是什么意思？
      *      不可分割，完整性，也即某个线程正在做某个具体业务时，中间不可以被加塞或者被分割。需要整体完整，要么同时成功，要么同时失败。
      *   volatile不保证原子性的案例展示
      *   如何解决原子性？
      *      * 加synchronized
      *      * 使用原子类
      */
     public class VolatileDemo {
         public static void main(String[] args) { // main线程
     
         }
     
         //volatile不保证原子性的案例展示
         private static void notAtmocValtile() {
             MyData myData = new MyData();
             //开20个线程 每个线程加1k次
             for (int i = 0; i < 20; i++) {
                 new Thread(()->{
                     for (int j = 0; j < 1000; j++) {
                         myData.addPlusPlus();
                         myData.addAtomic();
                     }
                 },i+1+"").start();
             }
             //main取得number的值
             // 需要等待以上线程都计算完成后，再用main线程取值
             while (Thread.activeCount() > 2) { //2:mian和gc线程
                 Thread.yield();//礼让
             }
             System.out.println(Thread.currentThread().getName()+"\tfinally number  is "+myData.number+"\t atmocNum is "+myData.atomicInteger.get());
         }
     
         // volatile可以保证可见性，及时通知其他线程，主内存的值被修改了
         private static void seeOkVolatile() {
             MyData myData = new MyData(); //资源类
     
             // 第一个线程修改值
             new Thread(()->{
                 System.out.println(Thread.currentThread().getName()+"线程执行");
                 // 暂停一会线程
                 try { Thread.sleep(3000); } catch (InterruptedException e) { e.printStackTrace(); }
                 myData.addTo60();
                 System.out.println(Thread.currentThread().getName() + "线程执行修改了值：" + myData.number);
             },"A").start();
     
             //第二个线程是main线程
             while (myData.number == 0) {
                 // main一直在这里等待 直到number的值不再是0  不加volatile修饰 A线程的修改的值对于main来说不可见
             }
             System.out.println(Thread.currentThread().getName()+"\t mission over, main get number value is "+myData.number);
         }
     
     }
     
     ```
     
     

 ## 2.  JMM你谈谈

   JVM：Java虚拟机

   JMM：Java内存模型

   > JMM（Java内存模型Java Momery Model，简称JMM）本身是一种抽象的概念，**并不真实存在**，它描述的是一组规则或规范，通过规范定制了程序中各个变量(包括实例字段,静态字段和构成数组对象的元素)的访问方式。

   #### JMM关于同步规定:

   1.线程解锁前,必须把共享变量的值刷新回主内存
   2.线程加锁前,必须读取主内存的最新值到自己的工作内存
   3.加锁解锁是同一把锁

   #### JMM内存模型

   由于JVM运行程序的实体是线程,而每个线程创建时JVM都会为其创建一个工作内存(有些地方称为栈空间)，工作内存是每个线程的私有数据区域，而Java内存模型中规定所有变量都存储在**主内存**，主内存是共享内存区域，所有线程都可访问，**但线程对变量的操作(读取赋值等)必须在工作内存中进行，首先要将变量从主内存拷贝到自己的工作空间，然后对变量进行操作,操作完成再将变量写回主内存**,不能直接操作主内存中的变量,各个线程中的工作内存储存着主内存中的*变量副本拷贝*，因此不同的线程无法访问对方的工作内存，线程间的通讯(传值) 必须通过主内存来完成，其简要访问过程如下图：

   ![1610806097649](01-volatile%E5%85%B3%E9%94%AE%E5%AD%97/1610806097649.png)

   #### JMM的三大特性

##### 可见性

通过前面对JMM的介绍，我们知道 各个线程对主内存中共享变量的操作都是各个线程各自拷贝到自己的工作内存进行操作后再写回到主内存中的。 

这就可能存在一个线程AAA修改了共享变量X的值但还未写回到主内存时，另外一个线程BBB又对主内存中的同一个变量X进行操作，但此时A线程工作内存中的共享变量X对线程B来说并不可见，这种工作内存与主内存同步延迟现象就造成了可见性问题。

##### 原子性

###### number++在多线程下是非线程安全的，如何不加synchronized解决？

使用原子类代替。

##### 有序性

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排，一般分为3种

![1610813773099](01-volatile%E5%85%B3%E9%94%AE%E5%AD%97/1610813773099.png)

单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。

处理器在进行重排序时不许要考虑指令之间的数据依赖性

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测



> 工作内存与主内存同步延迟现象导致的可见性问题，可以使用synchorized或volatile关键字解决，他们都可以使一个线程修改后的变量立即对其他线程可见。
>
> 对于指令重排导致的可见性问题和有序性问题，可以利用volatile关键字解决，因为volatile的另外一个作用就是禁止重排序优化

## 3.  你在哪些地方用过volatile？

## 单例模式DCL代码

```java
public class Singleton {

    private static Singleton instance = null;

    private Singleton(){
        System.out.println(Thread.currentThread().getName() + "线程\t,构造器执行了");
    }

    public static Singleton getInstance() {
        if (instance == null) instance = new Singleton();
        return instance;
    }

    public static void main(String[] args) {
        // 单线程下  没有问题
        //System.out.println(Singleton.getInstance()==Singleton.getInstance());

        // 多线程下，情况发生了变化
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try { Thread.sleep(2000); } catch (InterruptedException e) { e.printStackTrace(); }
                Singleton.getInstance();
            }, i + 1 + "").start();
        }
    }
}
```

测试结果：多线程下，无法保证对象使单例的。

![1610897825797](01-volatile%E5%85%B3%E9%94%AE%E5%AD%97/1610897825797.png)

## 单例模式volatile分析

```java
 public class Singleton {

    private static volatile Singleton instance = null; // volatile禁止指令重排

    private Singleton(){
        System.out.println(Thread.currentThread().getName() + "线程\t构造器执行了");
    }

    /*
     * DCL模式：double check lock 双端检锁机制
     * DCL机制不一定线程安全，原因是有指令重排的存在，加入volatile可以禁止指令重排序。
     * 原因在于某一个线程执行到第一次检测，读取到instance不为null时，instance的引用对象可能没有完成初始化，
     * instance = new Singleton(); 可分为以下3步完成（伪代码）
     *   memory = allocate(); //1分配对象内存空间
     *   instance(memory);    //2初始化对象
     *   instance = memory;   //3设置instance指向刚分配的内存地址，此时instance!=null
     * 由于步骤2和步骤3不存在数据依赖关系，而且无论重排前还是重排后的执行结果在单线程中并没有改变，
     * 因此这种重排优化是允许的。
     *   memory = allocate(); //1分配对象内存空间
     *   instance = memory;   //3设置instance指向刚分配的内存地址，此时instance!=null，但是对象的初始化还没有完成！
     *   instance(memory);    //2初始化对象
     * 但是指令重排只会保证串行语义的执行的一致性（单线程），但并不会关心多线程间的语义一致性。
     * 所以当一条线程访问instance不为null时，由于instance实例未必已经初始化完成，也就造成了线程安全问题。
     */
    public static Singleton getInstance() {
        if (instance == null) {
            synchronized (Singleton.class) {
                if (instance == null) instance = new Singleton();
            }
        }
        return instance;
    }

    public static void main(String[] args) {
        // 单线程下  没有问题
        //System.out.println(Singleton.getInstance()==Singleton.getInstance());

        // 多线程下，情况发生了变化
        for (int i = 0; i < 10; i++) {
            new Thread(() -> {
                try { Thread.sleep(2000); } catch (InterruptedException e) { e.printStackTrace(); }
                Singleton.getInstance();
            }, i + 1 + "").start();
        }
    }
}
```

