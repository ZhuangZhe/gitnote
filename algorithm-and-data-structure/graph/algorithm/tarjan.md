# Tarjan算法

用户找到图中的强连通分量

强连通图指每一个顶点皆可以经由该图上的边抵达其他的每一个点的有向图。

强连通分量则是指一张有向图`G`的极大强连通子图`G'`。

伪代码：

```java
UNVISITED = -1
n = number of nodes in graph
g = adjacency list with directed edges

id = 0				# 用于遍历图并赋予顶点编号
sccCount = 0		# 用于计算强连通分量的个数

# Index i in these arrays represent node i
ids = [0, 0, ... 0, 0]									# 长度为n
low = [0, 0, ... 0, 0]									# 长度为n
onStack = [false, false, ... false, false]				# 长度为n
stack = an empty stack data structure

function findSccs():
	for(i = 0 ; i < n ; i++):
		ids[i] = UNVISITED
	for(i = 0 ; i < n ; i++):
		if(ids[i] == UNVISITED):
			dfs(i)
	return low

function dfs(at):
	stack.push(at)
	onStack[at] = true
	ids[at] = low[at] = id++
	
	# Visit all neighbours & min low-link on callback
	for(to : g[at]):
		if(ids[to] == UNVISITED):
			dfs(to)
		if(onStack[to]):
			low[at] = min(low[at],low[to])

	# After having visited all the neighbours of 'at'
	# if we're at the start of a SCC empty the seen
	# stack until we're back to the start of the SCC.
	if(ids[at] == low[at]):
		for(node = stack.pop() ;; node = stack.pop()):
			onStack[node] = false
			low[node] = ids[at]
			if(node == at):
				break
		sccCount++
```

