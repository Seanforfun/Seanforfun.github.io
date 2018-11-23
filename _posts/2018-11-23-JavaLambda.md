---
layout: post
title:  "Java8| Lambda Expression"
date:   2018-11-22 15:38
author: Botao Xiao
categories: Javacore
description: Lambda表达式是一种简洁的表示可传递的匿名函数的一种方法：它没有名称，但是它有参数列表，函数主体，返回类型还能抛出异常列表。
---
# Lambda Expression
Lambda表达式是一种简洁的表示可传递的匿名函数的一种方法：它没有名称，但是它有参数列表，函数主体，返回类型还能抛出异常列表。
匿名： 没有明确的名称。
函数： 不像方法属于某个特定的类。
传递： Lambda表达式可以作为参数传递给方法或储存在变量中。
简洁：无需像匿名类写很多模板代码。

### Lambda表达式的构成
```Java
private Comparator<Apple> comparator =
            (Apple a1, Apple a2) -> a1.getWeight() - a2.getWeight();
```
1. 括号：参数列表
2. 箭头： 将参数列表和Lambda主体分隔开
3. Lambda主体：具体的函数给出返回值

### Lambda表达式的语法
1. (parameters) -> expression： 默认expression为返回值。
2. (parameters) ->{statements;}: 在花括号之中填充行为。

### Lambda的使用位置
在函数式接口（一个只定义一个抽象方法的接口）上使用抽象方法。Lambda表达式允许直接以内联的形式为函数式接口的抽象方法提供实现，并把整个表达式作为函数式接口的实例。

To Be continued.