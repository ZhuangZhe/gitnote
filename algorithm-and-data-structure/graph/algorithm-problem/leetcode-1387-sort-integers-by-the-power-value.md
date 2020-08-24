# LeetCode 1387 Sort Integers by The Power Value

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/sort-integers-by-the-power-value/)\*\*\*\*

## 题干

The power of an integer `x` is defined as the number of steps needed to transform `x` into `1` using the following steps:

* if `x` is even then `x = x / 2`
* if `x` is odd then `x = 3 * x + 1`

For example, the power of x = 3 is 7 because 3 needs 7 steps to become 1 \(3 --&gt; 10 --&gt; 5 --&gt; 16 --&gt; 8 --&gt; 4 --&gt; 2 --&gt; 1\).

Given three integers `lo`, `hi` and `k`. The task is to sort all integers in the interval `[lo, hi]` by the power value in **ascending order**, if two or more integers have **the same** power value sort them by **ascending order**.

Return the `k-th` integer in the range `[lo, hi]` sorted by the power value.

Notice that for any integer `x` `(lo <= x <= hi)` it is **guaranteed** that `x` will transform into `1` using these steps and that the power of `x` is will **fit** in 32 bit signed integer.

**Example 1:**

```text
Input: lo = 12, hi = 15, k = 2
Output: 13
Explanation: The power of 12 is 9 (12 --> 6 --> 3 --> 10 --> 5 --> 16 --> 8 --> 4 --> 2 --> 1)
The power of 13 is 9
The power of 14 is 17
The power of 15 is 17
The interval sorted by the power value [12,13,14,15]. For k = 2 answer is the second element which is 13.
Notice that 12 and 13 have the same power value and we sorted them in ascending order. Same for 14 and 15.
```

**Example 2:**

```text
Input: lo = 1, hi = 1, k = 1
Output: 1
```

**Example 3:**

```text
Input: lo = 7, hi = 11, k = 4
Output: 7
Explanation: The power array corresponding to the interval [7, 8, 9, 10, 11] is [16, 3, 19, 6, 14].
The interval sorted by power is [8, 10, 11, 7, 9].
The fourth number in the sorted array is 7.
```

**Example 4:**

```text
Input: lo = 10, hi = 20, k = 5
Output: 13
```

**Example 5:**

```text
Input: lo = 1, hi = 1000, k = 777
Output: 570
```

**Constraints:**

* `1 <= lo <= hi <= 1000`
* `1 <= k <= hi - lo + 1`

## 思路

每个计算除出每个数值的power值，再进行排序

### stage 1

1. 创建一个一维数组`nums`存所有需要排序的数值。
2. 写一个函数`getPower(int num)`计算每个数组的power值
3. 写一个比较器用于对`nums`排序进行，其中调用`getPower`函数获取power值
4. `nums`排序后，返回第`k`个数

#### 不足

1. 排序时多次进行power值计算，做了大量重复工作
2. 比较器中需要使用包裹类型，增加了代码的复杂程度

### Stage 2

1. 该用二维数组，规避上述的放不足2，以及大大改善不足1

```java
class Solution {
    
    public int getKth(int lo, int hi, int k) {
        int len = hi - lo + 1;
        int[][] powers = new int[len][2];
        
        for(int i = 0 ; i < len ; i++) {
            powers[i][0] = lo + i;
            powers[i][1] = getPowerValue(lo + i);
        }
        
        Arrays.sort(powers, new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                if(m[1] == n[1])
                    return m[0] - n[0];
                return m[1] - n[1];
            }
        });
        
        return powers[k - 1][0];
    }
    
    public int getPowerValue(int num) {
        int power = 0;
        
        while(num != 1) {
            if(num % 2 == 0) {
                num /= 2;
            } else {
                num = 3 * num + 1;
            }
            power++;
        }
        
        return power;
    }
}
```

#### 不足

1. 不同数值的power值计算中会出现重叠的部分
2. power计算中可能存在重复的的中间值，形成循环

### Stage 3

1. 使用Map将已经计算过power值的数值和中间值的对应power值都记下来
2. 在`getPower`函数中添加一个set用于存已经见过的数

```java
class Solution {
    
    Map<Integer, Integer> cache = new HashMap<>();
    
    public int getKth(int lo, int hi, int k) {
        int len = hi - lo + 1;
        int[][] powers = new int[len][2];
        
        for(int i = 0 ; i < len ; i++) {
            powers[i][0] = lo + i;
            powers[i][1] = getPowerValue(lo + i);
        }
        
        Arrays.sort(powers, new Comparator<int[]>() {
            public int compare(int[] m, int[] n) {
                if(m[1] == n[1])
                    return m[0] - n[0];
                return m[1] - n[1];
            }
        });
        
        for(int[] power : powers)
            System.out.println(power[0] + " - " + power[1]);
        
        return powers[k - 1][0];
    }
    
    public int getPowerValue(int num) {
        if(num == 1)
            return 0;
        
        if(cache.containsKey(num)) 
            return cache.get(num);
        
        int power = 1;
        
        if(num % 2 == 0) {
            power += getPowerValue(num / 2);
        } else {
            power += getPowerValue(num * 3 + 1);
        }
        
        cache.put(num, power);
        
        return power;
    }
}
```

**使用了recursion，使得消耗的时间和内存大大增加。**



