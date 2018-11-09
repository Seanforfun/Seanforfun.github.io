---
layout: post
title:  "ReentrantLock, Szychronized"
date:   2018-11-09 16:39:59
author: Botao Xiao
comment: true
categories: Concurrency
description: 在我的第一次接触到C的时候就已经开始了对锁的研究，并且在总结《MultiThreadAndLock》的时候也有所提及,本次已经时隔大半年，技术方面的理解也更加深刻，我想重新研究这个话题，尽量做到更深入，更全面。同时在我的另一篇文章《volatile, Szychronized关键字》也有提及,可以参考。
---
在我的第一次接触到C的时候就已经开始了对锁的研究，并且在总结[《MultiThreadAndLock》](https://github.com/Seanforfun/Java-Knowledge/blob/master/Conclusions/MultiThreadAndLock.txt)的时候也有所提及,本次已经时隔大半年，技术方面的理解也更加深刻，我想重新研究这个话题，尽量做到更深入，更全面。同时在我的另一篇文章[volatile, Szychronized关键字](https://seanforfun.github.io/javacore/concurrency/2018/11/07/VolatileSynchronized.html)也有提及,可以参考。

### ReentrantLock和szychronized的区别
1. szycrhonized是Java中的关键字，锁机制是通过JVM实现的。而ReentrantLock是JUC下的一个类，是通过代码实现的。两者的实现位置略有不同。
2. 在早期的版本中，synchronized的实现效率较为低一些，但是后期版本中为synchronized引入了自旋锁和偏向锁，使得synchronized和ReentrantLock的效率相差很小。
3. 功能上的区别
    * ReentrantLock可以实现公平锁和非公平锁，而szychronized只能实现非公平锁。
    * ReentrantLock必须要手动解锁，不然会造成死锁，而synchronized会有JVM帮助解锁。
    * ReentrantLock有Condition功能，可以实现分组解锁。
    * ReentrantLock实现了中断等待锁的机制。lock.lockInterruptibly().

### synchronized关键字
1. synchronized关键字是通过JVM实现的。
2. 通过加入monitorenter和monitorexit的汇编指令，保证线程安全。
3. 作用域：
    1. 对象方法, 获取当前对象实例的实例锁，对于同一个对象实例才有线程安全的作用。
    ```Java
     public synchronized  void add1(){
        count++;
    }
    ```
    
    2. 作用静态方法。 获取当前类的类锁，左右于所有调用这个方法的对象。锁粒度大一些。
    ```Java
    public synchronized static void add() {
        System.out.println(count++);
    }
    ```
    
    3. 作用在对象上，分为三种情况，作用于类，当前实例和某个对象实例。实际上都是将某个对象当做锁在使用1中是将实例锁当做锁(synchronized(this))，2是将类锁当做锁（synchronize(Test.class)），如果我们传入一个任意的实例，我们就是将该实例传入当做锁。
    ```Java
    /**
     * 实际上我们使用string作为一个锁，即使当前对象和锁毫无关系。
     */
    public synchronized static void add() {
        /**
         * 此处string已经作为锁在使用了。
         */
        synchronized (string){
            System.out.println(count++);
        }
    }
    ```

4. 可重入锁
在同一个代码块中，synchronized是可重入的。
```Java
public class TestSynchronized {
    private volatile int countA = 0;
    public int getCountA() {
        return countA;
    }
    private volatile int countB = 0;
    public synchronized void addA(){
        synchronized (this){
            countA ++;
            addB();
        }
    }
    public synchronized  void addB(){
        System.out.println("countA: " + countA + " countB: " + countB++);
    }
    public static void main(String[] args) {
        ExecutorService executors = Executors.newCachedThreadPool();
        TestSynchronized t = new TestSynchronized();
        CountDownLatch latch = new CountDownLatch(10000);
        for(int i = 0; i < 10000; i++){
            executors.execute(() -> {
                t.addA();
                latch.countDown();
            });
        }
        try {
            latch.await();
            System.out.println("A: " + t.getCountA());
        } catch (InterruptedException e) {
            e.printStackTrace();
        } finally {
            executors.shutdown();
        }
    }
}
```

### ReentrantLock
对于ReentrantLock的总结针对于其区别于synchronized的地方。
1. 设置公平锁
```Java
// 非公平锁，默认
private Lock countALock = new ReentrantLock(false);
// 公平锁
private Lock countALock = new ReentrantLock(true);
```

2. Condition
#### Condition的介绍
1. Condition是通过锁获取的，Condition依赖于Lock接口，生成一个Condition的基本代码是lock.newCondition()。
2. 调用Condition的await()和signal()方法，都必须在lock保护之内，就是说必须在lock.lock()和lock.unlock之间才可以使用。所以必须在获取到锁以后使用！
3. condition.await()
	* 首先在condition.await()必须在获取所以后使用。
	* 调用该方法会让线程阻塞，并且让锁可以被别的线程获取，但是最终仍然要释放锁资源！
	* 一般都要在while中使用，一般通过一个别的变量来让condition在while中循环生效等待，原因是为了协同，不让condition在await之前就被signal过了，这样的话当前所期望的await将会永久的阻塞。
4. condition.signal()
	* 必须要获取锁才能使用该方法，不然会出现java.lang.IllegalMonitorStateException。
	* 解除condition.await()造成的阻塞。

#### Condition的使用
```Java
public class LocksTest implements Runnable{
//	ReentrantLock fairLock = new ReentrantLock(true);
	private final ReentrantLock unfairLock;
	private final Condition lockCondition;
	private final Condition lockCondition1;
	@Override
	public void run() {
		unfairLock.lock();
		try {
			lockCondition.await();	//condition进入阻塞，此处编程并不好，应该让condition进入阻塞应该配合别的变量在while中使用。不然会造成signal在await之前调用。
			System.out.println("After await......");
		} catch (InterruptedException e) {
			e.printStackTrace();
		}finally{
			unfairLock.unlock();
		}
		unfairLock.lock();
		try {
			lockCondition1.await();	//condition1进入阻塞
			System.out.println("After await1......");
		} catch (InterruptedException e1) {
			// TODO Auto-generated catch block
			e1.printStackTrace();
		}finally{
			unfairLock.unlock();
		}
	}
	public LocksTest(ReentrantLock lock, Condition condition, Condition condition1){
		this.unfairLock = lock;
		this.lockCondition = condition;
		this.lockCondition1 = condition1;
	}
	public static void main(String[] args) {
		ReentrantLock lock = new ReentrantLock();
		Condition condition = lock.newCondition();
		Condition condition1 = lock.newCondition();
		Thread t = new Thread(new LocksTest(lock, condition, condition1));
		Thread t1 = new Thread(new ConditionReleaseTest(lock, condition, condition1));
		t.start();
		t1.start();
	}
}
```
```Java
public class ConditionReleaseTest implements Runnable {
	private final Condition condition;
	private final Condition condition1;
	private final ReentrantLock lock;
	public ConditionReleaseTest(ReentrantLock lock, Condition condition, Condition condition1){
		this.lock = lock;
		this.condition = condition;
		this.condition1 = condition1;
	}
	@Override
	public void run() {
		try {
			Thread.sleep(2000);
			lock.lock();
			condition.signalAll();	//condition解除阻塞
			lock.unlock();
			Thread.sleep(2000);
			lock.lock();
			condition1.signalAll();	//condition1解除阻塞
			lock.unlock();
		} catch (InterruptedException e) {
			e.printStackTrace();
		}
	}
}
```

### 引用
1. [深入理解Java并发之synchronized实现原理](https://blog.csdn.net/javazejian/article/details/72828483)
2. [Java并发之锁测试与超时理解（lock、lockInterruptibly、trylock）](https://blog.csdn.net/jisuanjiguoba/article/details/80095506)