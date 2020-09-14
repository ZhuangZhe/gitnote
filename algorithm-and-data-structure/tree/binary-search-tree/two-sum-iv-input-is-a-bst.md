# Two Sum IV - Input is a BST



## 题干

Given a Binary Search Tree and a target number, return true if there exist two elements in the BST such that their sum is equal to the given target.

**Example 1:**

```text
Input: 
    5
   / \
  3   6
 / \   \
2   4   7

Target = 9

Output: True
```

**Example 2:**

```text
Input: 
    5
   / \
  3   6
 / \   \
2   4   7

Target = 28

Output: False
```

## 思路

```java
class Solution {
    public boolean findTarget(TreeNode root, int k) {
        Set<Integer> values = new HashSet<>();
        Stack<TreeNode> stack = new Stack<>();
        TreeNode cur = root;
        
        while(cur != null || !stack.isEmpty()) {
            if(cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop();
                
                if(values.contains(cur.val))
                    return true;
                
                values.add(k - cur.val);
                cur = cur.right;
            }
        }
        
        return false;
    }
}
```

