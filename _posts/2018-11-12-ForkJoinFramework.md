---
layout: post
title:  "Concurrency | Fork/Join Framework"
date:   2018-11-12 12:09:59
author: Botao Xiao
comment: true
categories: Concurrency
description: Fork/Join框架是一种用于多线程并行的框架，实际上是利用了分治(divide and conquer)的思想,将大任务拆分成小任务进行运算。每次将任务分割成小任务，直到无法分割以后再进行运算，最后将所有的结果汇总。
---
Fork/Join框架是一种用于多线程并行的框架，实际上是利用了分治(divide and conquer)的思想,将大任务拆分成小任务进行运算。每次将任务分割成小任务，直到无法分割以后再进行运算，最后将所有的结果汇总。

### 工作窃取算法（work-stealing算法）
1. 我们将一个较为繁重的业务分割成多个小任务添加到不同的线程中。每个线程有一个对应的双端队列（deque）。
2. 线程分别从队列头取出任务执行。
3. 如果一个线程已经执行完了自己线程中的所有任务，将会从别的线程的队列中取出任务执行，使当前线程一直有任务可做。
![Imgur](https://i.imgur.com/mM6HYgI.png)

### Fork / Join框架的使用
```Java
@Slf4j
public class ForkJoinExp extends RecursiveTask<Integer> {
    private int start;
    private int end;
    private static final int THRESHOD = 2;
    public ForkJoinExp(int start, int end) {
        this.start = start;
        this.end = end;
    }
    @Override
    protected Integer compute() {
        int sum = 0;
        /**
         * 如果任务已经无法分割，就直接进行运算。
         * 实际上就是判断是否还可以divide。
         */
        if(end - start <= THRESHOD){
            log.info("thread - {}", Thread.currentThread().getId());
            for(int i = start; i <= end; i++)
                sum += i;
        }else{
            /**
             * 如果任务可以分割，则将任务进行分割，通过递归的方式向下传递。
             */
            int middle = start + (end - start) / 2;
            ForkJoinExp leftTask = new ForkJoinExp(start, middle);
            ForkJoinExp rightTask = new ForkJoinExp(middle + 1, end);
            /**
             * Fork出子任务并执行。
             */
            leftTask.fork();
            rightTask.fork();
            /**
             * 等待子任务返回，并将结果进行合并。
             */
            sum += leftTask.join() + rightTask.join();
        }
        return sum;
    }

    public static void main(String[] args) throws ExecutionException, InterruptedException {
        ForkJoinPool pool = null;
        try {
            /**
             * Fork/Join框架的代码一定要通过ForkJoinPool进行运算。
             */
            pool = new ForkJoinPool();
            log.info("Result {}", pool.submit(new ForkJoinExp(1, 100)).get());
        }finally {
            /**
             * 将pool资源释放。
             */
            pool.shutdown();
        }
    }
}
```

### Fork/ Join的局限性
1. 只能使用Fork/Join作为同步机制。
2. 不能再任务的过程中执行I/O操作。
3. 任务不能抛出检查异常。

### 引用
1. [聊聊并发（八）——Fork/Join框架介绍](聊聊并发（八）——Fork/Join框架介绍)
2. [8 3 J U C ForkJoin](https://www.youtube.com/watch?v=b4c4beZ2yes&list=PLfezYSPixwxv7Y5FcrWXSQqX9vBF8OLMA&index=22)