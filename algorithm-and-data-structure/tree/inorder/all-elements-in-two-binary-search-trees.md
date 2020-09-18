# All Elements in Two Binary Search Trees

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/all-elements-in-two-binary-search-trees/)

## 题干

Given two binary search trees `root1` and `root2`.

Return a list containing _all the integers_ from _both trees_ sorted in **ascending** order.

**Example 1:**![](https://assets.leetcode.com/uploads/2019/12/18/q2-e1.png)

```text
Input: root1 = [2,1,4], root2 = [1,0,3]
Output: [0,1,1,2,3,4]
```

**Example 2:**

```text
Input: root1 = [0,-10,10], root2 = [5,1,7,0,2]
Output: [-10,0,0,1,2,5,7,10]
```

**Example 3:**

```text
Input: root1 = [], root2 = [5,1,7,0,2]
Output: [0,1,2,5,7]
```

**Example 4:**

```text
Input: root1 = [0,-10,10], root2 = []
Output: [-10,0,10]
```

**Example 5:**![](https://assets.leetcode.com/uploads/2019/12/18/q2-e5-.png)

```text
Input: root1 = [1,null,8], root2 = [8,1]
Output: [1,1,8,8]
```

**Constraints:**

* Each tree has at most `5000` nodes.
* Each node's value is between `[-10^5, 10^5]`.

## 思路

### 方法一

先将两棵树这中序找出来，再做一个merge。

```java
class Solution {
    public List<Integer> getAllElements(TreeNode root1, TreeNode root2) {
        List<Integer> e1 = getAllElements(root1);
        List<Integer> e2 = getAllElements(root2);
        List<Integer> res = new ArrayList<>();
        
        int index1 = 0;
        int index2 = 0;
        
        while(index1 < e1.size() && index2 < e2.size()) {
            if(e1.get(index1) < e2.get(index2)) {
                res.add(e1.get(index1));
                index1++;
            } else {
                res.add(e2.get(index2));
                index2++;
            }
        }
        
        while(index1 < e1.size()) {
            res.add(e1.get(index1));
            index1++;
        }
        while(index2 < e2.size()) {
            res.add(e2.get(index2));
            index2++;
        }
        
        return res;
    }
    
    List<Integer> getAllElements(TreeNode root) {
        List<Integer> elements = new ArrayList<>();
        
        Stack<TreeNode> stack = new Stack<>();
        TreeNode cur = root;
        
        while(cur != null || !stack.isEmpty()) {
            if(cur != null) {
                stack.push(cur);
                cur = cur.left;
            } else {
                cur = stack.pop();
                elements.add(cur.val);
                cur = cur.right;
            }
        }
        
        return elements;
    }
}
```

### 方法二

使用迭代器进行merge。



