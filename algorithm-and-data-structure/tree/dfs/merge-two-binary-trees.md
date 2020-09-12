# Merge Two Binary Trees

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/merge-two-binary-trees/)

## 题干

Given two binary trees and imagine that when you put one of them to cover the other, some nodes of the two trees are overlapped while the others are not.

You need to merge them into a new binary tree. The merge rule is that if two nodes overlap, then sum node values up as the new value of the merged node. Otherwise, the NOT null node will be used as the node of new tree.

**Example 1:**

```text
Input: 
	Tree 1                     Tree 2                  
          1                         2                             
         / \                       / \                            
        3   2                     1   3                        
       /                           \   \                      
      5                             4   7                  
Output: 
Merged tree:
	     3
	    / \
	   4   5
	  / \   \ 
	 5   4   7
```

**Note:** The merging process must start from the root nodes of both trees.

## 思路

情况一：两个节点都是null，返回空。

情况二：有一个或两个节点不为努力了，对两个节点两个的子节点进行递归。

### 不改输入

```java
class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if(t1 == null && t2 == null) 
            return null;
        
        int value = 0;
        if(t1 != null)
            value += t1.val;
        if(t2 != null)
            value += t2.val;
        
        TreeNode root = new TreeNode(value);
        root.left = mergeTrees(t1 == null ? null : t1.left, t2 == null ? null : t2.left);
        root.right = mergeTrees(t1 == null ? null : t1.right, t2 == null ? null : t2.right);
        
        return root;
    }
}
```

### 改输入

```java
class Solution {
    public TreeNode mergeTrees(TreeNode t1, TreeNode t2) {
        if(t1 == null && t2 == null)
            return null;
        
        TreeNode left = mergeTrees(t1 != null ? t1.left : null, t2 != null ? t2.left : null);
        TreeNode right = mergeTrees(t1 != null ? t1.right : null, t2 != null ? t2.right : null);
        
        if(t1 != null && t2 != null)
            t1.val += t2.val;
        else if(t2 != null)
            t1 = t2;
        
        t1.left = left;
        t1.right = right;
            
        return t1;
    }
}
```





