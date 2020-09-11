# Increasing Order Search Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/increasing-order-search-tree/)

## 题干

Given a binary search tree, rearrange the tree in **in-order** so that the leftmost node in the tree is now the root of the tree, and every node has no left child and only 1 right child.

```text
Example 1:
Input: [5,3,6,2,4,null,8,1,null,null,null,7,9]

       5
      / \
    3    6
   / \    \
  2   4    8
 /        / \ 
1        7   9

Output: [1,null,2,null,3,null,4,null,5,null,6,null,7,null,8,null,9]

 1
  \
   2
    \
     3
      \
       4
        \
         5
          \
           6
            \
             7
              \
               8
                \
                 9  
```

**Constraints:**

* The number of nodes in the given tree will be between `1` and `100`.
* Each node will have a unique integer value from `0` to `1000`.

## 思路

由于输入是一个BST，所以边用中序遍历，边讲节点重新链接起来即可。

```java
class Solution {
    public TreeNode increasingBST(TreeNode root) {
        if(root == null)
            return null;
        TreeNode[] res = helper(root);
        return res[0];
    }
    
    TreeNode[] helper(TreeNode root) {
        TreeNode[] res = new TreeNode[]{root, root};
        
        if(root.left != null) {
            TreeNode[] left = helper(root.left);
            root.left = null;
            left[1].right = root;
            res[0] = left[0];
        }
        
        if(root.right != null) {
            TreeNode[] right = helper(root.right);
            root.right = right[0];
            res[1] = right[1];
        }
        
        return res;
    }
}
```

也可以先获取顺序，再重新建树，就是得多遍历一次。

```java
class Solution {    
    public TreeNode increasingBST(TreeNode root) {
        List<Integer> vals = new ArrayList();
        inorder(root, vals);
        TreeNode ans = new TreeNode(0), cur = ans;
        for (int v: vals) {
            cur.right = new TreeNode(v);
            cur = cur.right;
        }
        return ans.right;
    }

    public void inorder(TreeNode node, List<Integer> vals) {
        if (node == null) return;
        inorder(node.left, vals);
        vals.add(node.val);
        inorder(node.right, vals);
    }
}
```

```java
class Solution {
    TreeNode cur;
    public TreeNode increasingBST(TreeNode root) {
        TreeNode ans = new TreeNode(0);
        cur = ans;
        inorder(root);
        return ans.right;
    }

    public void inorder(TreeNode node) {
        if (node == null) return;
        inorder(node.left);
        node.left = null;
        cur.right = node;
        cur = node;
        inorder(node.right);
    }
}
```

