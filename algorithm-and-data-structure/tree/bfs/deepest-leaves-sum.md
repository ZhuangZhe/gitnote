# Deepest Leaves Sum

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/deepest-leaves-sum/)

## 题干

Given a binary tree, return the sum of values of its deepest leaves.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/07/31/1483_ex1.png)

```text
Input: root = [1,2,3,4,5,null,6,7,null,null,null,null,8]
Output: 15
```

**Constraints:**

* The number of nodes in the tree is between `1` and `10^4`.
* The value of nodes is between `1` and `100`.

## 思路

层层往下遍历，直到找到最底下的一层，将那层所有元素加起来，基本的BFS。

```java
class Solution {
    
    public int deepestLeavesSum(TreeNode root) {
        //return recursive(root);
        
        return iterative(root);
    }
    
    int iterative(TreeNode root) {
        int res = 0;
        List<TreeNode> curLevel = new ArrayList<>();
        
        if(root != null)
            curLevel.add(root);
            
        while(!curLevel.isEmpty()) {
            List<TreeNode> nextLevel = new ArrayList<>();
            res = 0;
            for(TreeNode cur : curLevel) {
                res += cur.val;
                
                if(cur.left != null)
                    nextLevel.add(cur.left);
                
                if(cur.right != null)
                    nextLevel.add(cur.right);
            }
            curLevel = nextLevel;
        }
        return res;
    }
    
    int sum = 0;
    int depth = 0;
    int recursive(TreeNode root) {
        recursive(root, 1);
        return sum;
    }
    
    public void recursive(TreeNode root, int curDepth) {
        if(root == null)
            return;
        if(root.left == null && root.right == null) {
            if(curDepth < depth)
                return;
            else if(curDepth == depth)
                sum += root.val;
            else {
                sum = root.val;
                depth = curDepth;
            }
        }
        recursive(root.left, curDepth + 1);
        recursive(root.right, curDepth + 1);
    }
    
}
```

