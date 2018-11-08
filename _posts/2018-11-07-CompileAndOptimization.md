---
layout: post
title:  "Compile And Optimization"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: JVM
description: JVM的编译和优化。
---
### 早期（编译期）优化
> Java 语言的编译期是一端不确定的操作过程。这是一个把*.java转化成*.class的过程。

Sun javac的编译过程可以个分成三步：
1. 解析与填充符号表过程。
2. 插入式注解处理器的注解处理过程。
3. 分析与字节码生成过程。
![javac的编译过程](https://i.imgur.com/NDOFDuR.jpg)

### 源码解析（OpenJDK8）
> 编译的入口：com.sun.tools.javac.main.JavaCompiler

```Java
 try {
            initProcessAnnotations(processors);	//准备过程：初始化插入式注解处理器。

            // These method calls must be chained to avoid memory leaks
            delegateCompiler =
                processAnnotations(	//过程2：执行注解处理
					//enterTrees: 过程1.2:输入到符号表
					//parseFiles:过程1.1：词法分析、语法分析
                    enterTrees(stopIfError(CompileState.PARSE, parseFiles(sourceFileObjects))),
                    classnames);

            delegateCompiler.compile2();	//过程3：分析及字节码生成。
			//在compile2中
			{
				case BY_TODO:
                while (!todo.isEmpty())
				//generate,过程3.4：生成字节码
				//desugar,过程3.3：解语法糖
				//flow，过程3.2：数据流分析
				//attribute,过程3.1：标注
                    generate(desugar(flow(attribute(todo.remove()))));
                break;
			}
            delegateCompiler.close();
            elapsed_msec = delegateCompiler.elapsed_msec;
        } catch (Abort ex) {
            if (devVerbose)
                ex.printStackTrace(System.err);
        } finally {
            if (procEnvImpl != null)
                procEnvImpl.close();
        }
```

#### 解析与符号表填充
> 通过parseFiles()方法实现了编译原理中的词法分析和语法分析。

1. 词法分析(lexical analysis)将源代码的字符流转变成标记（Token）集合，单个Token是编译过程中的最小元素。例如int a = c + b, 可以被分解成int, a, =, c, +, b这六个Token.
2. 语法分析(grammatical analysis)根据Token序列来构造抽象语法树（Abstract Syntax Tree）的过程。AST中的每个结点都代表着程序代码中的一个语法结构。

#### 填充符号表
> 通过enterTrees()方法实现了填充符号表

1. 符号表(Symbol Table)是一组符号地址和符号信息。
2. 这个方法的出口是一个待处理列表(TO DO LIST)。包含了每一个编译单元的抽象语法树的顶级结点。

#### 注解处理器
> 在JDK5中注解是在运行期发挥作用的。而JDK6中提供了一组插入式注解器的标准，可以读取，修改，添加抽象语法树中的任意元素。

#### 语义分析和字节码生成
1. 语法分析后，编译器会获得抽象语法树，但是无法保证语义是正确的。
2. 我们需要通过语义分析来判断是否符合语法，其中分为标注检查和数据及控制流分析。
	* 标注检查，通过attribute()方法实现。
	* 数据及控制流分析，通过flow()方法实现。
3. 解语法糖
	* 语法糖（syntactic sugar）:并没有实现什么新的功能，但是只是为了程序员更方便使用。
	* 实际上JVM并不能准确地解释语法糖，所以在编译期间就会将语法糖解析成JVM可以识别的代码。
	* 由desugar()方法触发。
4. 字节码生成
	* 将前面步骤生成的信息转化成字节码写到磁盘中，并进行少量的代码添加和转化工作。
	* 例如添加默认的构造方法。字符串的加操作转化成StringBuilder或是StringBuffer方法。

### 晚期（运行期）优化
> Java的编译是通过解释器和编译器共同完成的。
1. 当程序一开始运行时，解释器首先发挥作用，省去编译时间，立即就可以开始运行。
2. 随着时间的推移，编译器将越来越多的代码编译成编译成本地代码，获取更高的执行效率。
3. 解释器和编译器的协同作用体现在解释器要替编译器收集性能监控信息，从而编译器可以变溢出更好的本地代码。

![Interpreter](https://i.imgur.com/KDztLic.png)

#### 分层编译（Tiered Compilation）
1. 解释器开始运行,解释器不开启监控功能(Profiling),可触发第一层编译。
2. 编译器开始编译（C1编译），加入简单可靠的优化，如有必要会加入性能监控的逻辑。
3. C2编译，将字节码编译成本地代码，会利用监控功能的信息做一些激进但不可靠的优化。

#### 热点代码
1. 多次调用的代码。
2. 多次循环的循环体。

* 如何判断一段代码是热点代码
	* Sample Based Hot spot detection
		* JVM会定期的采样栈顶的方法，如果总是出现则被判定为热点代码。
	* Counter based hot spot detection
		* JVM会为每个方法（甚至是代码块）分配计数器。
		![Counter based hot spot detection](https://i.imgur.com/NEgqfn8.png)

* 代码优化
	* Dead Code Elimination
	* Loop Unrolling
	* Loop Expression Hoisting
	* Common Subexpression Elimination
	* Constant Propagation
	* Basic Block Reordering