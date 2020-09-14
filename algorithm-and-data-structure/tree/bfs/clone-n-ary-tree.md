# Clone N-ary Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/clone-n-ary-tree/)

## 题干

Given a `root` of an N-ary tree, return a [**deep copy**](https://en.wikipedia.org/wiki/Object_copying#Deep_copy) \(clone\) of the tree.

Each node in the n-ary tree contains a val \(`int`\) and a list \(`List[Node]`\) of its children.

```text
class Node {
    public int val;
    public List<Node> children;
}
```

_Nary-Tree input serialization is represented in their level order traversal, each group of children is separated by the null value \(See examples\)._

**Follow up:** Can your solution work for the [graph problem](https://leetcode.com/problems/clone-graph/)?

**Example 1:**

![](https://assets.leetcode.com/uploads/2018/10/12/narytreeexample.png)

```text
Input: root = [1,null,3,2,4,null,5,6]
Output: [1,null,3,2,4,null,5,6]
```

**Example 2:**

![](https://assets.leetcode.com/uploads/2019/11/08/sample_4_964.png)

```text
Input: root = [1,null,2,3,4,5,null,null,6,7,null,8,null,9,10,null,null,11,null,12,null,13,null,null,14]
Output: [1,null,2,3,4,5,null,null,6,7,null,8,null,9,10,null,null,11,null,12,null,13,null,null,14]
```

**Constraints:**

* The depth of the n-ary tree is less than or equal to `1000`.
* The total number of nodes is between `[0, 10^4]`.

## 思路

使用两个队列分别用于存原始节点和复制节点，同步他们插入和弹出，每次弹出都否复制当前原始节点的的子节点，将子节点再插入两个队列中。

```java
class Solution {
    public Node cloneTree(Node root) {
        if(root == null)
            return null;
        
        Queue<Node> origin = new LinkedList<>();
        Queue<Node> copy = new LinkedList<>();
        Node rootCopy = new Node(root.val);
        
        origin.offer(root);
        copy.offer(rootCopy);
        
        while(!origin.isEmpty()) {
            Node curOrigin = origin.poll();
            Node curCopy = copy.poll();
            
            for(Node child : curOrigin.children) {
                Node childCopy = new Node(child.val);
                curCopy.children.add(childCopy);
                origin.offer(child);
                copy.offer(childCopy);
            }
        }
        
        return rootCopy;
    }
}
```





