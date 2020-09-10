# Range Sum of BST

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/range-sum-of-bst/)

## 题干

Given the `root` node of a binary search tree, return the sum of values of all nodes with value between `L` and `R` \(inclusive\).

The binary search tree is guaranteed to have unique values.

**Example 1:**

```text
Input: root = [10,5,15,3,7,null,18], L = 7, R = 15
Output: 32
```

**Example 2:**

```text
Input: root = [10,5,15,3,7,13,18,1,null,6], L = 6, R = 10
Output: 23
```

**Note:**

1. The number of nodes in the tree is at most `10000`.
2. The final answer is guaranteed to be less than `2^31`.

## 思路

输入是一个二分查找树，一个节点的左子树中所有节点都小于该节点，右子树中所有节点都大于该节点。

使用DFS层层向下遍历，同时判断当前值是否满足要求。

```java
class Solution {
    
    public int rangeSumBST(TreeNode root, int L, int R) {
        if(root == null)
            return 0;
        
        int res = 0;
        
        if(root.val >= L && root.val <= R)
            res += root.val;
        
        res += rangeSumBST(root.left, L, R);
        res += rangeSumBST(root.right, L, R);
        
        return res;
    }
}
```

