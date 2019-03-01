---
layout: post
title:  "Algorithm | Shortest Path"
date:   2019-02-28 22:51
author: Botao Xiao
categories: Algorithm
comment: true
description: 
---
### Description
Shortest path can be applied to multiple situations. The most populor ones are Floyd, Dijkstra and SPFA.

### Floyd Algorithm
#### Process
Floyd Algorithm can be applied to calculate the shortest path on directed graph and weighted direct graph. It is a requirement that weights in derected graph are positive numbers.
1. We need two matrix to save the intermediate variables.
    * Matrix S: s[i][j] means the distance between vertex i and j.
    * Matrix P: p[i][j] means the path(or the list of vertexs) between i and j.
2. Process:
    * Step 0: Update i and j, if they are neighbours, s[i][j] is their path weight, p[i][j] is j, else inf.
    * Step 1: we are using another vertx as intermediate, we check if s[i][j] > s[i][0] + s[0][j], if true, we update s[i][j] to s[i][0] + s[0][j], meanwhile p[i][j] = p[i][0].
    * Step N: Compare s[i][j] > s[i][n] + s[n][j], follow the updating rule as step 2.

#### Example
![Imgur](https://i.imgur.com/NsAN6QQ.png)
1. Step 1: ![Imgur](https://i.imgur.com/gkfgmo0.jpg)
2. Step 2: ![Imgur](https://i.imgur.com/nTAiFNx.jpg)

#### Code
1. Floyd Algorithm
```Java
public static void main(String[] args) {
    int inf = Integer.MAX_VALUE / 2 - 1000;
    int[][] s = {{0, 12, inf, inf, inf, 16, 14},
            {12, 0, 10, inf, inf, 7, inf},
            {inf, 10, 0, 3, 5, 6, inf},
            {inf, inf, 3, 0, 4, inf, inf},
            {inf, inf, 5, 4, 0, 2, 8},
            {16, 7, 6, inf, 2, 0, 9},
            {14, inf, inf, inf, 8, 9, 0}};
    int[][] p = {{0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6},
            {0, 1, 2, 3, 4, 5, 6}};
    for(int i = 0; i < s.length; i++){ //Each step(update each intermediate vertex)
        for(int r = 0; r < s.length; r++){ //each row
            for(int c = 0; c < s.length; c++){ // Each col
                if(s[r][c] > s[r][i] + s[i][c]){
                    s[r][c] = s[r][i] + s[i][c];    // Update value.
                    p[r][c] = i;
                }
            }
        }
    }
    for(int i = 0; i < s.length; i++){  //print
        for(int j = 0; j < s.length; j++){
            System.out.print(s[i][j] + "\t");
        }
        System.out.println();
    }
}
```

2. Result
```
0	12	22	22	18	16	14	
12	0	10	13	9	7	16	
22	10	0	3	5	6	13	
22	13	3	0	4	6	12	
18	9	5	4	0	2	8	
16	7	6	6	2	0	9	
14	16	13	12	8	9	0	
```

### Dijkstra Algorithm
Dijkstra Algorithm is used to find the minimum path in directed graph, we need to know the begining vertex and all path weights are positive.

#### Process
1. Step 1: We take the neighbour of the beigining vertex and select minimum path of to its neighbour. So the path is fixed(since all path weights are positive, we won't get a smaller one).
2. Step 2: We relax the vertex according to current updated value.
![Imgur](https://i.imgur.com/cQPiUAL.jpg)

#### Code
```Java
public static void main(String[] args) {
        int inf = Integer.MAX_VALUE / 2 - 1000;
        int[][] s ={{0, 1, 12 , inf, inf ,inf},
                        {inf, 0, 9, 3, inf, inf},
                        {inf, inf, 0, inf, 5, inf},
                        {inf, inf, 4, 0, 13, 15},
                        {inf, inf, inf, inf, 0, 4},
                        {inf, inf, inf, inf, inf, 0}};
        Set<Integer> set = new HashSet<>();
        set.add(0);
        int[] result = Arrays.copyOf(s[0], s.length);
        while(set.size() < s.length){
            int idx = -1;
            int min = Integer.MAX_VALUE;
            for(int i = 1; i < s.length; i++){
                if(!set.contains(i) && min > result[i]){ //find all the vertex that are not visited and find the minimum value and index.
                    min = result[i];
                    idx = i;
                }
            }
            for(int i = 0; i < s.length; i++){
                if(!set.contains(i) && idx != -1){
                    result[i] = Math.min(result[i], s[idx][i] + min);   //Compare the values between 0 to different vertex and result[idx] + s[idx][i](this is the distance from idx to j).
                }
            }
            set.add(idx);
        }
        for (int num : result){
            System.out.print(num + "\t");
        }
        System.out.println();
    }
```

#### Result
```Java
0	1	8	4	13	17	
```

### Reference
1. [最短路径问题---Floyd算法详解](https://blog.csdn.net/qq_35644234/article/details/60875818)