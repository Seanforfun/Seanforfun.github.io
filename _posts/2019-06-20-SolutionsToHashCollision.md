---
layout: post
title:  "Backend | Solutions to Hash Collision"
date:   2019-06-20 10:56
author: Botao Xiao
comment: true
categories: Backend
description: Solutions to hash collision
---
Here I listed several methods to solve the problem of Hash collision.

## Open address Method
The open address method will assign each key to a bucket. 

### Linear probing rehashing
* Formula: Hi=(H(key + di)) MOD m di=1,2,…,k(k<=m-1).
    1. di is linear, di = 1, 2, ..., m - 1.
    2. if H(key), we try H(key + i), i = 1, 2, ..., M-1

### Second order probing rehashing
* Formula: Hi=(H(key + di)) MOD m di = -1, 1, -4, 4, .., ((m - 1) / 2) ^2
    1. di is in second order.

### Fake random probing rehashing
* Formula: Hi=(H(key + di)) MOD m, di is a fake random number.
    1. This method is like hashhash.
    2. seed to random must be controllable.

## Hashhash
Hashhash means when we find a collison, we can apply hash calculation again:
* Formula: hash(hash...hash(key)), until we find a empty bucket.
* Pros: Tracable.
* Cons: Require too many calculation.

## Chaining Method
1. HashMap here is a K-V map where V here is a List holding the objects.
2. If there happens a hash collision, we need to append the object to the end of the list.
![Imgur](https://i.imgur.com/FvL85e2.png)
3. Pros:
    * Implement some algorithms like ip-defreagment.
    * Space saving
4. Cons: If the size of the hashmap is small, it will create some more space.

## Pulic collision Area
If a hash collision happends, we save it to a public collision area.

## Reference
1. [解决Hash碰撞冲突方法总结](https://blog.csdn.net/zeb_perfect/article/details/52574915)