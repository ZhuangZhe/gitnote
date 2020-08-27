# Keys and Rooms

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/keys-and-rooms/)\*\*\*\*

## 题干

There are `N` rooms and you start in room `0`.  Each room has a distinct number in `0, 1, 2, ..., N-1`, and each room may have some keys to access the next room. 

Formally, each room `i` has a list of keys `rooms[i]`, and each key `rooms[i][j]` is an integer in `[0, 1, ..., N-1]` where `N = rooms.length`.  A key `rooms[i][j] = v` opens the room with number `v`.

Initially, all the rooms start locked \(except for room `0`\). 

You can walk back and forth between rooms freely.

Return `true` if and only if you can enter every room.

**Example 1:**

```text
Input: [[1],[2],[3],[]]
Output: true
Explanation:  
We start in room 0, and pick up key 1.
We then go to room 1, and pick up key 2.
We then go to room 2, and pick up key 3.
We then go to room 3.  Since we were able to go to every room, we return true.
```

**Example 2:**

```text
Input: [[1,3],[3,0,1],[2],[0]]
Output: false
Explanation: We can't enter the room with number 2.
```

**Note:**

1. `1 <= rooms.length <= 1000`
2. `0 <= rooms[i].length <= 1000`
3. The number of keys in all rooms combined is at most `3000`.

## 思路

stage 1

1.  从房间0开始出发，每到一个新的房间都会拿到一串要是，对应着一些房间，我们继续打开这些房间的门，获得更多的钥匙。
2. 使用一个set用于储存所有去过的房间
3. 使用linkedlist存所即将要打开的门

```java
class Solution {
    public boolean canVisitAllRooms(List<List<Integer>> rooms) {
        int numOfRooms = rooms.size();
        Set<Integer> hasKey = new HashSet<>();
        Queue<Integer> keys = new LinkedList<>();
        keys.offer(0);
        hasKey.add(0);
        while(!keys.isEmpty()) {
            int cur = keys.poll();
            for(int curKey : rooms.get(cur)) {
                if(!hasKey.contains(curKey)) {
                    keys.offer(curKey);
                    hasKey.add(curKey);
                }
            }
        }
        
        return hasKey.size() == numOfRooms ? true : false;
    }
}
```



