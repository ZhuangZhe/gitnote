# Kruskal算法

用于寻找最小生成树

步骤：

1. 新建图`G`，`G`中拥有原图中相同的节点，但没有边
2. 将原图中所有的边按权值从小到大排序
3. 从权值最小的边开始，如果这条边连接的两个节点于图`G`中不在同一个连通分量中，则添加这条边到图`G`中
4. 重复3，直至图`G`中所有的节点都在同一个连通分量中

伪代码：

```java
procedure kruskal(G,w)
Input: A connected undirected graph G=(V,E) with edge weights w_e.
Output: A minimum spanning trr defind by the edges X.

for add u ∈ V：
	makeset(u)

X = {}
Sort the edges E by weight
for all edges {u,v} ∈ E, in increasing order of weight:
	if find(u) ≠ find(v):
		add edge {u,v} to X
		union(u,v)

procedure makeset(x)
π(x) = x
rank(x) = 0

procedure union(x,y)
r_x = find(x)
r_y = find(y)
if r_x = r_y:
	return
if rank(r_x) > rank(r_y):
	π(r_y) = r_x
else:
	π(r_x) = r_y
	if rank(r_x) == rank(r_y):
		rank(r_y) = rank(r_y) + 1

procedure find(x):
	while x ≠ π(x):
		x = π(x)
	return x
```

