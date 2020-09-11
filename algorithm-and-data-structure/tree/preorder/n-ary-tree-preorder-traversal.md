# N-ary Tree Preorder Traversal

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/n-ary-tree-preorder-traversal/)

## 题干

Given an n-ary tree, return the preorder traversal of its nodes' values.

_Nary-Tree input serialization is represented in their level order traversal, each group of children is separated by the null value \(See examples\)._

**Follow up:**

Recursive solution is trivial, could you do it iteratively?

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/12/narytreeexample.png)

```text
Input: root = [1,null,3,2,4,null,5,6]
Output: [1,3,5,6,2,4]
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/11/08/sample_4_964.png)

```text
Input: root = [1,null,2,3,4,5,null,null,6,7,null,8,null,9,10,null,null,11,null,12,null,13,null,null,14]
Output: [1,2,3,6,7,11,14,4,8,12,5,9,13,10]
```

**Constraints:**

* The height of the n-ary tree is less than or equal to `1000`
* The total number of nodes is between `[0, 10^4]`

## 思路

### 递归

```java
class Solution {
    
    List<Integer> list = new ArrayList<>();
    
    public List<Integer> preorder(Node root) {
        list = new ArrayList<>();
        dfs(root);
        return list;
    }
    
    void dfs(Node root) {
        if(root == null) 
            return;
        
        list.add(root.val);
        
        for(Node child : root.children) {
            dfs(child);
        }
    }
}
```

### 迭代

```java
class Solution {
    public List<Integer> preorder(Node root) {
        List<Integer> res = new ArrayList<>();
        Stack<Node> stack = new Stack<>();
        
        if(root != null)
            stack.push(root);
        
        while(!stack.isEmpty()) {
            Node cur = stack.pop();
            res.add(cur.val);
            for(int i = cur.children.size() - 1; i >= 0; i--) {
                stack.push(cur.children.get(i));
            }
        }
        return res;
    }
}
```

