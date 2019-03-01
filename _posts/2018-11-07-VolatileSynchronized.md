---
layout: post
title:  "Concurrency | volatile, Szychronized关键字"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JavaCore Concurrency
comment: true
description: volatile和szychronized关键字均Java中用于多线程并发。在初期学习这两个关键字的时候已经写了一些总结，可以参见[MultiThreadAndLock v1.0](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/MultiThreadAndLock.txt)， 但是几乎时隔一年，我再次翻阅这部分的内容，发现总结的部分多有疏漏，一些概念还是发生了混淆所以在本文中将会着重指出自己的一些疏漏并且加以梳理。
---
volatile和szychronized关键字均Java中用于多线程并发。在初期学习这两个关键字的时候已经写了一些总结，可以参见[MultiThreadAndLock v1.0](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/MultiThreadAndLock.txt)， 但是几乎时隔一年，我再次翻阅这部分的内容，发现总结的部分多有疏漏，一些概念还是发生了混淆所以在本文中将会着重指出自己的一些疏漏并且加以梳理。

## volatile
#### 作用域
volatile作用在域上，可以作用在静态域和非静态域上。
```Java
//作用在静态域上
protected static static Logger logger = LoggerFactory.getLogger(SyncExample.class);
//作用在非静态域上
private volatile int count = 0;
```

#### 可见性
讨论到可见性，我们必须要详细的理解JMM，可以参考我的另一篇文章[Java Memory Model](https://github.com/Seanforfun/JavaCore/blob/master/Conclusions/JMM.md)，我讲根据一张图来详细的解释可见性。
![Imgur](https://i.imgur.com/NPkfkei.png)。
这张图片中有三块存储区域, CPU寄存器， CPU缓存， 主内存。
速度： CPU寄存器 >  CPU缓存 > 主内存
只有在主内存上的变量才会是全局可见的，所以可见性的意思是修改后的对象将不会写入CPU寄存器或是CPU缓存，会直接写回到主内存。

#### 有序性
1. 在多线程中，事务的发生顺序是很重要的，但是因为编译器会在优化时将指令重排序会造成程序不按照我们的编程逻辑进行。
2. 对于volitile修饰的变量，我们可以确定happened-before原则，即对于一个volatile修饰的变量，它的在新的线程中的读取一定在写之后，或者说，一个新的线程读取变量一定在上个线程的写入之后。

#### volatile线程不安全！
* 线程不安全的例子,在这个例子中我选用了Long变量（64位）累加，因为64位变量会被JVM拆分成两个32位操作，本身不具有原子性。
```Java
@Slf4j
@ThreadNotSafe
public class SyncExample {
    protected static Logger logger = LoggerFactory.getLogger(SyncExample.class);
    @Getter
    private volatile long count = 0;
    public void add(CountDownLatch latch){
        this.count++;
        logger.info("count = {}", this.count);
        if(latch != null )latch.countDown();
    }
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executors = Executors.newCachedThreadPool();
        SyncExample syncExample = new SyncExample();
        SyncExample syncExample1 = new SyncExample();
        CountDownLatch latch = new CountDownLatch(10000);
        for (int i = 0; i < 10000; i++){
            executors.execute(() -> {
                syncExample.add(latch);
            });
        }
        latch.await();
        System.out.println(syncExample.getCount());
        executors.shutdown();
    }
}
9996
```
问题分析：此处的count++分为3步：
1. 从主存中获取最新的count值。（安全）
2. 将取出的值加一。（不安全）
3. 写回主存中。（不安全）
第一步可以保证count的值是最新的，但是整个操作不是原子性的，所以是无法保证线程安全的。

#### volatile的使用场景
作为标志位使用
```Java
public volatile boolean init = false;
//线程1
init = true；
//线程2
while(!init){   //可以确保每次读出的值都是最新的。
    sleep();
}
load(context);
```

## synchronized 可重入锁，互斥锁
对于synchronized锁定的对象，同一时间只有一个对象能对其进行访问。

### synchronized的作用域
1. 修饰代码块。对当前对象有效。
2. 修饰普通方法，对当前对象有效。
3. 修饰静态方法，对类有效。
4. 修饰一个类，对类中的所有对象有效。

#### 修饰代码块, 只对当前对象有效。如果创建了一个新的对象实例，则两个对象同时调用该方法，对于对象本身的顺序是确定的，但是两个实例的调用顺序则不确定
```Java
synchronized(this){
    // TODO
}
public void add(CountDownLatch latch){
    synchronized (this){
        this.count++;
        logger.info("count = {}", this.count);
        if(latch != null )latch.countDown();
    }
}
```

#### 修饰普通方法， 和代码块类似
```Java
//同步方法
public synchronized void add(CountDownLatch latch){
        this.count++;
        logger.info("count = {}", this.count);
        if(latch != null )latch.countDown();
    }
```

#### 修饰静态方法，作用于整个类,实际上可以理解为当前类作为了一个对象，整体上是线程安全的。
```Java
@Slf4j
@ThreadSafe
public class SyncExample {
    protected static Logger logger = LoggerFactory.getLogger(SyncExample.class);
    @Getter
    private static volatile long count = 0;
    public static synchronized void addStatic(){
        count++;
        logger.info("count = {}", count);
    }
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executors = Executors.newCachedThreadPool();
        SyncExample syncExample = new SyncExample();
        SyncExample syncExample1 = new SyncExample();
        for (int i = 0; i < 10000; i++){
            executors.execute(() -> {
                SyncExample.addStatic();
            });
        }
        for (int i = 0; i <10000 ; i++){
            executors.execute(() -> {
                SyncExample.addStatic();
            });
        }
        executors.shutdown();
    }
}
```

#### 修饰类，实际上和修饰静态方法非常类似，锁定了整个类
```Java
@Slf4j
@ThreadSafe
public class SyncExample {
    protected static Logger logger = LoggerFactory.getLogger(SyncExample.class);
    @Getter
    private static  volatile long count = 0;
    public static void add(CountDownLatch latch){
        synchronized (SyncExample.class){
            count++;
            logger.info("count = {}", count);
            if(latch != null )latch.countDown();
        }
    }
    public static void main(String[] args) throws InterruptedException {
        ExecutorService executors = Executors.newCachedThreadPool();
        SyncExample syncExample = new SyncExample();
        SyncExample syncExample1 = new SyncExample();
        for (int i = 0; i < 1000; i++){
            executors.execute(() -> {
                SyncExample.add(null);
            });
        }
        for (int i = 0; i <1000 ; i++){
            executors.execute(() -> {
                SyncExample.add(null);
            });
        }
        executors.shutdown();
    }
}
```

### 总结
1. synchronized是不可中断的，适合竞争不激烈的情况。
2. 可读性好。

### 引用
1. [Java中Volatile关键字详解](https://www.cnblogs.com/zhengbin/p/5654805.html)
2. [使用关键字volatile时出现非线程安全的原因](https://blog.csdn.net/en_joker/article/details/80338163)
