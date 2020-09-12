# Invert Binary Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/invert-binary-tree/)

## 题干

Invert a binary tree.

**Example:**

Input:

```text
     4
   /   \
  2     7
 / \   / \
1   3 6   9
```

Output:

```text
     4
   /   \
  7     2
 / \   / \
9   6 3   1
```

## 思路

如果当前节点为空，直接返回空。

如果当前节点不为空，则尝试反转左右子树，并且尝试检查两子树是否可以交换子树，如此类推。

```java
class Solution {
    public TreeNode invertTree(TreeNode root) {
        if(root == null)
            return null;
        
        TreeNode left = invertTree(root.left);
        TreeNode right = invertTree(root.right);
        
        root.left = right;
        root.right = left;
        return root;
    }
}
```



