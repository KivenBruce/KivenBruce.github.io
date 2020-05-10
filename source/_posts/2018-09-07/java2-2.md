---
title: Java进阶系列(三)-多线程编程深入
date: 2018-09-07 19:51:55
tags: java
---

进一步谈谈多线程编程一些细节的使用;涉及到线程通信的一些知识;以及线程池的使用

Q:两个线程,当线程1满足条件A时,线程2能够及时知道并且做出相应的动作,如何实现?

<!--more-->

```java
public class WayOne{
    private volatile List<String> list = new ArrayList<>();
    public void add(){
        list.add("a");        
    }
    main(){
        WayOne one = new WayOne();
        new Thread(()->{
            one.add();            
        },"t1").start();
        new Thread(()->{
            while(true){
                if(list.size == 50){
                    t1.interrupted();
                    break;
                }
            }
        },"t2")
    }
}
```

线程t2在list元素满50时,将t1线程终止,t2线程启动之前一直是处于运行中轮询list.size;

```java
public class WayTwo{
    main(){
        WayTwo two = new WayTwo();
        Object object = new Object();
        new Thread(()->{
            synchronized(object){
                if(list.size!=50){
                    **object.wait();**  //release the object lock                  
                }
                // do t2 things;
            }
        },"t2").start();
        new Thread(()->{
            synchronized(object){
                for(i=0;i<100;i++){
                    two.add();
                }
                if(list.size == 50){
                   **object.notify();**// do not release the object lock
                }
            }
        },"t1").start();
    }
}
```

注意:wait 和 notify都是object对象自身就有的方法,需要配合synchronized一起使用,运行效果可以看出,有"通知不及时的问题",当size==50的时候,t2并没有及时的do t2 things;而是等到t1运行结束,再do t2 things;原因是,**object.wait()阻塞并且释放锁,能够让t1线程获取到object锁运行,但是object.notify()并不会释放锁,所有必须等到t1运行结束,t2继续;**

```java
public class WayThree{
    main(){
        WayThree three = new WayThree();
        CountDownLantch countDownLatch = new CountDownLatch(1);
        new Thread(()->{
           for(int i=0;i<10;i++){
               three.add();
               sleep(100);
               if(list.size()==5){
                   **countDownLatch.countDown();**
               }
           }
        },"t1").start();
        new Thread(()->{
            if(list.size !=5){
                **countDownLatch.await();**
            }
            //do t2 things...
        },"t2").start();
    }
}
```

注意:countDownLatch.await();阻塞以后,知道countDownLatch.countDown()执行n次--对应CountDownLatch(n)的n;能够及时得到通知;



**Q:如何用线程实现一个队列的功能,一个负责放入,一个负责取出**

```java
public class MyQueue{
    //模拟队列容器,负责装元素
    private LinkedList<Object> list = new LinkedList<Object>();
    //模拟队列长度计数器
    private AtomicInteger count = new AtomicInteger(0);
    //设置最大深度
    private final int minSize = 0,maxSize;
    public MyQueue(int maxSize){
        this.maxSize = maxSize;
    }
    //保证有序存取,不能同时存取
    private final Object lock = new Object();
    public void put(Object obj){
        synchronized(lock){
            if(count.get() == this.maxSize){
                lock.wait();//阻塞,让别人去取
            }
            list.add(obj);
            count.increntAndGet();
            lock.notify();//通知另外线程等到我放完就可以取了
        }
    }
    
    public Object take(){
        Object ret = null;
        synchronized(lock){
            if(count.get == this.minSize){
                lock.wait();//阻塞,让别人去放
            }
            ret = list.removeFirst();
            count.decrementAndGet();
            lock.nitify();//通知另外线程等到我取完就可以放了
        }
        return ret;
    }    
    main(){
        MyQueue queue = new MyQueue(5);
        queue.put("a");
        queue.put("b");
        new Thread(()->{
            queue.put("c");
            sleep(200);
            queue.put("d");
        },"t1").start();
        new Thread(()->{
            Object o1 = queue.take();
            sleep(300);
            Object o2 = queue.take();
        },"t2").start();
    }
}
```

## 线程池的使用

列举如下四种常见的线程池:了解即可,一般不会直接这么创建线程池,都在自定义参数的;

```java
ExecutorService executorService1 = Executors.newCachedThreadPool();
ExecutorService executorService2 = Executors.newFixedThreadPool(1);
ExecutorService executorService3 = Executors.newSingleThreadExecutor();
ExecutorService executorService4 = Executors.newScheduledThreadPool(1);
```

他们都是实现了下面的接口:

```java
new ThreadPoolExecutor(int corePoolSize, int maximumPoolSize, long keepAliveTime, TimeUnit unit, BlockingQueue<Runnable> workQueue, RejectedExecutionHandler handler)
```

**注意:在使用有界队列时，若有新的任务需要执行，如果线程池实际线程数小于corePoolSize，则优先创建线程,若大于corePoolSize，则会将任务加入队列， 若队列已满，则在总线程数不大于maximumPoolSize的前提下，创建新的线程，若线程数大于maximumPoolSize，则执行拒绝策略。**maximumPoolSize,包括了corePoolSize正在运行的线程

所以,可以看出默认的四种常见线程池创建的时候,特殊情况下会造成内存溢出问题

executorService1中,没有限制线程的数量,所以会一直创建线程

executorService2中,使用无界队列,请求会一直堆积到队列中


