# Leaf-Similar Trees

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/leaf-similar-trees/)

## 题干

Consider all the leaves of a binary tree, from left to right order, the values of those leaves form a **leaf value sequence**_._

![](https://s3-lc-upload.s3.amazonaws.com/uploads/2018/07/16/tree.png)

For example, in the given tree above, the leaf value sequence is `(6, 7, 4, 9, 8)`.

Two binary trees are considered _leaf-similar_ if their leaf value sequence is the same.

Return `true` if and only if the two given trees with head nodes `root1` and `root2` are leaf-similar.

**Example 1:**![](https://assets.leetcode.com/uploads/2020/09/03/leaf-similar-1.jpg)

```text
Input: root1 = [3,5,1,6,2,9,8,null,null,7,4], root2 = [3,5,1,6,7,4,2,null,null,null,null,null,null,9,8]
Output: true
```

**Example 2:**

```text
Input: root1 = [1], root2 = [1]
Output: true
```

**Example 3:**

```text
Input: root1 = [1], root2 = [2]
Output: false
```

**Example 4:**

```text
Input: root1 = [1,2], root2 = [2,2]
Output: true
```

**Example 5:**![](https://assets.leetcode.com/uploads/2020/09/03/leaf-similar-2.jpg)

```text
Input: root1 = [1,2,3], root2 = [1,3,2]
Output: false
```

**Constraints:**

* The number of nodes in each tree will be in the range `[1, 200]`.
* Both of the given trees will have values in the range `[0, 200]`.

## 思路

先获取两棵树的叶节点序列。

如果两个叶节点序列大小不一样，返回false。

如果两个叶节点序列顺序有不一致的地方，返回false。

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



