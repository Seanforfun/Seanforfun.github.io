---
layout: post
title:  "ThreadPool 线程池"
date:   2018-11-12 14:39:59
author: Botao Xiao
comment: true
categories: Concurrency
description: 在我最开始总结多线程的时候文章MultiThreadAndLock中曾经提到过线程池，但是当时对设计模式并没有很深刻的理解（Fly weight），并且对线程池的研究也没有很深入，此次我将会更加深入的总结线程池相关的应用。
---
在我最开始总结多线程的时候文章[MultiThreadAndLock](https://github.com/Seanforfun/Java-Knowledge/blob/master/Conclusions/MultiThreadAndLock.txt)中曾经提到过线程池，但是当时对设计模式并没有很深刻的理解（Fly weight），并且对线程池的研究也没有很深入，此次我将会更加深入的总结线程池相关的应用。

### ThreadPool的优势
1. 重用存在的线程，减少线程创建，消亡的时间。
2. 可以控制最大并发数，提高资源利用率，避免过多资源竞争，造成阻塞。
3. 提供定时执行，定期执行，单线程，并发控制。

### ThreadPoolExecutor类
线程池的类是ThreadPoolExecutor类
```Java
 /**
 * @param corePoolSize 线程池中长期激活的线程的个数。
 * @param maximumPoolSize 线程池所允许的线程的最大个数。
 * @param keepAliveTime 如果此时激活的线程并发数已经大于corePoolSize（小于等于maximumPoolSize），
 * 但是业务并发量下降（小于等于 corePoolSize），需要等待keepAliveTime后，将大于corePoolSize的线程终结，
 * 这有助于我们控制虚拟机中的线程栈空间。
 * @param unit keepAliveTime的单位
 * @param workQueue 阻塞任务队列，线程池将从其中取出任务并执行。
 * @param threadFactory 线程的工厂。此处使用了工厂模式，有助于对线程对象的统一装配。
 * @param handler 当线程使用到达上限或是其他原因导致业务被阻塞时的回调函数，此处实际上是用了策略模式。
 */
public ThreadPoolExecutor(int corePoolSize,
                          int maximumPoolSize,
                          long keepAliveTime,
                          TimeUnit unit,
                          BlockingQueue<Runnable> workQueue,
                          ThreadFactory threadFactory,
                          RejectedExecutionHandler handler)；
```

### 线程池的状态
![Imgur](https://i.imgur.com/V2FjQZ3.png)
1. Running: 线程池已经被创建，无论是否有业务在运行，都是Running状态。
2. Shutdown: Running状态的线程池接收到了shutdown()信号以后则会进入Shutdown状态。在Shutdown状态的线程池仍然会执行队列中的任务以及正在执行的任务。但是不会再接收新的任务。
3. Stop: Running状态的对象接收到了shutdownNow()信号以后，将会直接清空等待中的业务，中断当前正在执行的线程。
4. Tidying: 此时等待队列中业务数量为0，正在执行的所有业务已经清空（可能是由被中断或是已经执行完成），将会调用terminated()策略。在线程池的类是ThreadPoolExecutor类中，该方法是一个空方法，我们可以选择继承ThreadPoolExecutor类并重写该方法，实现自定义的清理。
5. Terminated: 线程被终止。

### 线程池的重要方法
```Java
// 向线程池提交任务，无返回值。
public void execute(Runnable command);
// 向线程池提交任务，有返回值，和FutureTask配合使用。
public <T> Future<T> submit(Callable<T> task)
// 将线程从Running态转为Shutdown态
void shutdown();
// 将线程从Running态转为Stop态。
List<Runnable> shutdownNow();
```

### 四种线程池模型
1. CachedThreadPool， 可缓存的线程池。会根据需要自动创建和回收线程池。
```Java
public static ExecutorService newCachedThreadPool() {
    return new ThreadPoolExecutor(0, Integer.MAX_VALUE,
                                  60L, TimeUnit.SECONDS,    //60s idle就会回收
                                  new SynchronousQueue<Runnable>());
                                  }
```

2. FixedThreadPool, 定长线程池，确定线程池的最大并发数。
```Java
 public static ExecutorService newFixedThreadPool(int nThreads) {
    return new ThreadPoolExecutor(nThreads, nThreads,   // coreSize和maxSize一致
                                  0L, TimeUnit.MILLISECONDS,
                                  new LinkedBlockingQueue<Runnable>());
}
```

3. ScheduledThreadPool, 定时、定期线程池，利用了blockingQueue和DelayQueue的特点
```Java
    public static ScheduledExecutorService newScheduledThreadPool(int corePoolSize) {
        return new ScheduledThreadPoolExecutor(corePoolSize);
    }
    public ScheduledThreadPoolExecutor(int corePoolSize) {
        super(corePoolSize, Integer.MAX_VALUE, 0, NANOSECONDS,
              new DelayedWorkQueue());  //此处的阻塞队列是DelayedWorkQueue，提供了定时定期的的数据结构。
    }    
```

4. SingleThreadPool, 单任务线程池，保证了任务在单线程中实施，任务先入先出。
```Java
    public static ExecutorService newSingleThreadExecutor() {
        return new FinalizableDelegatedExecutorService
            (new ThreadPoolExecutor(1, 1,
                                    0L, TimeUnit.MILLISECONDS,
                                    new LinkedBlockingQueue<Runnable>()));
    }
```

### 四种线程池的用例
1. CachedThreadPool,在少量并发下每个线程的ID不相同。
```Java
@Slf4j
public class ThreadPoolExp {
    public static void main(String[] args) {
        ExecutorService executors = null;
        try {
            executors = Executors.newCachedThreadPool();
            for(int i = 0; i < 10; i++){
                final int count = i;
                executors.execute(() -> {
                    log.info("Doing task - {}", count);
                });
            }
        } finally {
            executors.shutdown();
        }
    }
}
13:44:57.060 [pool-1-thread-6] INFO com.example.demo.ThreadPoolExp - Doing task - 5, Thread id - 16
13:44:57.059 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
13:44:57.062 [pool-1-thread-5] INFO com.example.demo.ThreadPoolExp - Doing task - 4, Thread id - 15
13:44:57.059 [pool-1-thread-7] INFO com.example.demo.ThreadPoolExp - Doing task - 6, Thread id - 17
13:44:57.059 [pool-1-thread-3] INFO com.example.demo.ThreadPoolExp - Doing task - 2, Thread id - 13
13:44:57.063 [pool-1-thread-9] INFO com.example.demo.ThreadPoolExp - Doing task - 8, Thread id - 19
13:44:57.059 [pool-1-thread-2] INFO com.example.demo.ThreadPoolExp - Doing task - 1, Thread id - 12
13:44:57.063 [pool-1-thread-10] INFO com.example.demo.ThreadPoolExp - Doing task - 9, Thread id - 20
13:44:57.063 [pool-1-thread-4] INFO com.example.demo.ThreadPoolExp - Doing task - 3, Thread id - 14
13:44:57.063 [pool-1-thread-8] INFO com.example.demo.ThreadPoolExp - Doing task - 7, Thread id - 18
```

2. FixedThreadPool， 总共只有3条线程，所有的业务都由这三条线程在执行。
```Java
@Slf4j
public class ThreadPoolExp {
    public static void main(String[] args) {
        ExecutorService executors = null;
        try {
            // 线程池中只有三条线程
            executors = Executors.newFixedThreadPool(3);
            for(int i = 0; i < 10; i++){
                final int count = i;
                executors.execute(() -> {
                    log.info("Doing task - {}, Thread id - {}", count, Thread.currentThread().getId());
                });
            }
        } finally {
            executors.shutdown();
        }
    }
}
13:46:02.131 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
13:46:02.135 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 3, Thread id - 11
13:46:02.135 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 4, Thread id - 11
13:46:02.135 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 5, Thread id - 11
13:46:02.136 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 6, Thread id - 11
13:46:02.132 [pool-1-thread-3] INFO com.example.demo.ThreadPoolExp - Doing task - 2, Thread id - 13
13:46:02.136 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 7, Thread id - 11
13:46:02.136 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 9, Thread id - 11
13:46:02.136 [pool-1-thread-3] INFO com.example.demo.ThreadPoolExp - Doing task - 8, Thread id - 13
13:46:02.133 [pool-1-thread-2] INFO com.example.demo.ThreadPoolExp - Doing task - 1, Thread id - 12
```

3. ScheduledThreadPool， 可以定期/定时循环执行
```Java
@Slf4j
public class ThreadPoolExp {
    public static void main(String[] args) {
        ScheduledExecutorService executors = null;
        try {
            executors = Executors.newScheduledThreadPool(10);
            for(int i = 0; i < 1; i++){
                final int count = i;
                executors.scheduleAtFixedRate(() -> {
                    log.info("Doing task - {}, Thread id - {}", count, Thread.currentThread().getId());
                }, 0, 2, TimeUnit.SECONDS); //第一次延时0，每次执行间隔5秒
            }
        } finally {
//            executors.shutdown(); //因为是一个循环的业务，所以一般会选择时机回收而不是直接回收。
        }
    }
}
14:25:28.323 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
14:25:30.322 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
14:25:32.321 [pool-1-thread-2] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 13
14:25:34.321 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
```

4. SingleThreadExecutor单线程池,所有的业务按照顺序执行。FIFO。
```Java
@Slf4j
public class ThreadPoolExp {
    public static void main(String[] args) {
        ExecutorService executors = null;
        try {
            executors = Executors.newSingleThreadExecutor();
            for(int i = 0; i < 10; i++){
                final int count = i;
                executors.execute(() -> {
                    log.info("Doing task - {}, Thread id - {}", count, Thread.currentThread().getId());
                });
            }
        } finally {
            executors.shutdown();
        }
    }
}
14:28:25.605 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 0, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 1, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 2, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 3, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 4, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 5, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 6, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 7, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 8, Thread id - 11
14:28:25.612 [pool-1-thread-1] INFO com.example.demo.ThreadPoolExp - Doing task - 9, Thread id - 11
```

### 线程池的配置
1. CPU密集型任务： 需要通过代码端主动压榨CPU，线程书设置为NCPU + 1。
2. IO密集型任务： 设置成2 * NCPU。
3. 还是要根据自己的业务设置配置。

### 引用
1. [9 1 线程池 1](https://www.youtube.com/watch?v=lc1eRMxXGb4&list=PLfezYSPixwxv7Y5FcrWXSQqX9vBF8OLMA&index=24)
2. [Java多线程线程池（4）--线程池的五种状态](https://blog.csdn.net/l_kanglin/article/details/57411851)