# Connecting Cities With Minimum CostMedium

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/connecting-cities-with-minimum-cost/)\*\*\*\*

## 题干

There are `N` cities numbered from 1 to `N`.

You are given `connections`, where each `connections[i] = [city1, city2, cost]` represents the cost to connect `city1` and `city2` together.  \(A _connection_ is bidirectional: connecting `city1` and `city2` is the same as connecting `city2` and `city1`.\)

Return the minimum cost so that for every pair of cities, there exists a path of connections \(possibly of length 1\) that connects those two cities together.  The cost is the sum of the connection costs used. If the task is impossible, return -1.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/04/20/1314_ex2.png)

```text
Input: N = 3, connections = [[1,2,5],[1,3,6],[2,3,1]]
Output: 6
Explanation: 
Choosing any 2 edges will connect all cities so we choose the minimum 2.
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/04/20/1314_ex1.png)

```text
Input: N = 4, connections = [[1,2,3],[3,4,4]]
Output: -1
Explanation: 
There is no way to connect all cities even if all edges are used.
```

**Note:**

1. `1 <= N <= 10000`
2. `1 <= connections.length <= 10000`
3. `1 <= connections[i][0], connections[i][1] <= N`
4. `0 <= connections[i][2] <= 10^5`
5. `connections[i][0] != connections[i][1]`

## **思路**

### **st​age 1**

1. 本质时找最小生成数，使用Kruskal或Prim算法就可以。

```java
class Solution {
    public int minimumCost(int N, int[][] connections) {
        //Kruskal kruskal = new Kruskal();
        //return kruskal.mstCost(N, connections);
        
        Prim prim = new Prim();
        return prim.mstCost(N, connections);
    }
    
    class Kruskal {

        private int[] roots;
        int treeCount;

        public int mstCost(int numOfNodes, int[][] edges) {
            roots = new int[numOfNodes];
            treeCount = numOfNodes;
            int cost = 0;

            for(int i = 0; i < numOfNodes; i++) {
                roots[i] = i;
            }

            Arrays.sort(edges, new Comparator<int[]>() {
                @Override
                public int compare(int[] o1, int[] o2) {
                    return o1[2] - o2[2];
                }
            });

            for(int[] edge : edges) {
                int node1 = edge[0] - 1;
                int node2 = edge[1] - 1;
                if(union(node1, node2))
                    cost += edge[2];
            }

            return treeCount == 1 ? cost : -1;
        }

        private int find(int id) {
            while(roots[id] != id) {
                id = roots[id];
            }
            return id;
        }

        private boolean union(int node1, int node2) {
            int root1 = find(node1);
            int root2 = find(node2);

            if(root1 == root2) {
                return false;
            } else if(root1 < root2) {
                roots[root2] = root1;
                treeCount--;
            } else {
                roots[root1] = root2;
                treeCount--;
            }

            return true;
        }
    }
    
    
    
    
    class Prim {

        public int mstCost(int numOfNode, int[][] edges) {
            if(numOfNode == 0)
                return 0;

            if(edges == null || edges.length == 0)
                return -1;

            // declare data
            int cost = 0;
            int selectedEdgeCount = 0;
            boolean[] visited = new boolean[numOfNode + 1];
            Map<Integer, Set<int[]>> graph = new HashMap<>();
            Queue<int[]> edgeMinHeap = new PriorityQueue<int[]>(new Comparator<int[]>() {
                @Override
                public int compare(int[] o1, int[] o2) {
                    return o1[2] - o2[2];
                }
            });

            // init data
            for(int[] edge : edges) {
                graph.computeIfAbsent(edge[0], addedEdge -> new HashSet<>()).add(edge);
                graph.computeIfAbsent(edge[1], addededge -> new HashSet<>()).add(new int[]{edge[1], edge[0], edge[2]});
            }
            visited[1] = true;
            for(int[] edge : graph.get(1)) {
                edgeMinHeap.offer(edge);
            }

            // process
            while(!edgeMinHeap.isEmpty()) {
                int[] edge = edgeMinHeap.poll();

                if(!visited[edge[1]]) {
                    visited[edge[1]] = true;
                    cost += edge[2];
                    selectedEdgeCount++;
                    for(int[] neighbor : graph.get(edge[1])) {
                        edgeMinHeap.offer(neighbor);
                    }
                }
            }

            return selectedEdgeCount == numOfNode - 1 ? cost : -1;
        }

    }
 
}
```

