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

### Lambda表达式的实现
1. 行为参数化：所谓的行为参数化是一种策略模式的体现。将一个函数式接口作为一种行为作为参数传递给另一个方法。此时p就是一种行为，对于文件的处理是不固定的，根据接口的实现类而定。典型的策略模式。当前例子中体现在参数FileReaderProcessor p。
```Java
public static String processFile(String file, FileReaderProcessor p) throws IOException {
    File f = new File(file);
    try(BufferedReader br = new BufferedReader(new FileReader(f))) {
        return p.process(br);
    }
}
```

2. 使用函数式接口传递行为。@FunctionalInterface,该接口中只有一个抽象方法。
```Java
@FunctionalInterface
public interface FileReaderProcessor {
    String process(BufferedReader br) throws IOException;
}
```

3. 执行行为。此时的行为是不确定的，需要传入策略。
```Java
return p.process(br);
```

4. 传递Lambda。此处的lambda充当接口的实现类。
```Java
public static void main(String[] args) throws IOException {
    System.out.println(processFile("E:\\LocalProject\\Concurrent\\src\\main\\java\\ca\\mcmaster\\concurrent\\pojo\\UserInfo.java",
            (BufferedReader br) -> {
                return br.readLine() + br.readLine() + br.readLine();
            }));
}
```

### Java8中新增的函数式接口
1. Predicate<T>:判断是否满足条件, 用于添加判断条件。
```Java
@FunctionalInterface
public interface Predicate<T> {
  /**
   *Evaluates this predicate on the given argument.
   *@param t the input argument
   *@return {@code true} if the input argument matches the predicate,
   *otherwise {@code false}
   */
  boolean test(T t);
}
```

2. Consumer<T>: 其中有一个accept方法，用于对某个元素进行操作，无返回值。
```Java
public interface Consumer<T> {
  /**
   *Performs this operation on the given argument.
   *@param t the input argument
   */
  void accept(T t);
}
// 对于每个元素而言，操作是隐藏的。
public static <T> void processList(List<T> list, Consumer<T> c){
    for(T t : list)
        c.accept(t);
}
List<Integer> list = Arrays.asList(1,2,3,4,5,6,7,8);
processList(list, (Integer i) -> System.out.println(i));  //实际的行为是打印每一个元素。
```
