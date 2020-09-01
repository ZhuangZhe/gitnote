# Validate Binary Tree Nodes

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/validate-binary-tree-nodes/)

## 题干

You have `n` binary tree nodes numbered from `0` to `n - 1` where node `i` has two children `leftChild[i]` and `rightChild[i]`, return `true` if and only if **all** the given nodes form **exactly one** valid binary tree.

If node `i` has no left child then `leftChild[i]` will equal `-1`, similarly for the right child.

Note that the nodes have no values and that we only use the node numbers in this problem.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/08/23/1503_ex1.png)

```text
Input: n = 4, leftChild = [1,-1,3,-1], rightChild = [2,-1,-1,-1]
Output: true
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/08/23/1503_ex2.png)

```text
Input: n = 4, leftChild = [1,-1,3,-1], rightChild = [2,3,-1,-1]
Output: false
```

**Example 3:**

![](https://assets.leetcode.com/uploads/2019/08/23/1503_ex3.png)

```text
Input: n = 2, leftChild = [1,0], rightChild = [-1,-1]
Output: false
```

**Example 4:**

![](https://assets.leetcode.com/uploads/2019/08/23/1503_ex4.png)

```text
Input: n = 6, leftChild = [1,-1,-1,4,-1,-1], rightChild = [2,-1,-1,5,-1,-1]
Output: false
```

**Constraints:**

* `1 <= n <= 10^4`
* `leftChild.length == rightChild.length == n`
* `-1 <= leftChild[i], rightChild[i] <= n - 1`

## 思路

### Stage 1

* 假如图中有两个或以上节点是无入边的，那么这个图就是不满足条件的。
* 一个节点如果有多于一条入边，那么这个图也是不符合条件的。

```java
class Solution {
    public boolean validateBinaryTreeNodes(int n, int[] leftChild, int[] rightChild) {
        
        Set<Integer> roots = new HashSet<>();
        Set<Integer> reached = new HashSet<>();
        
        for(int i = 0 ; i < n ; i++) {
            if(!reached.contains(i))
                roots.add(i);
            
            if(leftChild[i] != -1) {
                if(reached.contains(leftChild[i]))
                    return false;
                reached.add(leftChild[i]);
                
                if(roots.contains(leftChild[i]))
                    roots.remove(leftChild[i]);
                    
            }
            
            if(rightChild[i] != -1) {
                if(reached.contains(rightChild[i]))
                    return false;
                reached.add(rightChild[i]);
                
                if(roots.contains(rightChild[i]))
                    roots.remove(rightChild[i]);
            }
        }
        
        return roots.size() == 1;
    }
}
```



