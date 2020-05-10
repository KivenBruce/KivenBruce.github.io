---
title: Java进阶系列(二)-多线程编程初探
date: 2018-09-01 16:16:08
tags: java
---

了解多线程编程之前,首先回忆日常开发各个场景中是否运用到了多线程编程,为何要用,以及如何解决线程安全问题!

然后,谈谈 cpu个数、核数、线程数、Java多线程关系;最后看看几个简单的线程安全样例

<!--more-->

## 一,为什么使用多线程

通过多个线程并发执行,提高系统处理能力;

### 那是不是线程越多越好呢?

如果有大量的线程,会影响性能,因为操作系统需要在它们之间切换.线程之间的上下文切换是有时间花费的;

更多的线程需要更多的内存空间;

所以,并不是开启线程数越多越好的,受服务器硬件配置的影响,同时也与业务场景的实际需求有关;

受内核限制，Linux系统单个进程最多开启1000个线程，Windows系统单个进程最多开启2000个线程。

默认情况下，Java中每创建一个线程，操作系统会分配1M的栈空间。

### 那如何设置合适大小的线程数量呢?

如果所有的任务都是计算密集型的，则创建的多线程数 = 处理器核心数就可以了

如果io操作比较耗时，则根据具体情况调整线程数，此时 多线程数 = n*处理器核心数



## 二,如何控制线程安全性

什么场景下会出现线程安全问题?多个线程并发访问公共资源时,会有安全问题,即共享数据，才存在线程安全问题，不共享数据不存在线程安全。比如,每个线程自己维护栈内存空间,共享一片公共堆内存空间,所以当访问全局变量,并对其进行操作时,会有线程安全问题;

那么,我们就要对这些共同操作共享数据的时候,做安全处理,同步控制对共享资源的访问.常用的处理方式如下:

### synchronized关键字

即有synchronized关键字修饰的方法.
由于java的每个对象都有一个内置锁，当用此关键字修饰方法时， 
内置锁会保护整个方法。在调用该方法前，需要获得内置锁，否则就处于阻塞状态。

```java
public synchronized void save(){
    //safe logical
}
```

synchronized同步方法,锁对象，锁代码块，锁方法，锁变量;

**注： synchronized关键字也可以修饰静态方法，此时如果调用该静态方法，将会锁住整个类**
**注：同步是一种高开销的操作，因此应该尽量减少同步的内容。**

### 使用特殊域变量(volatile)实现线程同步

a.volatile关键字为域变量的访问提供了一种免锁机制， 
b.使用volatile修饰域相当于告诉虚拟机该域可能会被其他线程更新， 
c.因此每次使用该域就要重新计算，而不是使用寄存器中的值 
d.volatile不会提供任何原子操作，它也不能用来修饰final类型的变量 

注:volatile只能保证变量的一致性,即每次更新变量的值,都写入主内存,每次都从主内存中读取,但是不能保证原子性,多个线程可能同时读取得到同一个值,然后各自操作之后,同时写入主存,那么会有"覆盖"的效果,下次再读,就不是正确的值了,所以适合读取的场景,如果有涉及更新的,可以使用JDK中自带的Atomic**

注:Atomic**只能保证单个操作时原子的,不能保证多个操作时原子操作

```java
//not safe ;need add synchronized to make this function safe
public void add(){
    AtomicInteger.incrementAndGet();
    AtomicInteger.incrementAndGet();
    AtomicInteger.incrementAndGet();
    AtomicInteger.incrementAndGet();
}
```

### 使用ThreadLocal实现线程同步

如果使用ThreadLocal管理变量，则每一个使用该变量的线程都获得该变量的副本

```java
private static ThreadLocal<Integer> account = new ThreadLocal<Integer>();
```

副本之间相互独立，这样每一个线程都可以随意修改自己的变量副本，而不会对其他线程产生影响。
ThreadLocal() : 创建一个线程本地变量
get() : 返回此线程局部变量的当前线程副本中的值
initialValue() : 返回此线程局部变量的当前线程的"初始值"
set(T value) : 将此线程局部变量的当前线程副本中的值设置为value

## 三, cpu个数、核数、线程数之间的关系

cpu个数：是指物理上，也及硬件上的核心数；

核数：是逻辑上的，简单理解为逻辑上模拟出的核心数；

线程数：是同一时刻设备能并行执行的程序个数，线程数=cpu个数 * 核数【如果有超线程，再乘以超线程数】



## *`四,线程安全案例讨论`*

下面列举一些多线程样例,看看是否会有线程安全问题,为什么,又该如何避免,有更好更优雅的方式吗?



<font color="#CA0C16">demo1:</font>

```java
    private int count = 10;
    run(){
        count ++ ;
    }
    main(){
       MyThread mythread = new MyThread();
       Thread t1,t2,t3,t4,t5 = new Thread(myThread, "t1,t2,t3,t4,t5");

	   t1,t2,t3,t4,t5.start();

    }
}
```

demo1会有线程安全问题吗?

t1,t2,t3,t4,t5操作的是同一个实例mythread的count对象,多个线程同时对他进行操作,会有线程安全问题;

如何解决?

```java
synchronized run(){
    count++;
}
或者
private AtomicInteger count = new AtomicInteger(10);
run(){
    count.increaseAndGet();
}
```

**注:在demo1中,synchronized获取的都是对象的锁;如果不是同一个对象,即使加synchronized修饰,也不能阻碍其他线程访问run();**

**demo2:**

```java
public class StaticClass{
    public synchronized static void method1(){
        //do something a cost long time
    }
    public static void method2(){
    // do something b
    }
    public void mehtod3(){
    //do something c
    }
    main(){
        StaticClass a;
        StaticClass b;
        new thread(()->{
            a.method1();//// process a
        }).start();
        new thread(()->{
            a.method1();//wait process a done;
            a.method2();//wait process a done;
            a.method3();//do not need to wait;
            b.method1();//wait process a done;
            b.method2();//wait process a done;
            b.method3();//do not need to wait;
         }).start();
    }
}
```

**结论:如果访问的是静态方法,锁住整个类对象,但是不会妨碍自己或者其他线程访问其他非synchronized的对象或者方法**



**demo3:deadlock**

```java
public class DeadLock implements Runnable{
    private static Object object1 = new Object();
    private static Object object2 = new Object();
    private String flag = null;
    setFlag(String flag){
        this.flag = flag;        
    }
    run(){
        if("a".equals(flag)){
           synchronized(object1){
             sleep(5000);
             synchronized(object2){
                 //do something
             }
         } else {
             synchronized(object2){
                 sleep(5000);
                 synchronized(object1){
                     //do something
                 }
             }
         }
        }
       main(){
           DeadLock d1 = new DeadLock();
           DeadLock d2 = new DeadLock();
           new thread(d1,"t1").start();
           new thread(d2,"t2").start();
       }
    }

}
```

用jconsole命令查看死锁情况如下:

![java2-1\deadlock](java2-1\deadlock.png)
