---
layout: post
title:  "Algorithm | Bipartite of Graph"
date:   2019-05-13 14:42
author: Botao Xiao
categories: Algorithm
comment: true
description: Algorithm to check is a graph is bipartite.
---
# Algorithm | Bipartite of Graph

### Definition
All node in a bipartite graph is a undirect graph which can be divided into 2 sets and all nodes in the same set is not internal connected.
![Imgur](https://i.imgur.com/ffTa7a6.png)

### How we can check is a graph is bipartite?
There are two methods to check if a graph is bipartite, both of them use the color method.
* Method 1: Color method using dfs.
    ![Imgur](https://i.imgur.com/GvI8e9y.png)
    * for any initial node, we set the node to a color.
    * for its unvisited adjacency node, we want to color it to another color, if it is colored with the same color as current one, we return false.
    * otherwise we continue using recursion.
    ```Java
    class Solution {
        private int[] visited;
        private int[][] graph;
        public boolean isBipartite(int[][] graph) {
            this.visited = new int[graph.length];
            this.graph = graph;
            for(int i = 0; i < visited.length; i++){    // try to visited all nodes.
                if(visited[i] == 0){    // if current node is not visited, assign it to an initial value.
                    visited[i] = 1;
                    if(!dfs(i)) return false;   
                }
            }
            return true;
        }
        private boolean dfs(int index){
            for(int next : graph[index]){
                if(visited[next] == 0){ // if not initialized, we assign another value to that one.
                    visited[next] = -visited[index];
                    if(!dfs(next)) return false;
                }else if(visited[next] == visited[index])   // if we found 2 connected value to be the same, we know it cannot be bipartite.
                    return false;
            }
            return true;
        }
    }
    ```

* Method 2: Using bfs.
    ![Imgur](https://i.imgur.com/2weLlj0.png)
    1. we use loop to interate all nodes.
    2. If it is visited, skip it, otherwise add it to a queue.
    3. We assgin all nodes in different levels to be different values, it they are the same, we need to return false.
    ```Java
    class Solution {
        public boolean isBipartite(int[][] graph) {
            int[] visited = new int[graph.length];
            Queue<Integer> queue = new LinkedList<>();
            for(int i = 0; i < graph.length; i++){
                if(visited[i] == 0){
                    visited[i] = 1;
                    queue.offer(i);
                    while(!queue.isEmpty()){
                        int cur = queue.poll();
                        for(Integer next : graph[cur]){
                            if(visited[next] == visited[cur]) return false;
                            else if(visited[next] == 0){
                                visited[next] = -visited[cur];
                                queue.offer(next);
                            }
                        }
                    }
                }
            }
            return true;
        }
    }
    ```

### Reference
1. [花花酱 LeetCode 886. Possible Bipartition](https://zxi.mytechroad.com/blog/graph/leetcode-886-possible-bipartition/)