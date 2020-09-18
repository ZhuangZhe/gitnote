# Maximum Binary Tree

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/maximum-binary-tree/)

## 题干

Given an integer array with no duplicates. A maximum tree building on this array is defined as follow:

1. The root is the maximum number in the array.
2. The left subtree is the maximum tree constructed from left part subarray divided by the maximum number.
3. The right subtree is the maximum tree constructed from right part subarray divided by the maximum number.

Construct the maximum tree by the given array and output the root node of this tree.

**Example 1:**  


```text
Input: [3,2,1,6,0,5]
Output: return the tree root node representing the following tree:

      6
    /   \
   3     5
    \    / 
     2  0   
       \
        1
```

**Note:**  


1. The size of the given array will be in the range \[1,1000\].

## 思路

重载函数，在参数列表中添加两个表示左右指针边际。函数中找到指针范围内的最大值并以此值建立一个节点，同时将数组切分成两个子数组，对两个子数组进行递归调用，并将递归结果链接到相应的左右节点。左边际大于右边际是，返回null。

```java
class Solution {

    public TreeNode constructMaximumBinaryTree(int[] nums) {
        return constructMaximumBinaryTree(nums, 0, nums.length - 1);
    }
    
    TreeNode constructMaximumBinaryTree(int[] nums, int left, int right) {
        if(left > right)
            return null;
        
        int index = left;
        for(int i = left ; i <= right ; i++) {
            if(nums[index] < nums[i])
                index = i;
        }
        
        TreeNode cur = new TreeNode(nums[index]);
        cur.left = constructMaximumBinaryTree(nums, left, index - 1);
        cur.right = constructMaximumBinaryTree(nums, index + 1, right);
        
        return cur;
    }
    
}
```

