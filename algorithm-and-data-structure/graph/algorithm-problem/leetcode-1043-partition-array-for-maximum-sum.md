# Partition Array for Maximum Sum

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/partition-array-for-maximum-sum/)\*\*\*\*

## 题干

Given an integer array `A`, you partition the array into \(contiguous\) subarrays of length at most `K`.  After partitioning, each subarray has its value changed to become the maximum value of that subarray.

Return the largest sum of the given array after partitioning.

**Example 1:**

```text
Input: A = [1,15,7,9,2,5,10], K = 3
Output: 84
Explanation: A becomes [15,15,15,9,10,10,10]
```

**Note:**

1. `1 <= K <= A.length <= 500`
2. `0 <= A[i] <= 10^6`

## 思路

### **Stage 1**

1. 使用DP方法，从左向右遍历，每当遍历到一个新的数值的时候，用一个window来搜索近k个数字的最大值，并用DP数组存当使用当前window最大值存数值覆盖window中所有值时的和。
2. 注意，当前点是无法知道后面是有更大的值的，所以如果当前数值是子数组边界，就可以存之前这段子数组的最大和。当遍历到后边的时候，可能会讲这个值覆盖为更大的值，但是已经影响不到之前的和运算了。

```text
input: A = [1, 15, 7, 9, 2, 5, 10], k = 3;
DP = [0, 0, 0, 0, 0, 0, 0];

[1, 0, 0, 0, 0, 0, 0]
[1, 30, 0, 0, 0, 0, 0]
[1, 30, 45, 0, 0, 0, 0]
[1, 30, 45, 45+9=54, 0, 0, 0]
    9+45(window=1) vs 18+30(window=2) vs 45+1(window=3)
[1, 30, 45, 54, 27+30=57, 0, 0]
    2+54(window=1) vs 18+45(window=2) vs 27+30(window=3)
[1, 30, 45, 54, 57, 27+45=72, 0]
    5+57(window=1) vs 10+54(window=2) vs 27+45(window=3)
[1, 30, 45, 54, 57, 72, 30+54=84]
    10+72(window=1) vs 20+57(window=2) vs 30+54(window=3)
    
output: 84
```

```java
class Solution {
    public int maxSumAfterPartitioning(int[] A, int K) {
        int len = A.length;
        int[] dp = new int[len];
        
        for(int i = 0 ; i < len ; i++) {
            int curMax = A[i];
            int windowSize = 1;
            while(windowSize <= K && i - windowSize + 1 >= 0) {
                curMax = Math.max(curMax, A[i - windowSize + 1]);
                dp[i] = Math.max(dp[i], (i >= windowSize ? dp[i - windowSize] : 0) + windowSize * curMax);
                windowSize++;
            }
        }
        
        return dp[len - 1];
    }
}
```







