# Minimum Absolute Difference in BST

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/minimum-absolute-difference-in-bst/)

## 题干

Given a binary search tree with non-negative values, find the minimum [absolute difference](https://en.wikipedia.org/wiki/Absolute_difference) between values of any two nodes.

**Example:**

```text
Input:

   1
    \
     3
    /
   2

Output:
1

Explanation:
The minimum absolute difference is 1, which is the difference between 2 and 1 (or between 2 and 3).
```

**Note:**

* There are at least two nodes in this BST.
* This question is the same as 783: [https://leetcode.com/problems/minimum-distance-between-bst-nodes/](https://leetcode.com/problems/minimum-distance-between-bst-nodes/)

## 思路

```java
class Solution {
    public int getMinimumDifference(TreeNode root) {
        //return iteractive(root);
        
        return recursive(root);
    }
    
    int iteractive(TreeNode root) {
        int min = Integer.MAX_VALUE;
        TreeNode cur = root;
        TreeNode prev = null;
        Stack<TreeNode> stack = new Stack<>();
        
        while(cur != null || !stack.isEmpty()) {
            if(cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop();
                if(prev != null)
                    min = Math.min(min , cur.val - prev.val);
                prev = cur;
                cur = cur.right;
            }
        }
        
        return min;
    }
    
    int res = Integer.MAX_VALUE;
    TreeNode previous = null;
    int recursive(TreeNode root) {
        dfs(root);
        return res;
    }
    
    void dfs(TreeNode root) {
        if(root == null)
            return;
        
        dfs(root.left);
        
        if(previous != null)
            res = Math.min(res , root.val - previous.val);
        
        previous = root;
        
        dfs(root.right);
    }
}
```



