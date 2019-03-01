---
layout: post
title:  "Concurrency | Concurrent on JMM"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JVM Concurrency
comment: true
description: JVM层面的并发解析。
---

### Java Memory Model and Thread
> TPS(Transaction per second)是衡量服务性能好坏的一个重要指标。代表着一秒内服务端平均能响应的请求总数。

#### 硬件效率一致性
![processer, cache and memory](https://i.imgur.com/jaNupIN.png)

#### Java内存模型(Java Memory Model)
1. JMM定义了程序中每个变量的访问规则。此处变量指的是实例字段，静态字段和构成数组对象的元素，这些变量都不是线程私有的。
![JMM](https://i.imgur.com/254ieV7.png)
	* Main Memory 主内存
		* 这是虚拟机内存的一部分。
		* 所有的变量都存储在主内存中。
	* Working Memory 工作内存
		* 保存所有当前线程需要用到的变量在主内存中的副本拷贝。
		* 线程对变量的所有操作都必须在工作内存中完成，不能直接读写主内存的变量。

#### 内存间的交互操作
* lock：作用于主内存的变量，它把一个变量标识为一条线成独占的状态。
* unlock：作用于主内存的变量，把一个出于锁定状态的变量释放出来，释放后的变量才能被别的线程锁定。
* read：作用于主内存的变量，将主内存中的变量传输到线程的工作内存中，以便随后load使用。
* load：作用于工作内存的变量，将read从主内存中得到的变量放入工作内存的变量副本中。
* use:作用于工作内存的变量，它把工作内存中的一个变量的值传递给执行操作，每当虚拟机需要用到变量的值的时候会执行这个操作。
* assign：作用于工作内存的变量。当JVM需要对某个变量进行赋值时，将调用该指令对工作内存中的变量进行赋值。
* store： 将工作内存的一个变量的值传到主内存，为了write做准备。
* write：作用于主内存的变量，将store得到的变量的值写入。

#### volatile
1. volatile不是线程安全的，能保证的是在进行读取的时候这个值是主内存中的最新值。但是同时别的线程可能也已经获取了值，在写回主内存中时候可能造成线程不安全。
```Java
public class VolatileTest{
	public static volatile int count = 0;
	public static void increase(){
		count ++;
	}
	public static void main(String[] args) throws InterruptedException {
		Thread[] threads = new Thread[20];
		for(int i = 0; i < 20; i++){
			threads[i] = new Thread(() -> {
				for(int j = 0; j < 10000; j++)
					increase();	//每个线程都自增一个volatile变量。
			}, "thread-" + i);
			threads[i].start();
		}
		for(Thread t : threads)
			t.join();
		System.out.println(count);	//最后得到结果一般都小于200000
	}
}
```
2. 写入时要保证修改变量后要立刻写入主内存中，保证其他变量都能看到线程对V的改变。
3. 保证volatile修饰的变量不会被指令重排序优化。

#### 原子性与可见性与有序性
1. 原子性
	* 虚拟机没有把lock和unlock开放给用户使用，却提供了monitorenter和monitorexit来隐式的使用lock和unlock。反映到Java的上层代码中就是syncronized,所以在synchronized之间的代码是有原子性的。
2. 可见性
	* 通过volatile变量可以实现。与普通变量相比，volatile变量可以保证当前变量是主内存中的最新值。
	* syncronized和final也可以保证可见性。
3. 有序性
	* 如果在线程内部观察，所有的线程都是有序的，如果在一个线程观察另一个线程，所有的操作都是无序的。

#### 先行发生原则（happens-before）
1. 程序次序原则(Program Order Rule):在一个线程内部，按照代码的顺序，书写在前面的操作先行发生于书写在后面的操作。
2. 管程锁定规则(Monitor Lock Rule):同一把锁，上一次unlock之前的代码在此次的lock之前。
3. volatile变量规则(Volatile Variable Rule):对于一个volatile变量，写操作优先于对这个变量的读操作。
4. 线程启动规则(Thread Start Rule):Thread对象的start方法优先于此线程的每一个操作。
5. 线程终止规则(Thread Termination Rule):线程所有的指令优先于线程的终止。
6. 线程中断规则(Thread Interrupt Rule):对线程interrupt方法的调用先行发生于被中断线程的代码检测到中段发生的事件。
7. 对象终结规则(Finalizer Rule):对象的初始化优先于对象对于Finalize方法的调用。
8. 传递性(Transitivity):A优先于B，B优先于C，所以A优先于C。
