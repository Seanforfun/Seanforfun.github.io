---
layout: post
title:  "Algorithm | Reservoir Sampling"
date:   2019-05-10 13:55
author: Botao Xiao
categories: Algorithm
comment: true
description: Algorithm to get random numbers from a unknown length array.
---
### Randomly get a number with known length
For a list with known length n, each of the number has a possibility 1/n to be selected.
```Java
int rand = random.nextInt(n);
return arr[rand];
```

### Randomly get a number without known length
Since we don't know the total length of the array, we don't really known what's the probability of single number is, but we want 1/n. So we use the reservoir sampling to find the value.

#### The procedure of picking:
1. For first one, we hold the value(doesn't mean to return the value, could be replaced).
2. For second one, it has a chance of 1/2 to replace the holding value.
3. For third one, it has a chance of 1/3 to replace the holding value.
4. For n, it has a chance of 1/n to replace the holding value.

#### Explaination:
* For a single number, the possibility for any single term to be chosen is:
    ```
    possibility to be chosen = possibility chosen to hold * possibility not to be replaced
    ```
* Provement, 1 <= a <= n
    * p = 1/a(hold) * ((a / a + 1) * (a + 1) / (a + 2) * ... * (n - 1)/n) = 1/n

### Randomly get k numbers without known length
1. Same as the previous part, we take the first k.
2. Then we take the [1, k + 1].
3. finally we take [n - k + 1, n]
![Imgur](https://i.imgur.com/lAW1Dcz.png)


### Reference
1. [【大数据算法】蓄水池抽样算法](https://www.cnblogs.com/MrLJC/p/4113276.html)