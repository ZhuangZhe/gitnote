# Dijkstra算法

用于找到一个点到其他所有点的最短路径

伪代码：

```java
g = adjacency list of weighted graph
n = the number og nodes in the graph
s = the index of the starting node (0 <= s < n)

function dijkstra(g,n,s):
	vis = [false, false, ... false, false]	# 长度为n
	dist = [∞, ∞, ... ∞, ∞]	# 长度为n
	dist[s] = 0
	ipq = empty indexed priority queue
	ipq.insert(s,0)
	
	while ipq.size() != 0:
		index, minValue = ipq.poll()
		vis[index] = true
		
		if dist[index] < minValue:
			continue
			
		for(edge : g[index]):
		
			if vis[edge.to]:
				continue
				
			newDist = dist[index] + edge.cost
			
			if newDist < dist[edge.to]:
				dist[edge.to] = newDist
				
				if !ipq.contains(edge.to):
					ipq.insert(edge.to, newDist)
				else:
					ipq.decreaseKey(edge.to, newDist)
					
	return dist
```

