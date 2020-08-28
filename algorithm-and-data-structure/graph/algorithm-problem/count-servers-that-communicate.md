# Count Servers that Communicate

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/count-servers-that-communicate/)\*\*\*\*

## 题干

You are given a map of a server center, represented as a `m * n` integer matrix `grid`, where 1 means that on that cell there is a server and 0 means that it is no server. Two servers are said to communicate if they are on the same row or on the same column.  
  
Return the number of servers that communicate with any other server.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/11/14/untitled-diagram-6.jpg)

```text
Input: grid = [[1,0],[0,1]]
Output: 0
Explanation: No servers can communicate with others.
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/11/13/untitled-diagram-4.jpg)

```text
Input: grid = [[1,0],[1,1]]
Output: 3
Explanation: All three servers can communicate with at least one other server.
```

**Example 3:**

![](https://assets.leetcode.com/uploads/2019/11/14/untitled-diagram-1-3.jpg)

```text
Input: grid = [[1,1,0,0],[0,0,1,0],[0,0,1,0],[0,0,0,1]]
Output: 4
Explanation: The two servers in the first row can communicate with each other. The two servers in the third column can communicate with each other. The server at right bottom corner can't communicate with any other server.
```

**Constraints:**

* `m == grid.length`
* `n == grid[i].length`
* `1 <= m <= 250`
* `1 <= n <= 250`
* `grid[i][j] == 0 or 1`

## 思路

### stage 1

1. 一个服务器是否可以和别的服务器连通，在于是否有其他服务器和他同行或同列。
2. 遍历一次数组记录下每行和每列有几个服务器，并且记录服务器总数。
3. 然后再遍历服务器一次，将无同行同列服务器的服务器减去。

```java
class Solution {
    public int countServers(int[][] grid) {
        if(grid == null || grid.length == 0)
            return 0;
        
        int height = grid.length;
        int width = grid[0].length;
        int totalServerCount = 0;
        int[] serverInRowsCount = new int[height];
        int[] serverInColsCount = new int[width];
        Set<int[]> serverLocations = new HashSet<>();
        
        for(int i = 0 ; i < height ; i++) {
            for(int j = 0 ; j < width ; j++) {
                if(grid[i][j] == 1) {
                    serverInRowsCount[i]++;
                    serverInColsCount[j]++;
                    totalServerCount++;
                    serverLocations.add(new int[]{i , j});
                } 
            }
        }
        
        int communicableServerCount = totalServerCount;
        for(int[] location : serverLocations) {
            if(serverInRowsCount[location[0]] == 1 && serverInColsCount[location[1]] == 1)
                communicableServerCount--;
        }
        
        return communicableServerCount;
    }
}
```







