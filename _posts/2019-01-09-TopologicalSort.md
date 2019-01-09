---
layout: post
title:  "Topological Sort 拓扑排序"
date:   2019-1-9 13:53
author: Botao Xiao
categories: Algorithm
comment: true
description: Topological sorting is based on DAG(Directed Acyclic Graph). There are so many implementations for topological sorting in real world, for example, course selection, driver installation, etc. The key requirement is "pre-requests". In DAG, we need to figure out the relationships between vertices and find out a way that we can traverse all vertices and match all pre-requests.
---
## Topological Sort 拓扑排序

### Introduction
Topological sorting is based on DAG(Directed Acyclic Graph). There are so many implementations for topological sorting in real world, for example, course selection, driver installation, etc. The key requirement is "pre-requests". In DAG, we need to figure out the relationships between vertices and find out a way that we can traverse all vertices and match all pre-requests.

![Imgur](https://i.imgur.com/nZ0kAAD.png)

### Basic knowledge
1. DAG: 
    * Directed: There is a order between different vertices, for u and v, we use (u, v) to show that u is the pre-request of v, and (u, v) is a arc between this two vertices, means this arc is from u to v.
    * Acyclic: Acyclic means there is no loop relationship between vertices. For example (a b), (b a) is a cyclic relationship. It is impossible for us to find the initial request for this loop, so there is no topological sorting for the cyclic graph.

2. Vertice:
    * indegree: How many arces are going into this vertice.
    * outdegree: How many arces are going out from this vertices.

### Two ways of topological sorting
#### Indegree table(BFS)
1. Create a indegree table and outdegree adj for DAG.
2. Delete the vertices whose indegree is zero and we decrease all outdegrees for the vertices connected to this vertice.
3. Until remove all vertices.
4. The following example is based on leetcode question [207. Course Schedule](https://leetcode.com/problems/course-schedule/description/)

```Java
class Solution {
    public List<Integer> sort(int numCourses, int[][] prerequisites) {
        int[] indegree = new int[numCourses];   //indegree table
        Map<Integer, List<Integer>> adj = new HashMap<>(); //outdegree table
        for(int[] pre : prerequisites){ //traveral all vertices and fill the indegree table and outdegree table
            List<Integer> temp = adj.containsKey(pre[0]) ? adj.get(pre[0]) : new ArrayList<Integer>();
            temp.add(pre[1]);
            adj.put(pre[0], temp);
            indegree[pre[1]]++;
        }
        LinkedList<Integer> queue = new LinkedList<>();
        for(int i = 0; i < numCourses; i++){
            if(indegree[i] == 0)  queue.add(i);
        }
        List<Integer> sort = new LinkedList<>();
        while(!queue.isEmpty()){    //Use bfs to traversal all vertices
            int pre = queue.poll();
            if(adj.containsKey(pre)){
                List<Integer> list = adj.get(pre);
                for(Integer v : list){
                    indegree[v]--;  // If indegree of current vertice is 0, we add current vertice to the queue.
                    if(indegree[v] == 0){
                        queue.add(v);
                        sort.add(v);
                    }
                }
            }
        }
        return sort;
    }
}
```

#### DFS
1. First get the indegree table and outdegree table.
2. Traverse all the vertex whose indegree table is zero, and recursively go through all vertex whose indegree is not zero. We will for sure visit all vertices if the indegree of current vertex is zero.
```Java
class Solution {
    List<Integer> result = new LinkedList<>();
    public boolean sort(int numCourses, int[][] prerequisites) {
        int[] indegree = new int[numCourses];
        Map<Integer, List<Integer>> adj = new HashMap<>();
        boolean[] visited = new boolean[numCourses];
        for(int[] pre : prerequisites){
            indegree[pre[1]]++;
            List<Integer> temp = adj.containsKey(pre[0]) ? adj.get(pre[0]): new ArrayList<>();
            temp.add(pre[1]);
            adj.put(pre[0], temp);
        }
        for(int i = 0; i < numCourses; i++){
            if(indegree[i] == 0 && !visited[i]){
                dfs(i, indegree, visited, adj);
            }
        }
        return this.result;
    }
    private void dfs(int cur, int[] indegree, boolean[] visited, Map<Integer, List<Integer>> adj){
        this.result.add(cur);
        visited[cur] = true;
        if(adj.containsKey(cur)){
            for(Integer next : adj.get(cur)){
                indegree[next]--;
                if(indegree[next] == 0 && !visited[next]){
                    dfs(next, indegree, visited, adj);
                }
            }
        }
    }
}
```

### Conclusion
1. For a DAG and we can still use bfs and dfs methods to traverse all vertex.
2. Must think about the indegree and outdegree of every single vertex.

### Reference
1. [【图论】有向无环图的拓扑排序](https://www.cnblogs.com/en-heng/p/5085690.html)