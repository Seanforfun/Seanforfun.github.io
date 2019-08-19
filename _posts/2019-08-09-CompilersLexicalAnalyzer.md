---
layout: post
title:  "Compiling | Lexical Analyzer 词法分析器 (ch)"
date:   2019-08-09 13:43
author: Botao Xiao
categories: Compiling
comment: true
description: Explain the theoretical detail of Lexical Analyzer. IN PROGRESS!!!
---

# Compiling | Lexical Analyzer 词法分析器

### 介绍
1. 编译器的基本结构
    
    ![Imgur](https://i.imgur.com/6c6Pvqi.png)

### 词法分析器（Lexical Analyzer）
词法分析是编译的第一个步骤。分析器接收字符流作为输入，将字符流解析成词素(lexeme)。

其中词素的表示方法是(token-name, attribute-value)。其中token name可以是一个虚拟符号(abstract symbol), 而attribute value指向符号表中的一个entry。

词法分析器分为两种类型：手工分析器和自动分析器。此处我们分析自动分析器的实现方法。

![Imgur](https://i.imgur.com/Pg5QP1Q.png)

### 正则表达式 Regex Expression
正则表达式是构成词法分析器的基础，实际上是一种用于定位匹配词素的基本规则。
1. 字符集Σ，提供了所有用于生成正则的字符, 例如Σ={e1, e2, ... , en}。

2. 正则表达式：
    * 简单正则表达式
        1. ε: 空字符。
        2. e1: 单独字符
    * 复合正则表达式
        3. ```e1|e2```: 选择表达式，当前字符可为e1或e2，此处e1,e2均可为复合表达式。
        4. e1e2: 连接表达式，连接两个表达式。有先后性。
        5. e*: 闭包表达式。 e* = {ε, e, ee, eee...}。

3. 除了上述五种基本表达式以外，还有一些为了简便操作而生成的表达式。
    * [e1-en]: ```e1 | e2 | e3 | ... | en```, 其中e1到en必须在字符集中是连续的。
    * e+: 一个或多个e， {e, ee, eee ...}
    * e?: {ε, e}
    * . : 除了'\n'以外的所有字符。
    * "a*": a*自身，不是闭包表达式（是否类似转义字符？）
    * e{i, j}: i到j个的e的连接。

4. 正则表达式是构造语句的规则。

### 有限状态自动机 Finite state automaton
有限状态机内部包含了有限个状态。我们希望将字符流输入到有限状态机中，并得到结果{yes|no}。

有限状态机可以被抽象成一个表达式:
```
FA = (Σ, S, q0, F, δ)
```
1. Σ 字符集。
2. S FA中所有状态的集合。
3. q0 初始状态。
4. F 终结状态的集合。
5. δ转移方程。可以写成转移方程表的形式。
    
    ![Imgur](https://i.imgur.com/H5hgRIV.png)

#### NFA  Nondeterministic finite automaton
当一个有限状态机中存在某个结点，当其接收的字符是确定的，转移的目标装态是不确定的，这种状态机被称为NFA。

![Imgur](https://i.imgur.com/gK03gdY.png)

#### DFA deterministic finite automaton
DFA是一个和NFA对应的概念。表示FA中的每个状态的转移都是明确的。

![Imgur](https://i.imgur.com/Mpo1S9P.png)

### Thompson算法: RE 转换 NFA
Thompson算法实现了RE向NFA的转换。具体原则是，基本RE直接构造，复合RE递归构造。
1. ε

    ![Imgur](https://i.imgur.com/oH1SjRC.png)
2. e

    ![Imgur](https://i.imgur.com/oH1SjRC.png)

3. N(s) | N(t)

    ![Imgur](https://i.imgur.com/eYmZXtw.png)

4. N(s) N(t)

    ![Imgur](https://i.imgur.com/NagW5TO.png)
   
5. N(s)*

    ![Imgur](https://i.imgur.com/Zi7XqMC.png)

#### Exmaple a(b|c)*
![Imgur](https://i.imgur.com/kURUbD3.jpg?1)

### Powerset construction: NFA to DFA
TO BE CONTINUED.

### 引用
1. [编译原理 Youtube](https://www.youtube.com/playlist?list=PLe68gYG2zUeXCCJBewCrWYCKGQc24ialj)
2. [编译原理 中文版第二版](https://www.baidu.com/link?url=6AcWWF-1TGwGYlkjrSXXlxCy51U2g8HL9NeweWSx2ut-P1X-DBcPA80g87RVa4_g2OFuhOCdS4cPEDRC2oLeMq&wd=&eqid=dddcf83000276611000000065d4d9d9c)
3. [编译原理 英文版](http://www.informatik.uni-bremen.de/agbkb/lehre/ccfl/Material/ALSUdragonbook.pdf)
4. [Finite-state machine](https://en.wikipedia.org/wiki/Finite-state_machine)
