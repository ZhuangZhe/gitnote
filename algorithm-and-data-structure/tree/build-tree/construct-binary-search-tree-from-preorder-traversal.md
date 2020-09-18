# Construct Binary Search Tree from Preorder Traversal

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/construct-binary-search-tree-from-preorder-traversal/)\*\*\*\*

## 题干

Return the root node of a binary **search** tree that matches the given `preorder` traversal.

_\(Recall that a binary search tree is a binary tree where for every node, any descendant of `node.left` has a value `<` `node.val`, and any descendant of `node.right` has a value `>` `node.val`.  Also recall that a preorder traversal displays the value of the `node` first, then traverses `node.left`, then traverses `node.right`.\)_

It's guaranteed that for the given test cases there is always possible to find a binary search tree with the given requirements.

**Example 1:**

```text
Input: [8,5,1,7,10,12]
Output: [8,5,10,1,7,null,12]

```

**Constraints:**

* `1 <= preorder.length <= 100`
* `1 <= preorder[i] <= 10^8`
* The values of `preorder` are distinct.

## 思路

用一个栈存树的节点。从左往右遍历数组，每到一个数用这个构造一个新节点，如果栈为空，或栈顶的值大于当前值，链接当前节点到栈顶节点的左子树，否则判断栈中第二个值是够大于当前节点值，如果是，链接到当前节点的右子树。

```java
class Solution {
    public TreeNode bstFromPreorder(int[] preorder) {
        if(preorder == null || preorder.length == 0) 
            return null;
        
        Stack<TreeNode> stack = new Stack<>();
        TreeNode root = new TreeNode(preorder[0]);
        
        stack.push(root);
        
        for(int i = 1 ; i < preorder.length ; i++) {
            TreeNode cur = new TreeNode(preorder[i]);
            if(stack.peek().val > preorder[i]) {
                stack.peek().left = cur;
            } else {
                TreeNode pre = stack.pop();
                while(!stack.isEmpty() && stack.peek().val < preorder[i]) {
                    pre = stack.pop();
                } 
                pre.right = cur;
            }
            stack.push(cur);
        }
        
        return root;
    }
}
```

```java
class Solution {
    int i = 0;
    public TreeNode bstFromPreorder(int[] A) {
        return bstFromPreorder(A, Integer.MAX_VALUE);
    }

    public TreeNode bstFromPreorder(int[] A, int bound) {
        if (i == A.length || A[i] > bound)
            return null;
        TreeNode root = new TreeNode(A[i++]);
        root.left = bstFromPreorder(A, root.val);
        root.right = bstFromPreorder(A, bound);
        return root;
    }
}
```



