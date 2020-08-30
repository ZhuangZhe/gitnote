# Number of Connected Components in an Undirected Graph

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/number-of-connected-components-in-an-undirected-graph/)

## 题干

Given `n` nodes labeled from `0` to `n - 1` and a list of undirected edges \(each edge is a pair of nodes\), write a function to find the number of connected components in an undirected graph.

**Example 1:**

```text
Input: n = 5 and edges = [[0, 1], [1, 2], [3, 4]]

     0          3
     |          |
     1 --- 2    4 

Output: 2
```

**Example 2:**

```text
Input: n = 5 and edges = [[0, 1], [1, 2], [2, 3], [3, 4]]

     0           4
     |           |
     1 --- 2 --- 3

Output:  1
```

**Note:**  
You can assume that no duplicate edges will appear in `edges`. Since all edges are undirected, `[0, 1]` is the same as `[1, 0]` and thus will not appear together in `edges`.

## 思路

### stage 1

1. 使用union-find重新建图，同时记录连通分量的。
2. 未连接时，一共n个节点，就有n个连通分量，用一个变量记录连通分量的数量，每连通两个连通分量就减少一个计数。

```java
class Solution {
    
    int[] roots;
    
    public int countComponents(int n, int[][] edges) {
        roots = new int[n];
        int count = n;
        
        for(int i = 0 ; i < n ; i++)
            roots[i] = i;
        
        for(int[] edge : edges) {
            if(union(edge[0], edge[1]))
                count--;
        }
        
        return count;
    }
    
    boolean union(int id1 , int id2) {
        int root1 = find(id1);
        int root2 = find(id2);
        if(roots[root1] != roots[root2]) {
            roots[root1] = root2;
            return true;
        }
        return false;
    }
    
    int find(int id) {
        while(id != roots[id]) {
            id = roots[id];
        }
        return id;
    }
    
}
```



