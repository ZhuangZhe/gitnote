# Univalued Binary Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/univalued-binary-tree/)

## 题干

A binary tree is _univalued_ if every node in the tree has the same value.

Return `true` if and only if the given tree is univalued.

**Example 1:**![](https://assets.leetcode.com/uploads/2018/12/28/unival_bst_1.png)

```text
Input: [1,1,1,1,1,null,1]
Output: true
```

**Example 2:**![](https://assets.leetcode.com/uploads/2018/12/28/unival_bst_2.png)

```text
Input: [2,2,2,5,2]
Output: false
```

**Note:**

1. The number of nodes in the given tree will be in the range `[1, 100]`.
2. Each node's value will be an integer in the range `[0, 99]`.

## 思路

```java
class Solution {
    public boolean isUnivalTree(TreeNode root) {
        if(root == null)
            return true;
        
        return isUnivalTree(root, root.val);
    }
    
    public boolean isUnivalTree(TreeNode root, int val) {
        if(root == null)
            return true;
        
        if(root.val != val)
            return false;
        
        return isUnivalTree(root.left, val) && isUnivalTree(root.right, val);
    }
}
```

