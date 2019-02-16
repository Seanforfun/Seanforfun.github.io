---
layout: post
title:  "JVM ClassLoader"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JVM
comment: true
description: JVM层面的类加载器。
---
### Class类文件结构
1. Class文件是一组以8位字节为基础单位的二进制流。
2. 当遇到需要占用8字节以上空间的数据项时，会按照高位在前的方式分割若干个8位字节进行存储。
3. Class文件格式采用一种类似C语言结构体的伪结构储存：
	* 无符号数
	* 表是由多个无符号数或其他表作为数据项构成的复合数据类型。

![Class文件格式](https://i.imgur.com/UegLrwk.jpg)

#### 魔数与Class文件的版本
1. 每个Class文件的头4个字节叫做魔数（Magic Number），确定这个文件能否被JVM接受。
![Magic Number](https://i.imgur.com/NEBFnqp.png)

2. 第5,6字节表示词版本号，7,8代表主版本号。
![Version](https://i.imgur.com/LjDv7Vz.png)

3. 常量池
	* 常量池的长度是不固定的，常量池的开始有u2来代表常量池容量计数值。
		* 字面量（Literal）,文本字符串，被声明为final的常量值。
		* 符号引用（Symbolic Reference）
			* 类的接口和全限定名。（Fully qualified name）
			* 字段的名称和描述符。（Field Descriptor）
			* 方法的名称和描述符。（Method Descriptor）

![常量池的项目类型](https://i.imgur.com/lK6WaPK.jpg)

#### 访问标志
1. 访问标识用于识别一些类或接口的层次的访问信息。
![访问标志](https://i.imgur.com/iuz6sPi.jpg)

#### 类引索（this_class），父类引索(super_class)，接口引索(interfaces)
1. 类引索用于确定这个类的全限定名。
2. 父类引索用于确定这个类的父类的全限定名。
3. 接口引索用来描述这个类实现了哪些接口。

#### 字段表集合
1. 字段表用于描述接口或类中声明的变量。（Fields）
	* 字段的作用域。（public, private, protected, default）
	* 是否是静态变量。（static）
	* 可变性（final）
	* 并发可见性（volatile，是否强制从内存读写）
	* 可否序列化（transient）
	* 字段数据类型（基本类型，对象，数组）
	* 字段名称

#### 方法表集合
1. 方法表用于描述类中的方法（Methods）
	* 访问标志（Access flag）
	* 名称引索（name index）
	* 描述符引索（descriptor index）
	* 属性集合表（attributes）

#### 属性集合表
1. 在Class文件，字段表，方法表都可以携带自己的属性表集合。
![虚拟机规范](https://i.imgur.com/VN8VAzw.jpg)
* Code属性
	* Javac编译器处理Java程序，最终变成字节码指令存在Code属性中。
	* 只有方法的实体才会具有Code属性，而接口和抽象类中的一些方法就没有Code属性。
* Exception属性
	* Exception属性列出方法的可能的受查异常。
* LineNumberTable属性
	* 它用于描述Java源码行号与字节码行号之间的对应关系。
* LocalVariableTable属性
	* 它用于描述栈帧中局部变量表中的变量与Java源码中定义的变量之间的对应关系。
* SourceFile属性
	* 它用于记录生成这个Class文件的源码文件名称。
* ConstantValue属性
	* ConstantValue属性的作用是通知虚拟机自动为静态变量赋值，只有被static修饰的变量才可以使用这项属性。
* InnerClasses属性
	*  该属性用于记录内部类与宿主类之间的关联。如果一个类中定义了内部类，那么编译器将会为它及它所包含的内部类生成InnerClasses属性。
* Deprecated属性
	* 该属性用于表示某个类、字段和方法，已经被程序作者定为不再推荐使用，它可以通过在代码中使用@Deprecated注释进行设置。
* Synthetic属性
	* 该属性代表此字段或方法并不是Java源代码直接生成的，而是由编译器自行添加的，如this字段和实例构造器、类构造器等。

### ClassLoader类加载器
> 虚拟机把描述类的class文件加载到内存，并对数据进行校验，转换解析和初始化，最终形成JVM可以直接使用的Java类型。这就是虚拟机的类加载机制。
> 在Java中，类型的加载和连接过程都是在运行期完成的。例如，如果编写一个使用接口的一个应用程序，会在运行时指定实现方式，这就是泛型的JVM的实现。

#### 类加载的时机
> 类的生命周期是指从二进制流加载到内存中开始到卸载出内存。类的生命周期被分成七个步骤
* 加载（Loading）
* 验证（Verification）
* 准备（Preparation）
* 解析（Resolution）
* 初始化（Initialization）
* 使用（Using）
* 卸载(Unloading)
![Life cycle of class](https://i.imgur.com/JihNPSD.jpg)

#### 初始化的时机
* 初始化的时机是很确定的。
	* 在遇到new, getstatic, putstatic, invokestatic时，如果类没有进行过初始化，则先需要初始化。
	* 在使用反射时，如果没有初始化，则进行类初始化让虚拟机可以反射调用类。
	* 在使用一个子类时，如果它的父类没有进行初始化，则将它的父类进行初始化。这样父类中的静态方法，静态变量才能被使用。
	* 当虚拟机启动时，会初始化main方法所在的类。

### 类加载的过程
> 类加载的过程分为五步，加载，验证，准备，解析，初始化。

* 加载（Loading）
> 加载不同于类加载，是类加载最开始的一个步骤。
1. 通过一个类的全限定名来获取定义此类的二进制流。例如ca.mcmaster.chapter.four.graph.spt.DirectedEdge，这个全限定名是唯一的。
2. 将这个字节流所代表的静态储存结构转化为方法区的运行时数据结构。
3. 在java堆中，生成一个代表这个类的java.lang.Class对象，作为方法区这些数据的入口。

* 验证（Verifying）
> 验证是连接阶段的第一步，用于确保Class文件的字节流中包含的信息符合当前虚拟机的要求，并确保JVM的安全。
> 如果验证到Class文件转化的字节流不符合Class文件的储存格式，将会抛出java.lang.Verify异常。

1. 文件格式验证，用于保证输入的字节流可以正确的被解析。
2. 元数据校验，保证不存在不符合java语言规范的元数据信息。
3. 字节码验证，主要工作是进行数据流和控制流分析。保证被校验类的方法在运行时不会做出危害JVM的行为。例如跳到方法体以外的指令，在调用int时使用long型变量。
4. 符号引用验证，如果没有通过，会抛出NoSuchMethodError, NoSuchFieldError等异常。

* 准备（Preparation）
> 准备阶段是正式为类分配内存，并设置类变量的初始值。

1. `public static int value = 123`
此时在方法区中为类开辟内存，并实行memset到0的操作。此时虽然value被赋值123，但是当前步骤不会根据代码对静态变量进行赋值。

![基本数据类型的初始化值](https://i.imgur.com/xhJBnpt.jpg)

* 解析（Resolving）
> JVM将常量池中的符号引用替换为直接引用的过程。符号引用是指CONSTANT_CLASS_info，CONSTANT_Fieldref_info等等。直接引用是指向目标的指针，相对偏移量或是一个可以直接定义到目标的句柄。

1. 类或接口的解析
2. 字段解析，从当前类开始检查接口中是否有对应的字段名。如果在接口中没有找到对应的字段，则向上递归查找相匹配的字段，如果最终仍未找到，则抛出NoSuchFieldError异常。找到后，再检查是否有访问权限。
3. 类方法解析，和字段解析相似，先检查当前类中是否有匹配的方法，再在父类中查找是否有匹配的方法，最后检查接口中是否有匹配的类，如果在接口中找到，说明当前类是一个抽象类，抛出AbstractMethodError。如果在接口中也没有找到，在完成查找，抛出NoSuchMethodError。最后检查访问权限。
4. 接口方法解析，从当前类开始，往接口中递归查找。不存在访问权限的问题，因为接口中的所有方法都是public的。

* 初始化
> 在初始化阶段，根据程序指定的主观计划去初始化类变量和其他资源。从另一个角度进行表达：初始化阶段是执行类构造器<clinit>()方法的过程。

	* <clinit>()方法
		* <clinit>()方法是由编译器自动收集类中的所有静态变量的赋值动作和静态语句块（static{}）产生的。
		* JVM会保证父类的<clinit>()方法优先于子类的<clinit>()执行。

### ClassLoader类加载器
> 虚拟机设计团队把类加载阶段中“通过一个类的全限定名来描述此类的二进制字节流”这个动作放到JVM外部去实现，以便让程序自己决定如何去获取所需的类。实现这个动作的代码模块被称为“类加载器”。

1. 对于任意一个类，都需要由加载它的类加载器和这个类本身一同确立其在JVM中的唯一性。
```Java
public class ClassLoaderTest extends ClassLoader{
	@Override
	public Class<?> loadClass(String name) throws ClassNotFoundException {
		String filename = name.substring(name.lastIndexOf('.') + 1) + ".class";
		InputStream is = getClass().getResourceAsStream(filename);	//获取类名，并获取其二进制文本
		if(is == null)
			return super.loadClass(name);
		try {
			byte[] b = new byte[is.available()];
			is.read(b);	//Read the class binary file into the byte array.
			return defineClass(name, b, 0, b.length);	//Use the binary saved in the byte array to create a new class.
		} catch (IOException e) {
			throw new ClassNotFoundException();
		}
	}
	public static void main(String[] args) throws ClassNotFoundException, InstantiationException, IllegalAccessException {
		ClassLoaderTest classLoader = new ClassLoaderTest();
		Object object = classLoader.loadClass("ca.mcmaster.jvm.classloader.ClassLoaderTest").newInstance();
		System.out.println(object.getClass());
		System.out.println(object instanceof ca.mcmaster.jvm.classloader.ClassLoaderTest);
	}
}
```

#### 双亲委派模型（Parents Delegation Model）
* 从JVM的角度讲，只存在两种类加载器：
	* 启动类加载器（BootstrapLoader），通过C++实现，是虚拟机的一部分。
	* 其他类加载器，是通过Java语言写成，独立于JVM外部，全部继承自抽象类java.lang.ClassLoader。

* 从Java程序员的角度，类加载器分成三种：
	* 启动类加载器(Bootstrap Classloader):这个类加载器将存储在<JAVA_HOME>/lib中，可以被JVM识别的类，加载到JVM中，无法被Java程序直接引用。
	* 扩展类加载器(Extension Classloader):负责加载<JAVA_HOME>/lib/ext，开发者可以直接引用。
	* 应用程序加载器(Application Classloader):一般称为系统加载器，如果程序员没有定义过自己的类的加载器，这就是程序中的默认加载器。
![Parents Delegation Model](https://i.imgur.com/4tZ3sAE.jpg)

* Proccess of Parent Delegation Model
> 如果一个类加载器收到了加载类的请求，他首先不会自己尝试去加载这个类，而是把这个请求委派给父类加载器去完成。只有当父类加载器反馈自己无法完成这个请求，子类加载器才会去尝试加载。

```Java
protected Class<?> loadClass(String name, boolean resolve) throws ClassNotFoundException
    {
        synchronized (getClassLoadingLock(name)) {
            // First, check if the class has already been loaded
            Class<?> c = findLoadedClass(name);	//首先判断当前类是否已经被加载到JVM中，如果没有加入则开始使用类加载器加载。
            if (c == null) {
                long t0 = System.nanoTime();
                try {
                    if (parent != null) {
                        c = parent.loadClass(name, false);	//递归调用上层的类加载器，例如使用extension classloader。
                    } else {
                        c = findBootstrapClassOrNull(name);	//如果已经到了最上层的类加载器，则使用Bootstrap classloader。
                    }
                } catch (ClassNotFoundException e) {
                    // ClassNotFoundException thrown if class not found
                    // from the non-null parent class loader
                }

                if (c == null) {
                    // If still not found, then invoke findClass in order
                    // to find the class.
                    long t1 = System.nanoTime();
                    c = findClass(name);	//如果最终仍然没有加载到类，则使用findClass进行类加载。

                    // this is the defining class loader; record the stats
                    sun.misc.PerfCounter.getParentDelegationTime().addTime(t1 - t0);
                    sun.misc.PerfCounter.getFindClassTime().addElapsedTimeFrom(t1);
                    sun.misc.PerfCounter.getFindClasses().increment();
                }
            }
            if (resolve) {
                resolveClass(c);	//进入解析过程
            }
            return c;
        }
    }
```

#### 如何使用ClassLoader类
1. 继承ClassLoader，并且重写loadClass类，这样就能提供寻找类加载器的机制。这破坏了Parents delegant method。（不推荐）
2. 继承ClassLoader，并重写findClass类。这样仍然利用了双亲委派机制，只是在所有的上层类加载器均无法使用时才会根据findClass()找到程序员自定义的类加载器。

#### OSGi(Open Services Gateway Initiative)实现模块化热部署
1. OSGI容器需要实现的服务集合。
2. OSGI容器和应用之间的通信机制。
3. OSGI容器被设计专门用来开发可分解为功能模块的复杂的Java应用。
4. 打个比方，在MVC的框架中，我们想要修改Dao的代码，我们可以热更新Dao而不需要重启服务器。

### 虚拟机字节码执行引擎
#### 运行时栈帧结构(Stack Frame)
> 用于支持虚拟机进行方法调用和方法执行的数据结构。

![Stack Frame](https://i.imgur.com/c9jGwbQ.png)
* 局部变量表
1. 最小单位为slot，可能为32位或是64位,如果是64位则会对小于64位的进行对齐或是补白。
2. 局部变量表示线程私有的，虽然long和double都是64位，会拆分成两个32位的数进行运算，但是仍然是线程安全的。
3. 在方法执行时，虚拟机使用局部变量表完成参数值到参数变量列表的传递过程。局部变量表第0位引索的Slot默认是1用于传递方法所属对象的引用，即this。
4. 假如当前分配变量的所占用内存很大，并且已经失去了对变量引用（离开了变量的作用域）。当下次需要使用该段内存空间时，该段内存将会被释放并且被垃圾收集器处理。
* 操作数栈
1. 这是一个先入后出的栈。
2. 两个操作数栈可能会有一部分重叠，就无需进行参数的复制传递了。
3. 各种字节码指令向操作数栈中写入和提取数据。例如算术运算时是通过操作数栈进行的，在调用别的方法时通过操作数栈传递参数。
* 动态连接
1. 在常量池中，存储了类的符号引用，包括类的类，接口，方法。当类被加载后，JVM会将常量池中的数据放入运行时常量池中。
* 方法返回地址
1. Normal Method Invocation complete：当JVM运到任意一个方法返回的字节指令。
2. Abrupt Method Invocation complete: 异常导致的方法退出。
3. 方法退出实际上等同于把前帧出站，操作：
	1. 恢复上层方法的局部变量表和操作数栈。
	2. 把返回值（如果存在）压入上层的操作数栈中。
	3. 调整PC计数器指向下一条调用的指令。

### 方法调用（Method Invocation）
> 方法调用不等于方法执行，方法调用阶段唯一的任务就是确定被调用方法的版本（哪一个方法），暂时不涉及方法内部的具体过程。一切方法调用在Class文件里面存储的都是符号引用，而不是方法在实际运行时内存布局中的入口地址。

* 解析
1. 在类解析的过程中，会将一部分符号引用转化成直接引用，这成立的前提是：方法在运行之前就有一个可确定的调用版本，并且这个方法的调用版本在运行期是不可改变的。
2. 符合“编译期可知，运行期不可变”这个要求的方法主要有静态方法和私有方法两大类。这两种方法都不能通过继承重写出其他版本，符合要求。
	* 分派
	> JVM实现重载和重写的机制

```Java
public class OverloadTest {
	static abstract class Human{
	}
	static class Man extends Human{
	}
	static class Woman extends Human{
	}
	public void sayHello(Human guy){
		System.out.println("Hello guy!");
	}
	public void sayHello(Man guy){
		System.out.println("Hello gentleman!");
	}
	public void sayHello(Woman guy){
		System.out.println("Hello lady!");
	}
	public static void main(String[] args) {
		Human man = new Man();	//此时Human是静态类型（Apparent Type），后面的Man是实际类型（Actual Type）。
		Human woman = new Woman();
		OverloadTest test = new OverloadTest();
		test.sayHello(man);	//在方法重载时，会根据静态类型决定使用哪个重载版本。所以此时静态类型是Human，所以会调用参数为Human的方法。
		test.sayHello(woman);
	}
}
Hello guy!
Hello guy!
```

* 静态分派
	> 所有依赖静态类型来定位方法执行版本的分派作用，都成为静态分派。最典型的应用就是方法重载。
1. 虚拟机（准确的说是编译器）是通过参数的静态类型而不是实际类型作为判断依据的。

* 动态分派
```Java
public class OverrideTest {
	static abstract class Human{
		public abstract void sayHello();
	}
	static class Man extends Human{
		@Override
		public void sayHello() {
			System.out.println("Man says Hello.");
		}
	}
	static class Woman extends Human{
		@Override
		public void sayHello() {
			System.out.println("Woman says hello!");
		}
	}
	public static void main(String[] args) {
		Human man = new Man();
		Human woman = new Woman();
		man.sayHello();
		woman.sayHello();
		man = new Woman();
		man.sayHello();
	}
}
```
1. 在当前类中找到与常量中的描述符和简单名称都相符的方法，权限校验，如果通过则返回这个方法的直接引用。
2. 否则按照继承关系从下往上对这个类的父类进行步骤1,。
3. 如果始终没有找到合适的方法，抛出java.lang.AbstractMethodError。
> 在运行期确定接收者的实际类型，这种在运行期根据实际类型确定方法执行版本的分派过程成为动态分派。

### 动态分派的实现
1. 动态分派是非常频繁的动作，虚拟机的实现基于性能的考虑。
2. 会在类的方法区中建立一个虚方法表（Virtual Method Table, vtable）。
![vtable](https://i.imgur.com/1wrhuvz.png)
3. 虚方法表中存在着每个方法的实际入口地址。如果一个方法没有在子类中被重写，子类的虚方法表将会存储父类的方法的实际地址。如果在子类中被重写了，将会存储子类的方法入口地址。

### 基于栈的字节码解释执行引擎
> JVM的执行引擎在执行Java代码时都有解释执行（通过解释器执行）和编译执行（通过即时编译器产生本地代码执行）
