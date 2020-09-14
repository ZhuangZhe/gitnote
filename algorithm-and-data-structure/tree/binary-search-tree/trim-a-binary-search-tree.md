# Trim a Binary Search Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/trim-a-binary-search-tree/)

## 题干

Given the `root` of a binary search tree and the lowest and highest boundaries as `low` and `high`, trim the tree so that all its elements lies in `[low, high]`. You might need to change the root of the tree, so the result should return the new root of the trimmed binary search tree.

**Example 1:**![](https://assets.leetcode.com/uploads/2020/09/09/trim1.jpg)

```text
Input: root = [1,0,2], low = 1, high = 2
Output: [1,null,2]
```

**Example 2:**![](https://assets.leetcode.com/uploads/2020/09/09/trim2.jpg)

```text
Input: root = [3,0,4,null,2,null,null,1], low = 1, high = 3
Output: [3,2,null,1]
```

**Example 3:**

```text
Input: root = [1], low = 1, high = 2
Output: [1]
```

**Example 4:**

```text
Input: root = [1,null,2], low = 1, high = 3
Output: [1,null,2]
```

**Example 5:**

```text
Input: root = [1,null,2], low = 2, high = 4
Output: [2]
```

**Constraints:**

* The number of nodes in the tree in the range `[1, 104]`.
* `0 <= Node.val <= 104`
* The value of each node in the tree is **unique**.
* `root` is guaranteed to be a valid binary search tree.
* `0 <= l <= r <= 104`

## 思路

如果当前节点为空，返回空。

如果当前节点小于左边际，对当前节点的右子树进行递归调用，并返回结果。

如果当前节点大于右边际，对当前节点的左子树进行递归调用，并返回结果。

如当前节点在范围内，对两子树分别进行递归调用，将结果分别连接到当前节点，放回当前节点。

```java
class Solution {
    public TreeNode trimBST(TreeNode root, int low, int high) {
        if(root == null)
            return null;
        
        TreeNode left = trimBST(root.left, low, high);
        TreeNode right = trimBST(root.right, low, high);
        
        if(root.val < low) 
            return right;
        
        if(root.val > high)
            return left;
        
        root.left = left;
        root.right = right;
        return root;
    }
}
```



