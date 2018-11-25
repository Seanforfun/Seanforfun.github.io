---
layout: post
title:  "Java8| Stream"
date:   2018-11-25 12:32
author: Botao Xiao
categories: JavaCore
description: 从支持数据处理操作的源生成的元素序列。
---
# Stream
从支持数据处理操作的源生成的元素序列。
元素序列：流提供了一个接口，可以访问特定元素类型的一组有序值。集合的目的在于以特定的时间/空间复杂度存储和访问元素，流的目的是为了计算。
源：流会使用一个提供数据的源，如集合，数组或输入输出资源。从有序集合生成流时会保存原有的顺序。
数据处理操作：filter，map, reduce, find, match, sort等操作，都是线程安全的。
流水线：很多流操作都会返回一个流，这样多个操作就能连接起来形成流水线。
内部迭代：流的迭代操作是隐式的，并不像集合需要显式的迭代器。

### Stream的操作
1. filter(), 流水线中的过滤操作。接收一个Predicate对象进行filter
```Java
    /**
     *Returns a stream consisting of the elements of this stream that match
     *the given predicate.
     */
    Stream<T> filter(Predicate<? super T> predicate);
```

2. map(), 接收一个Lambda，将元素转换成其他形式或者提取信息。
```Java
    /**
     *Returns a stream consisting of the results of applying the given
     *function to the elements of this stream.
     */
    <R> Stream<R> map(Function<? super T, ? extends R> mapper);
```

3. limit(), 截断流，使其元素不超过给定数量。
```Java
    /**
     *Returns a stream consisting of the elements of this stream, truncated
     *to be no longer than {@code maxSize} in length.
     */
    Stream<T> limit(long maxSize);
```

4. collect(), 将流转换为其他的形式，例如collect(toList())将流转换成一个链表。
```Java
<R, A> R collect(Collector<? super T, A, R> collector);
```

TO BE CONTINUED.













