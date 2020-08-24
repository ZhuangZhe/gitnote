# Prim算法

用于寻找最小生成树‌

从单一顶点开始，普里姆算法按照以下步骤逐步扩大树中所含顶点的数目，直到遍及连通图的所有顶点。

1. 输入：一个加权连通图，其中顶点集合为`V`，边集合为`E`；
2. 初始化：`V_new = {x}`，其中`x`为集合`V`中的任一节点\(起始点\)，`E_new = {}`
3. 重复下列操作，直到`V_new = V`：
   1. 在集合`E`中选取权值最小的边`(u,v)`，其中`u`为集合`V_new`中的元素，而`v`则是`V`中没有加入`V_new`的顶点\(如果存在有多条满足前述条件即具有相同权值的边，则可任意选取其中之一\)；
   2. 将`v`加入集合`V_new`中，将`(u,v)`加入集合`E_new`中；
4. 输出：使用集合`V_new`和`E_new`来描述所得到的最小生成树。

伪代码：

```java
n	# 图中顶点的数量
pq	# PriorityQueue，用于存边与相应的值
g	# 用于存储每个顶点相连的边
visited = [false, ... false]

function lazyPrims(s = 0):
	m = n - 1 # 最小生成树边的数量
	edgeCount = 0
	mstCost = 0
	mstEdges = [null, null, ... null, null]	# 长度为m
	addEdges(s)

	while (!pq.isEmpty() and edgeCount != m):
		edge = pq.dequeue()
		nodeIndex = edge.to

		if visited[nodeIndex]:
			continue

		mstEdges[edgeCount++] = edge
		mstCost += edge.cost

		addEdges(nodeIndex)
	
	if edgeCount != m:
		return (null, null)	# 不存在最小生成树
	return (mstCost, mstEdges)

function addEdges(nodeIndex):
	visited[nodeIndex] = true
	edges = g[nodeIndex]
	for(edge : edges):
		if !isited[edge.to]:
			pq.enqueue(edge)
```

