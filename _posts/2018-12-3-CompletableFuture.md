---
layout: post
title:  "Java8| Asynchronized programming"
date:   2018-12-3 12:32
author: Botao Xiao
categories: JavaCore
description: CompletableFuture同时实现了Future接口和CompletionStage接口。Future接口保证了阻塞式的异步接口，而CompletionStage实现了异步流式接口，让我们能通过流式接口简单快捷的实现异步。
---
从Java5开始，Java便引入了Future接口，这是一种常见的异步式的编程方式。我们将一个任务提交给了异步线程并且可以立刻返回做自己的事情，过了一段时间，我们将会向异步线程对象请求结果。

### Future
![Imgur](https://i.imgur.com/A4soOjS.png)
Future是Java5中新增的接口，用于异步式编程。获得一个Task的句柄，可以在未来的某个时刻返回结果。
```Java
public interface Future<V> {
    // 取消当前任务，当前已经拥有了Future句柄，我们可以通过任务的句柄取消当前任务。
    boolean cancel(boolean mayInterruptIfRunning);
    // 返回当前任务是否已经被取消。
    boolean isCancelled();
    // 当前认识是否已经被完成，如果完成则可以通过get获得结果。
    boolean isDone();
    // 从Future句柄中获得结果，如果当前计算没有结束，则会一直阻塞。
    V get() throws InterruptedException, ExecutionException;
    // 从Future中获得结果，和get不同的是，最多等待timeout的时间，则会进行get操作。
    V get(long timeout, TimeUnit unit)
        throws InterruptedException, ExecutionException, TimeoutException;
}
```

#### Future的实现方式
1. 通过get()方法，实现异步，但是由于一致阻塞在get()方法处，和阻塞式的没什么区别。
```Java
public class myFuture {
    public static void main(String[] args) {
        ExecutorService executors = Executors.newCachedThreadPool();
        Future<String> future = executors.submit(() -> {
            System.out.println("Start running task.");
            Thread.sleep(5000);
            return "Get result!";
        });
        System.out.println("Do something else.");
        try {
            System.out.println(future.get());   // 当结果没有返回时，会一致阻塞在这儿。
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

2. 通过轮询只有当结果出现时才会获得结果。
```Java
public class myFuture {
    public static void main(String[] args) {
        ExecutorService executors = Executors.newCachedThreadPool();
        Future<String> future = executors.submit(() -> {
            System.out.println("Start running task.");
            Thread.sleep(5000);
            return "Get result!";
        });
        System.out.println("Do something else.");
        try {
            while(!future.isDone()){
                System.out.println("Check once.");
                Thread.sleep(1000);
            }
            System.out.println(future.get());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } catch (ExecutionException e) {
            e.printStackTrace();
        }
    }
}
```

3. 缺点：
    * 没有通知机制，只能通过轮询，会造成大量的CPU浪费。
    * 如果通过get()直接获得结果，则会发生阻塞，此时和非异步式无异。

### Java8 CompletableFuture 
CompletableFuture同时实现了Future接口和CompletionStage接口。Future接口保证了阻塞式的异步接口，而CompletionStage实现了异步流式接口，让我们能通过流式接口简单快捷的实现异步。

#### 创建CompletableFuture
```Java
// 创建一个已经完成的CompletableFuture对象，该对象已经完成了运算。返回值会封装形参中的value。
public static <U> CompletableFuture<U> completedFuture(U value) {
    return new CompletableFuture<U>((value == null) ? NIL : value);
}
// 使用默认的线程池获取一个U对象。
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier) {
    return asyncSupplyStage(asyncPool, supplier);
}
// 使用提供的线程池获取一个对象。
public static <U> CompletableFuture<U> supplyAsync(Supplier<U> supplier,
                                                   Executor executor) {
    return asyncSupplyStage(screenExecutor(executor), supplier);
}
// 使用默认的线程池异步执行Runnable方法。
public static CompletableFuture<Void> runAsync(Runnable runnable) {
    return asyncRunStage(asyncPool, runnable);
}
// 使用自定义的线程池异步执行Runnable方法。
public static CompletableFuture<Void> runAsync(Runnable runnable,
                                               Executor executor) {
    return asyncRunStage(screenExecutor(executor), runnable);
}
```

#### thenApply: 不停的输入输出的进行生产
```Java
// 当前阶段正常完成以后执行，而且当前阶段的执行的结果会作为下一阶段的输入参数。
public <U> CompletableFuture<U> thenApply(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(null, fn);
}
// 使用默认或自定义的线程池进行异步执行。thenApplyAsync默认是异步执行的。这里所谓的异步指的是不在当前线程内执行。
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn) {
    return uniApplyStage(asyncPool, fn);
}
public <U> CompletableFuture<U> thenApplyAsync(
    Function<? super T,? extends U> fn, Executor executor) {
    return uniApplyStage(screenExecutor(executor), fn);
}
```

#### thenRun, thenAccept: 不断消耗流中的资源
```Java
// 非异步执行一个行为。
public CompletableFuture<Void> thenRun(Runnable action) {
    return uniRunStage(null, action);
}
public CompletableFuture<Void> thenAccept(Consumer<? super T> action) {
    return uniAcceptStage(null, action);
}
// 在默认的线程池中异步执行一个行为。
public CompletableFuture<Void> thenRunAsync(Runnable action) {
    return uniRunStage(asyncPool, action);
}
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action) {
    return uniAcceptStage(asyncPool, action);
}
// 在自定义的线程池中异步执行一个行为。
public CompletableFuture<Void> thenRunAsync(Runnable action,
                                            Executor executor) {
    return uniRunStage(screenExecutor(executor), action);
}
public CompletableFuture<Void> thenAcceptAsync(Consumer<? super T> action,
                                               Executor executor) {
    return uniAcceptStage(screenExecutor(executor), action);
}
```

#### thenCombine：合并两个流的结果
```Java
// 此处传入一个CompletionStage的子类（大多是时候传入的是CompletableFuture）。
// 此时BiFunction的两个输入参数分别是当前的CompletableFuture和上一个参数传入的CompletableFuture。
public <U,V> CompletableFuture<V> thenCombine(
    CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn) {
    return biApplyStage(null, other, fn);
}
// 和上述的描述类似，唯独不同的是通过异步执行的。
public <U,V> CompletableFuture<V> thenCombineAsync(
    CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn) {
    return biApplyStage(asyncPool, other, fn);
}

public <U,V> CompletableFuture<V> thenCombineAsync(
    CompletionStage<? extends U> other,
    BiFunction<? super T,? super U,? extends V> fn, Executor executor) {
    return biApplyStage(screenExecutor(executor), other, fn);
}
```

#### whenComplete: 异步完成时的回调函数
```Java
// 所有业务完成后的回调函数，非异步。
public CompletableFuture<T> whenComplete(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(null, action);
}
// 异步的整体完成后的回调函数。
public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action) {
    return uniWhenCompleteStage(asyncPool, action);
}
public CompletableFuture<T> whenCompleteAsync(
    BiConsumer<? super T, ? super Throwable> action, Executor executor) {
    return uniWhenCompleteStage(screenExecutor(executor), action);
}
```

### 引用
1. [CompletableFuture基本用法](https://www.cnblogs.com/cjsblog/p/9267163.html)
















