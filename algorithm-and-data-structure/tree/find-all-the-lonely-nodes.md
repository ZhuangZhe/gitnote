# Find All The Lonely Nodes

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/find-all-the-lonely-nodes/)\*\*\*\*

## **题干**

n a binary tree, a **lonely** node is a node that is the only child of its parent node. The root of the tree is not lonely because it does not have a parent node.

Given the `root` of a binary tree, return _an array containing the values of all lonely nodes_ in the tree. Return the list **in any order**.

**Example 1:**![](https://assets.leetcode.com/uploads/2020/06/03/e1.png)

```text
Input: root = [1,2,3,null,4]
Output: [4]
Explanation: Light blue node is the only lonely node.
Node 1 is the root and is not lonely.
Nodes 2 and 3 have the same parent and are not lonely.
```

**Example 2:**![](https://assets.leetcode.com/uploads/2020/06/03/e2.png)

```text
Input: root = [7,1,4,6,null,5,3,null,null,null,null,null,2]
Output: [6,2]
Explanation: Light blue nodes are lonely nodes.
Please remember that order doesn't matter, [2,6] is also an acceptable answer.
```

**Example 3:**![](https://assets.leetcode.com/uploads/2020/06/03/tree.png)

```text

Input: root = [11,99,88,77,null,null,66,55,null,null,44,33,null,null,22]
Output: [77,55,33,66,44,22]
Explanation: Nodes 99 and 88 share the same parent. Node 11 is the root.
All other nodes are lonely.
```

**Example 4:**

```text
Input: root = [197]
Output: []
```

**Example 5:**

```text
Input: root = [31,null,78,null,28]
Output: [78,28]
```

**Constraints:**

* The number of nodes in the `tree` is in the range `[1, 1000].`
* Each node's value is between `[1, 10^6]`.

## **思路**

每个节点不知道自己是不是lonely的，只有父节点知道。

但是子节点知道自己是不是空的，通过返回自己是不是空让父节点考虑是否有lonely节点。

```java
class Solution {
    
    public List<Integer> getLonelyNodes(TreeNode root) {
        List<Integer> res = new ArrayList<>();
        getLonelyNodes(root, res);
        return res;
    }
    
    boolean getLonelyNodes(TreeNode root, List<Integer> lonelyNodes) {
        if(root == null) 
            return false;
        
        boolean left = getLonelyNodes(root.left, lonelyNodes);
        boolean right = getLonelyNodes(root.right, lonelyNodes);
        
        if(left && !right)
            lonelyNodes.add(root.left.val);
        
        if(!left && right)
            lonelyNodes.add(root.right.val);
        
        return true;
    }
}
```

