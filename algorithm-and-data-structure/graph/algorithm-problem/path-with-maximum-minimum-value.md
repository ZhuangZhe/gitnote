# Path With Maximum Minimum Value

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/path-with-maximum-minimum-value/)

## 题干

Given a matrix of integers `A` with R rows and C columns, find the **maximum** score of a path starting at `[0,0]` and ending at `[R-1,C-1]`.

The _score_ of a path is the **minimum** value in that path.  For example, the value of the path 8 →  4 →  5 →  9 is 4.

A _path_ moves some number of times from one visited cell to any neighbouring unvisited cell in one of the 4 cardinal directions \(north, east, west, south\).

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/04/23/1313_ex1.JPG)

```text
Input: [[5,4,5],[1,2,6],[7,4,6]]
Output: 4
Explanation: 
The path with the maximum score is highlighted in yellow. 
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/04/23/1313_ex2.JPG)

```text
Input: [[2,2,1,2,2,2],[1,2,2,2,1,2]]
Output: 2
```

**Example 3:**

![](https://assets.leetcode.com/uploads/2019/04/23/1313_ex3.JPG)

```text
Input: [[3,4,6,3,4],[0,2,1,1,7],[8,8,3,2,7],[3,2,4,9,8],[4,1,2,0,0],[4,6,5,4,3]]
Output: 3
```

**Note:**

1. `1 <= R, C <= 100`
2. `0 <= A[i][j] <= 10^9`

## 思路

### stage 1

```java
class Solution {
    public int maximumMinimumPath(int[][] A) {
        
        final int[][] DIRS = {{0, 1}, {1, 0}, {0, -1},{-1, 0}};
        Queue<int[]> pq = new PriorityQueue<>((a, b) -> Integer.compare(b[0], a[0]));
        pq.add(new int[] {A[0][0], 0, 0});
        int maxscore = A[0][0];
        A[0][0] = -1;
        
        while(!pq.isEmpty()) {
            int[] top = pq.poll();
            int i = top[1], j = top[2], n = A.length, m = A[0].length;
            maxscore = Math.min(maxscore, top[0]);
            
            if(i == n - 1 && j == m - 1)
                break;
            
            for(int[] d : DIRS) {
                int newi = d[0] + i, newj = d[1] + j;
                if(newi >= 0 && newi < n && newj >= 0 && newj < m && A[newi][newj]>=0){
                    pq.add(new int[] {A[newi][newj], newi, newj});
                    A[newi][newj] = -1;
                }
            }
        }
        return maxscore;
    }
}
```



