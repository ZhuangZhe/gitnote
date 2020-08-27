# Minimum Number of Vertices to Reach All Nodes

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/minimum-number-of-vertices-to-reach-all-nodes/)\*\*\*\*

## 题干

Given a **directed acyclic graph**, with `n` vertices numbered from `0` to `n-1`, and an array `edges` where `edges[i] = [fromi, toi]` represents a directed edge from node `fromi` to node `toi`.

Find _the smallest set of vertices from which all nodes in the graph are reachable_. It's guaranteed that a unique solution exists.

Notice that you can return the vertices in any order.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/07/07/untitled22.png)

```text
Input: n = 6, edges = [[0,1],[0,2],[2,5],[3,4],[4,2]]
Output: [0,3]
Explanation: It's not possible to reach all the nodes from a single vertex. From 0 we can reach [0,1,2,5]. From 3 we can reach [3,4,2,5]. So we output [0,3].
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/07/07/untitled.png)

```text
Input: n = 5, edges = [[0,1],[2,1],[3,1],[1,4],[2,4]]
Output: [0,2,3]
Explanation: Notice that vertices 0, 3 and 2 are not reachable from any other node, so we must include them. Also any of these vertices can reach nodes 1 and 4.
```

**Constraints:**

* `2 <= n <= 10^5`
* `1 <= edges.length <= min(10^5, n * (n - 1) / 2)`
* `edges[i].length == 2`
* `0 <= fromi, toi < n`
* All pairs `(fromi, toi)` are distinct.

## 思路

### stage 1

* 如度为0的节点必须选为起点，只要选取所有入度为0的就行，因为没办法从其他节点访问到。

```java
class Solution {
    public List<Integer> findSmallestSetOfVertices(int n, List<List<Integer>> edges) {
        List<Integer> res = new ArrayList<>();
        int[] inDegree = new int[n];
        
        for(List<Integer> edge : edges) {
            int from = edge.get(0);
            int to = edge.get(1);
            inDegree[to]++;
        }
        
        for(int i = 0 ; i < n ; i++) {
            if(inDegree[i] == 0)
                res.add(i);
        }
        
        return res;
    }
}
```







