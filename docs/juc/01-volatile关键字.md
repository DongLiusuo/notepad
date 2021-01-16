# 请谈谈你对volatile的理解

 ## 1. volatile是Java虚拟机提供的轻量级的同步机制

   - 保证可见性

   - 不保证原子性

   - 禁止指令重排

     volatile是Java虚拟机提供的轻量级的同步机制，他保证可见性，不保证原子性，禁止指令重排。
     
     ```java
     class MyData {
         volatile int number = 0;
         public void addTo60() {
             number = 60;
         }
     }
     
     /**
      * 1 验证volatile的可见性
      *   假设int number = 0,number变量没有添加volatile关键字修饰,即没有可见性
      */
     public class VolatileDemo {
         public static void main(String[] args) { // main线程
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

使用原子类代替，

##### 有序性

计算机在执行程序时，为了提高性能，编译器和处理器常常会对指令做重排，一般分为3种

![1610813773099](01-volatile%E5%85%B3%E9%94%AE%E5%AD%97/1610813773099.png)

单线程环境里面确保程序最终执行结果和代码顺序执行的结果一致。

处理器在进行重排序时不许要考虑指令之间的数据依赖性

多线程环境中线程交替执行，由于编译器优化重排的存在，两个线程中使用的变量能否保证一致性是无法确定的，结果无法预测

 ## 3.  你在哪些地方用过volatile？