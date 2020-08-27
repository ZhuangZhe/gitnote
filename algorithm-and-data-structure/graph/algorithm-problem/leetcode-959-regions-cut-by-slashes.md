# Regions Cut By Slashes

\*\*\*\*[**LeetCode 链接**](https://leetcode.com/problems/regions-cut-by-slashes/)\*\*\*\*

## **题干**

In a N x N `grid` composed of 1 x 1 squares, each 1 x 1 square consists of a `/`, `\`, or blank space.  These characters divide the square into contiguous regions.

\(Note that backslash characters are escaped, so a `\` is represented as `"\\"`.\)

Return the number of regions.

1. 
**Example 1:**

```text
Input:
[
  " /",
  "/ "
]
Output: 2
Explanation: The 2x2 grid is as follows:
```

![](../../../.gitbook/assets/image%20%2890%29.png)

**Example 2:**

```text
Input:
[
  " /",
  "  "
]
Output: 1
Explanation: The 2x2 grid is as follows:
```

![](../../../.gitbook/assets/image%20%2895%29.png)

**Example 3:**

```text
Input:
[
  "\\/",
  "/\\"
]
Output: 4
Explanation: (Recall that because \ characters are escaped, "\\/" refers to \/, and "/\\" refers to /\.)
The 2x2 grid is as follows:
```

![](../../../.gitbook/assets/image%20%2889%29.png)

**Example 4:**

```text
Input:
[
  "/\\",
  "\\/"
]
Output: 5
Explanation: (Recall that because \ characters are escaped, "/\\" refers to /\, and "\\/" refers to \/.)
The 2x2 grid is as follows:
```

![](../../../.gitbook/assets/image%20%2892%29.png)

**Example 5:**

```text
Input:
[
  "//",
  "/ "
]
Output: 3
Explanation: The 2x2 grid is as follows:
```

![](../../../.gitbook/assets/image%20%2893%29.png)

**Note:**

1. `1 <= grid.length == grid[0].length <= 30`
2. `grid[i][j]` is either `'/'`, `'\'`, or `' '`.

## **思路**

### **Stage 1**

1. 使用union-find判断相邻的方块是否连通
2. 一个方块分成四个全等三角形，分别判断与上下左右是否连通，给这四个三角形分别编号，使用函数：`row * col * 4 + triangleId`
3. 按先行后列的顺序遍历，访问每个方块时，先和左边上边的方块链接，再根据方块本身是否有斜线和斜线的方向来进行内部连通
4. 三角形ID分别为：左0、上1、右2、下3

```java
class Solution {
    int count, n;
    int[] f;
    
    public int regionsBySlashes(String[] grid) {
        n = grid.length;
        f = new int[n * n * 4];
        count = n * n * 4;
        for (int i = 0; i < n * n * 4; ++i)
            f[i] = i;
        for (int i = 0; i < n; ++i) {
            for (int j = 0; j < n; ++j) {
                if (i > 0) union(g(i - 1, j, 2), g(i, j, 0));
                if (j > 0) union(g(i , j - 1, 1), g(i , j, 3));
                if (grid[i].charAt(j) != '/') {
                    union(g(i , j, 0), g(i , j,  1));
                    union(g(i , j, 2), g(i , j,  3));
                }
                if (grid[i].charAt(j) != '\\') {
                    union(g(i , j, 0), g(i , j,  3));
                    union(g(i , j, 2), g(i , j,  1));
                }
            }
        }
        return count;
    }

    public int find(int x) {
        if (x != f[x]) {
            f[x] = find(f[x]);
        }
        return f[x];
    }
    
    public void union(int x, int y) {
        x = find(x); 
        y = find(y);
        if (x != y) {
            f[x] = y;
            count--;
        }
    }
    
    public int g(int i, int j, int k) {
        return (i * n + j) * 4 + k;
    }
}
```

{% page-ref page="../algorithm/unionfind.md" %}



\*\*\*\*

