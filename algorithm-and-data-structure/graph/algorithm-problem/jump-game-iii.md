# Jump Game III

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/jump-game-iii/)\*\*\*\*

## 题干

Given an array of non-negative integers `arr`, you are initially positioned at `start` index of the array. When you are at index `i`, you can jump to `i + arr[i]` or `i - arr[i]`, check if you can reach to **any** index with value 0.

Notice that you can not jump outside of the array at any time.

**Example 1:**

```text
Input: arr = [4,2,3,0,3,1,2], start = 5
Output: true
Explanation: 
All possible ways to reach at index 3 with value 0 are: 
index 5 -> index 4 -> index 1 -> index 3 
index 5 -> index 6 -> index 4 -> index 1 -> index 3 
```

**Example 2:**

```text
Input: arr = [4,2,3,0,3,1,2], start = 0
Output: true 
Explanation: 
One possible way to reach at index 3 with value 0 is: 
index 0 -> index 4 -> index 1 -> index 3
```

**Example 3:**

```text
Input: arr = [3,0,2,1,2], start = 2
Output: false
Explanation: There is no way to reach at index 1 with value 0.
```

**Constraints:**

* `1 <= arr.length <= 5 * 10^4`
* `0 <= arr[i] < arr.length`
* `0 <= start < arr.length`

## 思路

### stage 1

1. 使用BFS从start遍历数组
2. 使用一个queue记录下可以去到的位置，用于之后访问
3. 同时使用set记录下已经访问过位置，避免重复访问
4. 遍历知道遇到0或者queue为空时结束

```java
class Solution {
    public boolean canReach(int[] arr, int start) {
        int len = arr.length;
        if (start >= len)
            return false;
        Queue<Integer> queue = new LinkedList<>();
        queue.offer(start);
        boolean[] isVisited = new boolean[len];
        isVisited[start] = true;
        while (!queue.isEmpty()) {
            int cur = queue.poll();
            if (arr[cur] == 0)
                return true;
            int left = cur - arr[cur];
            int right = cur + arr[cur];
            if (left >= 0 && !isVisited[left]) {
                queue.offer(left);
                isVisited[left] = true;
            }
            if (right < len && !isVisited[right]) {
                queue.offer(right);
                isVisited[right] = true;
            }
        }
        return false;
    }
}
```





