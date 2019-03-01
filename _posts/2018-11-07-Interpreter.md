---
layout: post
title:  "Design Pattern | Interpreter Pattern"
date:   2018-11-07 08:43:59
author: Botao Xiao
categories: DesignPattern
comment: true
description: 解释器模式（Interpreter），给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。
---
> 解释器模式（Interpreter），给定一个语言，定义它的文法的一种表示，并定义一个解释器，这个解释器使用该表示来解释语言中的句子。

![interpreter Pattern](https://i.imgur.com/vhcNrrX.png)

### Interpreter Pattern中的角色
1. Expression： 抽象表达式,声明一个抽象的解释操作父类,定义一个抽象的解释方法,具体的实现由子类解释器完成。
2. 终结符表达式,实现文法中与终结符有关的解释操作,文法中每一个终结符都有一个具体的终结表达式与之对应。
3. NonterminalExpression: 非终结符表达式,实现文法中与非终结符有关的解释操作。
4. Context: 上下文环境类,包含解释器之外的全局信息。

### 解释器模式的实例
* 解释m + n + p， 其中我们将字母看作终结符号，将+代表的数学符号看做非终结符号。

1. 抽象表达式Expression
```Java
public interface Expression {
	public int interpret();	//声明抽象的解释方法。每个解释器元素都要实现解释方法。
}
```

2. 终结符解释器, 对数字进行语义分析
```Java
public class NumExpression implements Expression {
	private int num;
	public NumExpression(int num) {
		this.num = num;
	}
	@Override
	public int interpret() {	//继承自Expression方法，对语意进行分析。
		return this.num;
	}
}
```

3. 非终结符解释器
```Java
public abstract class OperationExpression implements Expression {
	protected NumExpression numExpression1;
	protected NumExpression numExpression2;
	public OperationExpression(NumExpression numExpression1, NumExpression numExpression2) {
		this.numExpression1 = numExpression1;
		this.numExpression2 = numExpression2;
	}
}
public class AdditionExpression extends OperationExpression {

	public AdditionExpression(NumExpression numExpression1,
			NumExpression numExpression2) {
		super(numExpression1, numExpression2);
	}
	@Override
	public int interpret() {	//依旧实现了解释方法，定义一种方法并进行操作。
		return super.numExpression1.interpret() + super.numExpression2.interpret();
	}
}
```

4. Context
```Java
public class Calculator {	//Context对象承载了除了解释器以外的信息并对解释器的调用进行了封装。
	protected Stack<NumExpression> numExpressionStack = new Stack<>();
	public Calculator(String expression){
		NumExpression numExpression1, numExpression2;
		String[] tokens = expression.split(" ");
		for(int i = 0; i < tokens.length; i++){
			switch (tokens[i].charAt(0)) {
			case '+':
				numExpression1 = numExpressionStack.pop();
				numExpression2 = new NumExpression(Integer.valueOf(tokens[++i]));
				numExpressionStack.push(new NumExpression(new AdditionExpression(numExpression1, numExpression2).interpret()));
				break;
			default:
				numExpressionStack.push(new NumExpression(Integer.valueOf(tokens[i])));
				break;
			}
		}
	}
	public int calculate(){
		return this.numExpressionStack.pop().interpret();
	}
	public static void main(String[] args) {
		System.out.println(new Calculator("1 + 2 + 3").calculate());
	}
}
```

### Conclusion
1. 灵活性强,如上边的例子,当我们想对文法规则进行扩展延伸时,只需要增加相应的非终结符解释器,并在构建语法树的时候使用新增的解释器对象进行具体的解释即可.


### Reference
1. [Java设计模式之解释器模式](https://blog.csdn.net/wbwjx/article/details/52456114)
