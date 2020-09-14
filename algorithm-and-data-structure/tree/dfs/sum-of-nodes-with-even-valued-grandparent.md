# Sum of Nodes with Even-Valued Grandparent



## 题干

Given a binary tree, return the sum of values of nodes with even-valued grandparent.  \(A _grandparent_ of a node is the parent of its parent, if it exists.\)

If there are no nodes with an even-valued grandparent, return `0`.

**Example 1:**

![](https://assets.leetcode.com/uploads/2019/07/24/1473_ex1.png)

```text
Input: root = [6,7,8,2,7,1,3,9,null,1,4,null,null,null,5]
Output: 18
Explanation: The red nodes are the nodes with even-value grandparent while the blue nodes are the even-value grandparents.
```

**Constraints:**

* The number of nodes in the tree is between `1` and `10^4`.
* The value of nodes is between `1` and `100`.

## 思路

重载函数，添加一个布尔数组用于记录父节点、祖父节点是否是偶数，在使用递归调用时可以判断的是否复合条件，同时更新父节点、祖父节点的更新。

```java
class Solution {
    public int sumEvenGrandparent(TreeNode root) {
        return helper(root, new boolean[]{false, false});
    }
    
    public int helper(TreeNode root, boolean[] summable) {
        if(root == null) 
            return 0;
        
        int sum = 0;
        if(summable[0]) 
            sum += root.val;
        
        boolean[] newSummable = new boolean[2];
        newSummable[0] = summable[1];
        newSummable[1] = root.val % 2 == 0 ? true : false;
        sum += helper(root.left, newSummable);
        sum += helper(root.right, newSummable);
        return sum;
    }
}
```



