---
layout: post
title:  "CountDownLatch,CyclicBarrier,Semaphore"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: Concurrent
comment: true
description: 这三个类都是concurrent包的辅助类，用于帮助我们确定线程并发的安全性。
---
这三个类都是concurrent包的辅助类，用于帮助我们确定线程并发的安全性。均实现了AQS(Abstract Queued Synchronizer)接口，适用于线程同步的工具。

### CountDownLatch
![Imgur](https://i.imgur.com/tujncqX.png)
CountDownLatch类位于java.util.concurrent包下，利用它可以实现类似计数器的功能。比如有一个任务A，它要等待其他4个任务执行完毕之后才能执行，此时就可以利用CountDownLatch来实现这种功能，指的是线程等待外部的一个状态。
* CountDownLatch的定义
这个类只有一个构造器，定义一个初始值，并且从这个值开始往下计数。
```Java
CountDownLatch latch = new CountDownLatch(count);
```

* CountDownLatch的重要方法
```Java
public void await() throws InterruptedException; //该方法一致阻塞直到latch的值变成0.
public boolean await(long timeout, TimeUnit unit)；//如果latch变成0,则返回true，或者阻塞的时间到了也会返回。
public void countDown(); //将latch减一；
```

* 通过CountDownLatch来确定线程的执行顺序的例子
```Java
public class CountDownLatchConclusion {
    private static int count = 0;
    public static void main(String[] args) {
        CountDownLatch latch1 = new CountDownLatch(1);
        CountDownLatch latch2 = new CountDownLatch(1);
        new Thread(() -> {
            count++;
            System.out.println("[target- 1]: finish counting.");
            latch1.countDown();
        }, "target-1").start();
        new Thread(() -> {
            try {
                latch1.await(); //等待target1线程结束
                count++;
                System.out.println("[target- 2]: finish counting.");
                latch2.countDown();
            } catch (InterruptedException e) {
                e.printStackTrace();
            }
        }, "target-2").start();
        try {
            latch2.await(); //等待target2线程结束。
            System.out.println("[Main]: Count is " + count);
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
此处三个线程的执行顺序是确定的：target1 -> target2 -> main
[target- 1]: finish counting.
[target- 2]: finish counting.
[Main]: Count is 2
```

### CyclicBarrier
字面意思回环栅栏，通过它可以实现让一组线程等待至某个状态之后再全部同时执行，是线程之间的相互等待。叫做回环是因为当所有等待线程都被释放以后，CyclicBarrier可以被重用。我们暂且把这个状态就叫做barrier，当调用await()方法之后，线程就处于barrier了。

* CyclicBarrier的创建
```Java
// parties: 在调用之前必须使用有parties个线程被调用。
// barrierAction： 当条件满足后的回调方法。
public CyclicBarrier(int parties, Runnable barrierAction);
public CyclicBarrier(int parties)；
```

* CyclicBarrier的重要方法
```Java
//阻塞方法，和CountDownLatch用法一致
public int await() throws InterruptedException, BrokenBarrierException { };
public int await(long timeout, TimeUnit unit)throws InterruptedException,BrokenBarrierException,TimeoutException { };
```

* CyclicBarrier的例子，在所有的线程都执行完后，会打印Finish
```Java
public class CyclicBarrierConclusion {
    private static final int N = 4;
    private static CyclicBarrier barrier = new CyclicBarrier(N, () -> {
        System.out.println("Finished all writting process.");
    });
    public static void main(String[] args) {
        for(int i = 0; i < N; i++){
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " is writting to file.");
                try {
                    Thread.sleep(5000L);
                    System.out.println(Thread.currentThread().getName() + " finished writting.");
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, "Thread-" + i).start();
        }
    }
}
```

* CyclicBarrier的重用
```Java
public class CyclicBarrierConclusion {
    private static final int N = 4;
    private static CyclicBarrier barrier = new CyclicBarrier(N, () -> {
        System.out.println("Finished all writting process.");
    });
    public static void main(String[] args) throws InterruptedException {
        writeToFile();
        Thread.sleep(10000L);
        System.out.println("Barrier reuse.");
        writeToFile();
    }
    private static void writeToFile(){
        for(int i = 0; i < N; i++){
            new Thread(() -> {
                System.out.println(Thread.currentThread().getName() + " is writting to file.");
                try {
                    Thread.sleep(5000L);
                    System.out.println(Thread.currentThread().getName() + " finished writting.");
                    barrier.await();
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } catch (BrokenBarrierException e) {
                    e.printStackTrace();
                }
            }, "Thread-" + i).start();
        }
    }
}
Thread-0 is writting to file.
Thread-2 is writting to file.
Thread-1 is writting to file.
Thread-3 is writting to file.
Thread-2 finished writting.
Thread-0 finished writting.
Thread-3 finished writting.
Thread-1 finished writting.
Finished all writting process.
Barrier reuse.
Thread-0 is writting to file.
Thread-1 is writting to file.
Thread-2 is writting to file.
Thread-3 is writting to file.
Thread-1 finished writting.
Thread-0 finished writting.
Thread-2 finished writting.
Thread-3 finished writting.
Finished all writting process.
```

### Semaphora信号量
Semaphore可以控同时访问的线程个数，通过 acquire() 获取一个许可，如果没有就等待，而 release() 释放一个许可。实际上信号量的使用很像锁。

* Semaphore的创建
```Java
public Semaphore(int permits) {          //参数permits表示许可数目，即同时可以允许多少线程进行访问
    sync = new NonfairSync(permits);
}
public Semaphore(int permits, boolean fair) {    //这个多了一个参数fair表示是否是公平的，即等待时间越久的越先获取许可
    sync = (fair)? new FairSync(permits) : new NonfairSync(permits);
}
```

* Semaphora的重要方法
```Java
// 阻塞式的
// acquire()用来获取一个许可，若无许可能够获得，则会一直等待，直到获得许可。
public void acquire() throws InterruptedException {  }     //获取一个许可
public void acquire(int permits) throws InterruptedException { }    //获取permits个许可
// release()用来释放许可。注意，在释放许可之前，必须先获获得许可。
public void release() { }          //释放一个许可
public void release(int permits) { }    //释放permits个许可

// 非阻塞式
public boolean tryAcquire() { };    //尝试获取一个许可，若获取成功，则立即返回true，若获取失败，则立即返回false，个人觉得这个方法应该是在循环中使用的。
public boolean tryAcquire(long timeout, TimeUnit unit) throws InterruptedException { };  //尝试获取一个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
public boolean tryAcquire(int permits) { }; //尝试获取permits个许可，若获取成功，则立即返回true，若获取失败，则立即返回false
public boolean tryAcquire(int permits, long timeout, TimeUnit unit) throws InterruptedException { }; //尝试获取permits个许可，若在指定的时间内获取成功，则立即返回true，否则则立即返回false
```

* 信号量的例子, 个人觉得信号量有些像锁的集合，一次性锁住多种对象。
```Java
public class SemaphoraStudy implements Runnable{
    private static final int RESOURCE_NUM = 5;
    private static final int WORKER_NUM = 8;
    private Integer id = null;
    private Semaphore semaphore = null;
    public SemaphoraStudy(int id,  Semaphore semaphore) {
        this.id = id;
        this.semaphore = semaphore;
    }
    public static void main(String[] args) {
        ExecutorService executors = Executors.newCachedThreadPool();
        Semaphore semaphore = new Semaphore(RESOURCE_NUM);
        for(int i = 0; i < WORKER_NUM; i++){
            executors.execute(new SemaphoraStudy(i, semaphore));
        }
    }
    @Override
    public void run() {
        try {
            semaphore.acquire();
            System.out.println("Worker-" + this.id +" got the resource.");
            Thread.sleep(2000L);
            System.out.println("Worker-" + this.id +" released the resource.");
            semaphore.release();
        } catch (InterruptedException e) {
            e.printStackTrace();
        }
    }
}
Worker-3 got the resource.
Worker-0 got the resource.
Worker-2 got the resource.
Worker-4 got the resource.
Worker-1 got the resource.
Worker-3 released the resource.
Worker-5 got the resource.
Worker-0 released the resource.
Worker-6 got the resource.
Worker-2 released the resource.
Worker-7 got the resource.
Worker-4 released the resource.
Worker-1 released the resource.
Worker-5 released the resource.
Worker-6 released the resource.
Worker-7 released the resource.
```

#### 尝试获取信号量,如果无法获取则会丢去所有的阻塞的线程。
```Java
@Slf4j
public class TestSemaphore {
    private static final int TASK_NUM = 100;
    private static final int THREAD_NUM = 10;

    public static void main(String[] args) throws InterruptedException {
        Semaphore semaphore = new Semaphore(THREAD_NUM);
        CountDownLatch latch = new CountDownLatch(TASK_NUM);
        ExecutorService executors = Executors.newCachedThreadPool();
        for (int i = 0; i < TASK_NUM; i++){
            final int count =  i;
            executors.execute(() -> {
                try {
                    if (semaphore.tryAcquire(2)){
                        log(count);
                        semaphore.release(2);
                    }
                } catch (InterruptedException e) {
                    e.printStackTrace();
                } finally {
                    latch.countDown();
                }
            });
        }
        latch.await();
        log.info("finish");
        executors.shutdown();
    }
    public static void log(int i) throws InterruptedException {
        log.info("{}", i);
        Thread.sleep(1000);
    }
}
19:35:03.560 [pool-1-thread-4] INFO ca.mcmaster.concurrent.TestSemaphore - 3
19:35:03.560 [pool-1-thread-6] INFO ca.mcmaster.concurrent.TestSemaphore - 5
19:35:03.560 [pool-1-thread-2] INFO ca.mcmaster.concurrent.TestSemaphore - 1
19:35:03.560 [pool-1-thread-1] INFO ca.mcmaster.concurrent.TestSemaphore - 0
19:35:03.560 [pool-1-thread-5] INFO ca.mcmaster.concurrent.TestSemaphore - 4
19:35:04.564 [main] INFO ca.mcmaster.concurrent.TestSemaphore - finish
```

### Conclusion
1. CountDownLatch和CyclicBarrier都能够实现线程之间的等待，只不过它们侧重点不同：CountDownLatch一般用于某个线程A等待若干个其他线程执行完任务之后，它才执行；而CyclicBarrier一般用于一组线程互相等待至某个状态，然后这一组线程再同时执行；另外，CountDownLatch是不能够重用的，而CyclicBarrier是可以重用的。
2. semaphore其实和锁有点类似，它一般用于控制对某组资源的访问权限。

### Reference
1. [Java并发编程：CountDownLatch、CyclicBarrier和 Semaphore](http://www.importnew.com/21889.html)
