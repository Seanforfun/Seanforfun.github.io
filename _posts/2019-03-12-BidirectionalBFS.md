---
layout: post
title:  "Algorithm | Bidirectional bfs"
date:   2019-03-12 13:40
author: Botao Xiao
categories: Algorithm
comment: true
description: Bidirectional bfs provides us a chance to search in both ways and may save some useless steps
---
### Why use bfs
bfs is used to solve shortest path question, we don't need to find all possible solutions and we don't even care about the path to destination, all we care is to find the minimum step to reach the destination.

### How to choose dfs and bfs
1. bfs is designed for shortest path question.
2. At one single stage, we could get the next stage and in every single step, the result is optimized, we can use bfs.
3. dfs can also solve bfs questions while it takes longer time.
4. dfs is used for recording all possible solutions(combination and permutation questions).

### What is bidirectional bfs
1. Bidirectional bfs provides us a chance to search in both ways and may save some useless steps, we search from the beginning and end point in turns(not really in turns but taking the smallest size).
2. Common bfs time efficiency is O(b^d), where b is the branching facter and d is the distance from source to destination. Bidirectional bfs can reduce the time efficiency to O(b^(d/2)).
![Imgur](https://i.imgur.com/X8K9Er9.png)

### Pseudo code
```
headSet add head element.
tailSet add tail element.
while not both set empty:
    ++step;
    s = pick min_size(head, tail);
    next = {};
    new_element = expend(node in s);
    if(new_element in the other set) return step + 1;
    next.add(new_element);
    head(or tail) = next;
return NOT_FOUND;
```

### Example Question(Leetcode 127)
1. [127. Word Ladder](https://github.com/Seanforfun/Algorithm-and-Leetcode/blob/master/leetcode/127.%20Word%20Ladder.md)

### Reference
1. [花花酱 LeetCode 127. Word Ladder](http://zxi.mytechroad.com/blog/searching/127-word-ladder/)