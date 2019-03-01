---
layout: post
title:  "JVM | Java memory management 自动内存管理机制"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JVM
comment: true
description:
---
### 运行时数据区域
![JVM ram areas](https://i.imgur.com/5a3nvpz.jpg)

1. 程序计数器（Program counter register）
 	1. 类似于Stack pointer。用于控制代码的运行行号，控制分支，循环，跳转，异常处理。
	2. 存储虚拟机字节码指令的地址。
	3. 每一条线程均有自己的PC。线程间的PC互不影响，相互独立。由JVM控制。
	4. 当正在运行native方法时，PC值为空（Undifined）。

2. Java虚拟机栈（JVM Stack）
	1. 线程私有，生命周期和线程相同。可以理解为一种局部变量表。
	2. 在一个方法被执行的时候，会开辟一块栈空间（Stack Frame）,用于存储局部变量，操作栈，动态链接，方法出口信息。
	3. 开辟栈空间后会将局部变量等信息压栈。
	4. 除了double和long会占用2个slots（4 bytes）。别的占用1slot（32位）。例如对象的引用，存储了对象的指针。returnAddress存储了字节码的地址。由此可见，JVM是32位机。
	5. 局部变量表所需的内存空间在编译期就完成了分配。

3. 本地方法栈（Native method stack）
	1. 作用和JVM Stack非常类似。
	2. 为虚拟机使用到的Native方法服务。

4. Java堆（Java Heap）
	1. 所有线程共享堆。
	2. 存放对象实例。所有的对象和数组均在对上分配空间。
	3. Garbage collection的重要区域。
	4. 物理上的不连续区域，但是在逻辑上是连续的区域。

5. 方法区（Method Area）
	1. 线程的共享区域。
	2. 存储类信息，常量，静态变量即时编译器编译后的代码。
	3. 在HotSpot的一些版本中，方法区的空间被定义为Permanent Generation（永久代）。

5.1. 运行时常量池(Runtime constant pool)
	1. 运行时常量池是方法区的一部分。
	2. Class文件中有一项信息是常量池（Constant Pool Table）,用于存放编译期生成的字面量和符号引用。这部分将在类加载后存放到方法区的运行时常量池中。

6. 直接内存（Direct Memory）
	1. 为了避免在Java堆和Native堆中来回复制数据而使用的堆外内存。
	2. 在NIO中被使用。例如我们使用ByteBuffer.allocate(128)。这种堆外内存的分配方法有区别于普通的Java堆，实现了零拷贝机制。

### 普通对象实例的创建
1. 加载类信息：当JVM检测到一个new语句，会去方法区(Method Area)中的常量池中定位到这个类的符号引用。如果这个符号引用没有被找到，JVM将会执行类的加载机制。
2. 空间分配： 在Java堆中开辟一段空间，此时该空间的大小已经完全确定。
3. 清空内存空间： 此处分配的内存空间上的数据是任意的，所以JVM会执行memset至0的过程，此时的基本数据类型已经被赋予了初始化值。
4. 执行init方法： 按照程序员的意愿初始化这个实例。

### 对象访问
> 对象的访问会涉及java栈，java堆，方法区三个区域。
```Java
Object obj = new Object();
```
1. `Object obj`会反应到Java栈的本地变量表中，作为一个reference类型数据出现。可以理解为在Java栈中存储了一个指针。
2. `new Object()`将会在Java堆中开辟一块空间，存储Object实例。
3. 在方法区中存储了对象的类型数据（对象类型，父类，实现的接口，方法等）。
	* 通过句柄存储
	![句柄访问对象](https://i.imgur.com/G4W4vqf.png)
	* 通过指针访问
	![指针访问对象](https://i.imgur.com/dpg3RYy.png)

### Java内存溢出
* 通过Eclispe Menmory Analyzer，可以分析出内存泄露的位置
* 通过减少内存的手段来判断何处造成内存溢出。
```
-Dfile.encoding=UTF-8    
-Xms20m -Xmx20m ##设置堆大小20m，并将最小和最大值设置相等，避免扩展
-XX:+HeapDumpOnOutOfMemoryError ##dump出当前的内存堆转储快照
-XX:HeapDumpPath=E:\job   ##指定路径(转储文件还是挺大的)
-XX:SurvivorRatio=8    ## 存活比2:8
```

![Eclispe Menmory Analyzer](https://i.imgur.com/DfWsc7G.png)

### 通过Unsafe分配内存(Very Bad!!!)
```Java
	public static void main(String[] args) throws Exception{
		Field f = sun.misc.Unsafe.class.getDeclaredField("theUnsafe");
		f.setAccessible(true);
		sun.misc.Unsafe unsafe = (sun.misc.Unsafe) f.get(null);
		long allocateMemory = unsafe.allocateMemory(_1MB);
		System.out.println(Long.toHexString(allocateMemory));
		unsafe.freeMemory(allocateMemory);
	}
```
>类似于C语言中的malloc和free。

### 垃圾收集器与内存分配策略
* 垃圾收集的三个问题
1. 什么内存需要回收
2. 什么时候回收
3. 如何回收

* 程序计数器，Java栈和本地方法区是线程私有的，和线程的生命周期一致。
	* 这三块区域在编译完成后的大小基本已经确定。
* Java堆和方法区
	* 对象实例的大小是无法确定的。
	* 方法区中的循环，分支造成的方法区所占的大小是无法确定的。
	* 垃圾收集器关注的就是这两块区域。

### 如何确定对象的区域是否仍然存活
#### 引用计数算法（JVM使用的并不是该方法）
1. 给对象添加一个引用计数器，当被引用时就加1，引用失效时就减1。
2. 缺陷：两个未失效的对象相互引用，都未失效但是变成了野指针。
```Java
	ObjA.instance = ObjB;
	ObjB.instance = ObjA;
```
由于两个对象相互引用，rc均不为0，但实际上都是游离的对象。

3. [Linux使用jstat命令查看jvm的GC情况](https://blog.csdn.net/zlzlei/article/details/46471627)

#### 根搜索算法（JVM使用的方法）
>通过一系列“GC ROOT”对象作为起始点，从这些结点开始向下搜索，搜索所走过的路径被称为引用链（Reference Chain）,当一个对象到所有GC ROOTS均不可达，说明是游离对象，可以被清理。

* 可以作为GC Root的对象：
1. JVM栈中的引用对象。（线程没有消亡时，是一个reference）
2. 方法区中的类静态属性引用的对象。（静态变量的reference，引用存储在方法区）
	`public static Integer i = new Integer(1);`
3. 方法区中的常量引用的对象。
	`private final Double PI = 3.1415D;`
4. 本地方法栈中引用对象。（Native 方法）

### 获取GC日志
```Java
    -Xms20m --jvm堆的最小值
    -Xmx20m --jvm堆的最大值
    -XX:+PrintGCTimeStamps -- 打印出GC的时间信息
    -XX:+PrintGCDetails  --打印出GC的详细信息
    -verbose:gc --开启gc日志
    -Xloggc:d:/gc.log -- gc日志的存放位置
    -Xmn10M -- 新生代内存区域的大小
    -XX:SurvivorRatio=8 --新生代内存区域中Eden和Survivor的比例
```

#### 引用
* 强引用（Strong Reference）
	* "Object obj = new Object()",只要强引用存在，内存不会被回收。
	* 强引用有引用变量指向时永远不会被垃圾回收，JVM宁愿抛出OutOfMemory错误也不会回收这种对象。
	* 如果想中断强引用和某个对象之间的关联，可以显示地将引用赋值为null，这样一来的话，JVM在合适的时间就会回收该对象。
* 软引用（Soft Reference）
	* 软引用说明某些引用仍然有用，但并非必要对象。在发生内存溢出前，会将这些引用列入回收范围进行二次回收。使用SoftReference实现。
	* 此处的软引用，因为一只调用了gc所以不会造成爆堆。
* 弱引用（Weak Reference）
	* 弱引用的等级比软引用还要低，对象生存时间只会存在到下一次垃圾回收。可以通过WeakReference实现，生存周期较软引用较短。
* 虚引用（Phantom Reference）
	* 虚引用是无法获取实例对象的。设置虚引用的唯一原因是希望这个对象在被回收时收到一个通知。通过Phantom Reference生成实例对象。
```Java
public class GCCollection {
	private static Byte[] bytes = new Byte[1024 *1024];
	public static void main(String[] args) throws InterruptedException {
		List<SoftReference<Byte[]>> l = new LinkedList<SoftReference<Byte[]>>();
		while(true){
			l.add(new SoftReference<Byte[]>(bytes));
			System.gc();
		}
	}
}
```
	* 换成强引用，即使不断调用gc仍然会爆堆
```Java
public class GCCollection {
	private static Byte[] bytes = new Byte[1024 *1024];
	public static void main(String[] args) throws InterruptedException {
		List<Byte[]> l = new LinkedList<Byte[]>();
		while(true){
			l.add(new Byte[1024 *1024]);
			System.gc();
		}
	}
}
Exception in thread "main" java.lang.OutOfMemoryError: Java heap space
	at ca.mcmaster.jvm.GCCollection.main(GCCollection.java:17)
```
	* 换做弱引用，调用System.gc()会回收掉弱引用的对象
```Java
public class GCCollection {
	private static Byte[] bytes = new Byte[1024 *1024];	//1Mb
	@SuppressWarnings("static-access")
	public static void main(String[] args) throws InterruptedException {
		List<WeakReference<Byte[]>> l = new LinkedList<WeakReference<Byte[]>>();
		int i = 100;
		while(i-- > 0){
			l.add(new WeakReference<>(bytes));
			System.gc();
			Thread.currentThread().sleep(1000);
		}
	}
}
```

	* 从垃圾回收日志中可以获取一些信息,可以看到回收了10240K，即为1MB。
```
0.121: [GC (System.gc()) [PSYoungGen: 5243K->648K(9216K)] 5243K->4752K(19456K), 0.0054301 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]
0.127: [Full GC (System.gc()) [PSYoungGen: 648K->0K(9216K)] [ParOldGen: 4104K->4631K(10240K)] 4752K->4631K(19456K), [Metaspace: 2737K->2737K(1056768K)], 0.0065591 secs] [Times: user=0.05 sys=0.00, real=0.01 secs]
```

### 什么样的空间要被宣布死亡
* 要被垃圾回收的内存要被标记两次
	* 根搜索后发现对象和GC Roots没有引用链。此时他们会被标记并进行一次筛选。
	* 第二次标记，如果经过了上一次的标记，将会把对象放在F-Queue中，并在稍后由一条由虚拟机自动建立，低优先级的Finalizer线程去执行。在执行对象回收的finalize方法（如果存在），如果没有重新构成引用链，就会被垃圾收集。

```Java
public class GCCollection {
	private Byte[] bytes = new Byte[1024 *1024];	//1Mb
	@SuppressWarnings("static-access")
	private static GCCollection HOOK = null;
	public static void main(String[] args) throws InterruptedException {
		HOOK = new GCCollection();
		HOOK = null;	//Detach the instance, call gc to collect the detached instance.
		System.gc();
		Thread.sleep(5000);
		System.out.println(HOOK);	//Find the instance is still alive.
	}
	@Override
	protected void finalize() throws Throwable {
		// TODO Auto-generated method stub
		System.out.println("enter finalize process...");
		super.finalize();
		HOOK = this;	//enter the finalize step and create a reference chain.
		System.out.println("I am still alive...");
	}
}
enter finalize process...
I am still alive...
ca.mcmaster.jvm.GCCollection@15db9742
```

### Garbage collection algorithm
#### Mark-sweep 标记-清除
1. Mark the instance that can be sweeped.
2. Clear garbage instance in F-Queue.
* Disadvantage
	1. Low efficiency.
	2. Cause a lot of unconnected memory.

#### Copying 复制算法
1. Copying用于回收新生代，新生代的对象朝生夕死，将新生代分配成两块Eden,Survivor(From, to)。
2. 将仍然存活的instance从Eden复制到Survivor上，然后将Eden全部清空。

#### Mark-Compact 标记-整理算法
1. 和Mark-sweep类似，但是不是直接清除，而是将存活的实例复制到一端，直接清理掉边界以外的内存。

#### Generation Collection 分代处理算法
![Heap Generation](https://i.imgur.com/EEGQTEC.jpg)
1. 根据对象的存活周期的不同将内存划分成不同的区域，根据每个区域采取适当的垃圾收集算法。
2. 新生代中使用复制算法。
3. 老年代中使用标记-清除和标记-整理算法。

### GC 垃圾收集器
>收集算法是内存回收的方法论，垃圾收集器就是内存回收的具体实现。

#### Serial收集器
1. 这是一个单线程收集器。
2. 在垃圾收集的时候，必须暂停其他所有的工作线程，直到他收集结束。
![Serial收集流程](https://i.imgur.com/9Y6zepM.jpg)

#### ParNew收集器
1. 这是Serial收集器的多线程版本。
![ParNew收集器](https://i.imgur.com/XJkL8z1.jpg)

#### Parallel Scavenge收集器
1. 新生代收集器，使用标记复制算法，并行的多线程收集器。
2. 达到可控制的吞吐量(throughput)。
3. 高吞吐量适用于后台大量运算，少用户交互的服务。
> throughput = code_runtime / (code_runtime + garbage_collection_time)

 -XX:MaxGCPauseMillis：控制最大垃圾收集停顿时间，大于零的毫秒数。
 -XX:GCTimeRatio：吞吐量大小，0到100的整数，垃圾收集时间占总时间的比例，计算1/（1+n）gc时间占用比例。
 -XX:UseAdaptiveSizePolicy：打开之后，就不需要设置新生代大小（-Xmn），Edian，survivor比例及（-XX:SurvivorRatio）晋升老年代年龄（-XX:PretenureSizeThreshold），虚拟机根据系统运行状况，调整停顿时间，吞吐量，GC自适应调节策略，区别parnew。

#### Serial Old收集器
1. 收集器的老年版本，他是一个单线程收集器，使用标记-整理方法。
2. 具体流程见图3-6。

#### Parallel Old收集器
1. Parallel Old收集器是Parallel Scavenge的老年代版本。多使用标记-整理算法。
![Parallel Old收集器](https://i.imgur.com/uzmnetq.jpg)

#### CMS收集器（Concurrent Mark Sweep）
1. 以获取最短停顿时间为目标。重视响应速度。
2. 分成四个步骤：
	* 初始标记 （CMS Initial Mark）
	* 并发标记 (CMS Concurrent Mark)
	* 重新标记（CMS remark）
	* 并发清除（CMS Concurrent Sweep）
![CMS收集器](https://i.imgur.com/09iVQPI.jpg)
> 其中初始标记和重新标记会Stop the world.

3. CMS默认的启动的回收线程数：（CPU# + 3）/4。其中，当CPU核数少于4时，CMS的吞吐量就会下降。
4. 无法处理浮动垃圾（Floating Garbage）,即在标记阶段未产生，此类垃圾必须等到下一次清理时被标记再次清理。
5. 默认的值是老年代的内存占满了68%后就会开始清理，实际中可以通过指令调高比例，但是有可能造成Concurrent Mode Failure。-XX:CMSInitiatingOccupancyFraction
6. 因为是基于标记-清理的，所以会产生不连续的内存，会为分配大内存对象造成影响。不得不提前触发一次Full GC。
![解决内存碎片问题](https://i.imgur.com/gVbQVla.jpg)

#### G1 收集器
1. 基于标记-整理算法。使用了区域划分和优先级回收。是最先进的垃圾收集算法。

### 收集器的参数设置
![收集器的参数设置](https://i.imgur.com/EfKLWdj.jpg)

### Memory allocation（内存分配）
#### 对象的实例在堆上分配。
1. 对象优先在新生代Eden区上分配。当Eden区没有足够的空间会发生一次Minor GC。
	* Minor GC:新生代GC，发生在新生代，很频繁，速度也快。
	* Major GC/Full GC:老年代GC，发生在老年代，经常会伴随新生代GC。
```Java
/**
 * @author SeanForFun E-mail:xiaob6@mcmaster.ca
 * @date Jun 6, 2018 5:21:16 PM
 * @version 1.0
 * -Xms20m
 * -Xmx20m
 * -Xmn10M
 * -XX:+PrintGCTimeStamps
 * -XX:+PrintGCDetails
 * -verbose:gc
 * -Xloggc:f:/dump/dc.log
 * -XX:SurvivorRatio=8
 * -XX:PretenureSizeThreshold=3145728
 */
public class GCCollection {
	private final static int _1MB = 1024 * 1024;
//	private Byte[] bytes = new Byte[1024 *1024];	//1Mb
	public static void main(String[] args) throws InterruptedException {
		byte[] a1 = new byte[2 * _1MB];
		byte[] a2 = new byte[2 * _1MB];
		byte[] a3 = new byte[2 * _1MB];
		byte[] a5 = new byte[2 * _1MB];
		byte[] a6 = new byte[4 * _1MB];
		byte[] a7 = new byte[4 * _1MB];
	}
}
/*GClog
0.124: [GC (Allocation Failure) [PSYoungGen: 7291K->664K(9216K)] 7291K->6816K(19456K), 0.0043880 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
0.128: [Full GC (Ergonomics) [PSYoungGen: 664K->0K(9216K)] [ParOldGen: 6152K->6677K(10240K)] 6816K->6677K(19456K), [Metaspace: 2699K->2699K(1056768K)], 0.0049759 secs] [Times: user=0.00 sys=0.00, real=0.00 secs]
0.134: [Full GC (Ergonomics) [PSYoungGen: 6302K->4096K(9216K)] [ParOldGen: 6677K->8724K(10240K)] 12979K->12820K(19456K), [Metaspace: 2699K->2699K(1056768K)], 0.0048303 secs] [Times: user=0.06 sys=0.00, real=0.00 secs]
0.139: [Full GC (Allocation Failure) [PSYoungGen: 4096K->4096K(9216K)] [ParOldGen: 8724K->8712K(10240K)] 12820K->12808K(19456K), [Metaspace: 2699K->2699K(1056768K)], 0.0060969 secs] [Times: user=0.00 sys=0.00, real=0.01 secs]*/
```

2. 大对象直接在直接进入老年代
	* 所谓的大对象就是需要占用大量连续内存的对象。
	* 作为程序员，我们要避免生成生存周期很短的大对象，这要对GC的要求很高。

3. 长期存活的对象将进入老年代
	* 虚拟机会分配给每一个对象一个年龄计数器。在新生代中，每在Survive区存货一个就会增加1，当他的年龄到达15就会被晋升到老年代。-XX:InitialTenuringThreshold=1

### 使用Java VisualVM来对JVM进行监控
![Java Visual](https://i.imgur.com/x7mWu0K.jpg)
>监视CPU活动状况和堆的使用状况。

### 在linux中使用jstat -l pid来获得进程堆栈的快照。
![snapshot](https://i.imgur.com/6fXLhB9.jpg)

### Reference
1. [深入理解JVM](https://book.douban.com/subject/6522893/)
