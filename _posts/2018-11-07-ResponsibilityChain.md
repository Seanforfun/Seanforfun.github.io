---
layout: post
title:  "Chain of Responsibility 责任链模式"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
description: 使多个对象都有机会处理同一个请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。
---
# Chain of Responsibility 责任链模式
> 使多个对象都有机会处理同一个请求，从而避免请求的发送者和接收者之间的耦合关系。将这些对象连成一条链，并沿着这条链传递该请求，直到有一个对象处理它为止。

![Chain of Responsibility 责任链模式](https://i.imgur.com/MBCBiIJ.png)

* 实际的执行流程
![责任链模式](https://i.imgur.com/CiUVc8D.png)

### Chain of Responsibility实现流程
0. 创建任务对象,在对象中存储了可以被哪个职责进行处理。
```Java
public class Task {
	private Integer level;
	private String taskName;
	public Task(Integer level, String task){
		this.level = level;
		this.taskName = task;
	}
	public Integer getLevel() {
		return level;
	}
	public void setLevel(Integer level) {
		this.level = level;
	}
	public String getTaskName() {
		return taskName;
	}
	public void setTaskName(String taskName) {
		this.taskName = taskName;
	}
}
```

1. 创建一个处理对象的接口
```Java
public interface Handler {	//通用的处理行为。
	public void handle(Task task) throws Exception;
}
```

2. 创建抽象类，实现通用性的方法，留出抽象方法作为对事物的具体处理。
```Java
public abstract class AbstractHandler implements Handler {
	protected Integer level;
	protected AbstractHandler nextHandler;
	@Override
	public void handle(Task task) throws Exception {
		if(this.level == task.getLevel()){	//如果当前对象可以处理，则让当前类进行处理。
			doHandler(this.level, task);
		}else{
			if(null != nextHandler){	//当前对象无法处理，则抛给责任链的下一个对象进行处理
				try {
					nextHandler.handle(task);
				} catch (Exception e) {	//接收责任链的下一个对象抛出的异常。
					System.out.println(nextHandler.level + ": Not able to handle the task!");
					throw new Exception();	//继续向上抛出异常。
				}
			}
			else
				throw new Exception("No one can deal with this task!");	//责任链的最后一个职责，如果仍无法处理，则抛出异常。
		}
	}
	public abstract void doHandler(Integer level, Task task);	//具体的对对象处理的方法实现。
}
```

3. 实现具体的可以处理任务的类
```Java
public class ConcreteHandler extends AbstractHandler {
	public ConcreteHandler(Integer level, AbstractHandler nextHandler) {
		super.nextHandler = nextHandler;
		super.level = level;
	}
	@Override
	public void doHandler(Integer level, Task task) {	//对任务的具体实现
		System.out.println(level + ": finish the task " + task.getTaskName());
	}
	public static void main(String[] args) {
		AbstractHandler handler3 = new ConcreteHandler(3, null);	//定义出责任链
		AbstractHandler handler2 = new ConcreteHandler(2, handler3);
		AbstractHandler handler1 = new ConcreteHandler(1, handler2);
		AbstractHandler handler = new ConcreteHandler(0, handler1);
		Task task = new Task(4, "A task!");	//此时的任务无法被处理。
		try {
			handler.handle(task);	//让责任链的最外层对象处理任务，内部会进行回归调用。
		} catch (Exception e) {
			System.out.println("No one can handle this task!");
		}
	}
}
3: Not able to handle the task!
2: Not able to handle the task!
1: Not able to handle the task!
No one can handle this task!
```

### 总结
1. 不确定处理对象的任务的处理模式。

### Reference
1. [Java设计模式之《职责链模式》及应用场景](https://www.cnblogs.com/V1haoge/p/6530089.html)