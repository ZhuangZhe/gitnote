# Clone Binary Tree With Random Pointer

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/clone-binary-tree-with-random-pointer/)

## **题干**

A binary tree is given such that each node contains an additional random pointer which could point to any node in the tree or null.

Return a [**deep copy**](https://en.wikipedia.org/wiki/Object_copying#Deep_copy) of the tree.

The tree is represented in the same input/output way as normal binary trees where each node is represented as a pair of `[val, random_index]` where:

* `val`: an integer representing `Node.val`
* `random_index`: the index of the node \(in the input\) where the random pointer points to, or `null` if it does not point to any node.

You will be given the tree in class `Node` and you should return the cloned tree in class `NodeCopy`. `NodeCopy` class is just a clone of `Node` class with the same attributes and constructors.

**Example 1:**

![](https://assets.leetcode.com/uploads/2020/06/17/clone_1.png)

```text
Input: root = [[1,null],null,[4,3],[7,0]]
Output: [[1,null],null,[4,3],[7,0]]
Explanation: The original binary tree is [1,null,4,7].
The random pointer of node one is null, so it is represented as [1, null].
The random pointer of node 4 is node 7, so it is represented as [4, 3] where 3 is the index of node 7 in the array representing the tree.
The random pointer of node 7 is node 1, so it is represented as [7, 0] where 0 is the index of node 1 in the array representing the tree.
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2020/06/17/clone_2.png)

```text
Input: root = [[1,4],null,[1,0],null,[1,5],[1,5]]
Output: [[1,4],null,[1,0],null,[1,5],[1,5]]
Explanation: The random pointer of a node can be the node itself.
```

**Example 3:**

![](https://assets.leetcode.com/uploads/2020/06/17/clone_3.png)

```text
Input: root = [[1,6],[2,5],[3,4],[4,3],[5,2],[6,1],[7,0]]
Output: [[1,6],[2,5],[3,4],[4,3],[5,2],[6,1],[7,0]]
```

**Example 4:**

```text
Input: root = []
Output: []
```

**Example 5:**

```text
Input: root = [[1,null],null,[2,null],null,[1,null]]
Output: [[1,null],null,[2,null],null,[1,null]]
```

**Constraints:**

* The number of nodes in the `tree` is in the range `[0, 1000].`
* Each node's value is between `[1, 10^6]`.

## **思路**

```java
class Solution {

    public NodeCopy copyRandomBinaryTree(Node root) {
        //return iterative(root);
        
        return recursive(root);
    }
    
    public NodeCopy recursive(Node root) {
        Map<Node, NodeCopy> map = new HashMap<>();
        return dfs(root, map);
    }
    
    public NodeCopy dfs(Node root, Map<Node,NodeCopy> map){
        if (root==null) 
            return null;        
        
        if (map.containsKey(root)) 
            return map.get(root);
        
        NodeCopy newNode = new NodeCopy(root.val);
        map.put(root,newNode);
        
        newNode.left=dfs(root.left,map);
        newNode.right=dfs(root.right,map);
        newNode.random=dfs(root.random,map);
        
        return newNode;
        
    }
    
    public NodeCopy iterative(Node root) {
        if(root == null)
            return null;
        
        Map<Node, NodeCopy> originToCopy = new HashMap<>();
        NodeCopy copyRoot = new NodeCopy(root.val);
        originToCopy.put(root , copyRoot);
        Stack<Node> stack = new Stack<>();
        stack.push(root);
        
        while(!stack.isEmpty()) {
            Node cur = stack.pop();
            Node left = cur.left;
            Node right = cur.right;
            Node random = cur.random;
            
            NodeCopy copy = originToCopy.get(cur);
            if(left != null) {
                stack.push(left);
                if(originToCopy.containsKey(left)) {
                    copy.left = originToCopy.get(left);
                } else {
                    NodeCopy copyLeft = new NodeCopy(left.val);
                    copy.left = copyLeft;
                    originToCopy.put(left, copyLeft);
                }
            }
            
            if(right != null) {
                stack.push(right);
                if(originToCopy.containsKey(right)) {
                    copy.right = originToCopy.get(right);
                } else {
                    NodeCopy copyRight = new NodeCopy(right.val);
                    copy.right = copyRight;
                    originToCopy.put(right, copyRight);
                }
            }
            
            if(random != null) {
                if(originToCopy.containsKey(random)) {
                    copy.random = originToCopy.get(random);
                } else {
                    NodeCopy copyRandom = new NodeCopy(random.val);
                    copy.random = copyRandom;
                    originToCopy.put(random, copyRandom);
                }
            }
            
        }
        
        return copyRoot;
    }
}
```

\*\*\*\*

\*\*\*\*

\*\*\*\*

