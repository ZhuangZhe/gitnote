# Evaluate Division

\*\*\*\*[**LeetCode链接**](https://leetcode.com/problems/evaluate-division/)

## 题干

Equations are given in the format `A / B = k`, where `A` and `B` are variables represented as strings, and `k` is a real number \(floating point number\). Given some queries, return the answers. If the answer does not exist, return `-1.0`.

**Example:**  
Given `a / b = 2.0, b / c = 3.0.`  
queries are: `a / c = ?, b / a = ?, a / e = ?, a / a = ?, x / x = ? .`  
return `[6.0, 0.5, -1.0, 1.0, -1.0 ].`

The input is: `vector<pair<string, string>> equations, vector<double>& values, vector<pair<string, string>> queries` , where `equations.size() == values.size()`, and the values are positive. This represents the equations. Return `vector<double>`.

According to the example above:

```text
equations = [ ["a", "b"], ["b", "c"] ],
values = [2.0, 3.0],
queries = [ ["a", "c"], ["b", "a"], ["a", "e"], ["a", "a"], ["x", "x"] ]. 
```

The input is always valid. You may assume that evaluating the queries will result in no division by zero and there is no contradiction.

## 思路

### stage 1

1. 等式中的数值关系可以看成是一个树，最小的值是根，其他值都是根的几倍。
2. 使用union-find来构建树，同时使用一个map来存数值与根数值倍数关系，在建树的过程中，这些倍数关系都可能会被改写。
3. 两棵树的合并是一个难点，先存下两个数值对各自根数值的倍数，然后两个根进行合并，根小的根会最为新的根。

```java
class Solution {
    
    Map<String, String> parents = new HashMap();
    Map<String, Double> valueMap = new HashMap();
    
    public double[] calcEquation(List<List<String>> equations, double[] values, List<List<String>> queries) {
        Set<String> zeros = new HashSet<>();
        for(int i = 0 ; i < equations.size() ; i++){
            if(values[i] == 0){
                zeros.add(equations.get(i).get(0));  
                continue;
            }
            union(equations.get(i).get(0), equations.get(i).get(1), values[i]);
        }
        
        List<Double> res = new LinkedList<>();
        for(List<String> q : queries){
            if(!parents.containsKey(q.get(0))||!parents.containsKey(q.get(1))) 
                res.add(-1.0);
            else if(zeros.contains(q.get(0)))
                res.add(0.0);
            else if(zeros.contains(q.get(1)))
                res.add(-1.0);
            else if(!find(q.get(0)).equals(find(q.get(1)))) 
                res.add(-1.0);
            else
                res.add(valueMap.get(q.get(0)) / valueMap.get(q.get(1)));
        }
        
        double[] resArray = new double[res.size()];
        for(int i = 0 ; i < res.size() ; i++) 
            resArray[i] = res.get(i);
        return resArray;
    }
    
    private void union(String s1, String s2, double val){ 
        String p1 = find(s1);
        String p2 = find(s2);
        if(p1.compareTo(p2) > 0){
            parents.put(p1,p2);
            valueMap.put(p1, (valueMap.get(s2) / valueMap.get(s1)) * val);
        }else{
            parents.put(p2,p1);
            valueMap.put(p2, (valueMap.get(s1) / valueMap.get(s2)) / val);
        }
    }
    
    private String find(String s){
        if(parents.containsKey(s) && !parents.get(s).equals(s)){
            String p = parents.get(s);
            parents.put(s , find(p));
            valueMap.put(s , valueMap.getOrDefault(s, 1.0) * valueMap.get(p));
        }else{
            parents.put(s , s);
            valueMap.put(s , 1.0);
        }
        return parents.get(s);
    }
    
}
```

