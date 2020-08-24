# 图

## 图论基本知识

* 顶点
  * vertex - V
* 边
  * edge - E
  * 代表顶点之间的连接关系
  * 表示方式：`<u, v>`
    * `u`和`v`为边两端的顶点
  * 边分为又向的和无向的
    * 有向边：规定了方向的边
      * 与一个节点相关联的边有出边和入边之分
      * 与一个有向边关联的两个点也有始点和终点之分
* 图
  * graph - G
  * 顶点集：储存顶点的有限非空集合
    * `V = {a, b, c…}`
  * 边集：储存边的有限集合
    * `E = {e1, e2, e3…}`
  * 表示方式：`G = (V, E)`
  * 有向图
    * 每条边规定了一个方向
* 度
  * 一个顶点的度是指与该顶点相关联的总边数，顶点`v`的度记作`d(v)`
  * 出度：一个顶点出边的条数，即以该顶点为终点的边的数目
  * 入度：一个顶点入边的条数，即以该顶点为起点的边的数目
* 环
  * 若一条边的两个顶点相同，则此边称作环
* 桥
  * 若去掉一条边，便会使得整个图不连通，该边称为桥
* 路径
  * 指一个从起点到终点经过的顶点序列
  * 如果起点也是终点，称该路径是闭的，反之为开的
  * 如果路径`v_1, ... , v_k`两两不等，则称之为简单路径，但是起点就是终点是允许的
* 行迹
  * 如果路径`P(u,v)`中边各不相同，则该路径称为`u`到`v`的一条行迹。
* 轨道
  * 即简单路径
  * 闭的行迹称作回路\(Circuit\)
  * 闭的轨道称作圈\(Cycle\)

### 图的类型

#### 补图

* 一个图G的补图是一个图有着跟G相同的点，而且这些点之间有边相连当且仅当在G里面他们没有边相连。在制作图的时候，你可以先建立一个有G所有点的完全图，然后清除G里面已经有的边来得到补图。
* 这里的补图并不是图本身的补集，因为只有边的部分合乎补集的概念。

#### 树状图

* 一种无向图
* 如果一个无向简单图G 满足以下相互等价的条件之一，那么G是一棵树：
  * G是没有回路的连通图。
  * G没有回路，但是在G内添加任意一条边，就会形成一个回路。
  * G是连通的，但是如果去掉任意一条边，就不再连通。
  * G内的任意两个顶点能被唯一路径所连通。

#### 连通图

* 在一个无向图G中，若从顶点`v_i`到顶点`v_j`有路径相连，则称`v_i`和`v_j`是连通的。如果G是有向图，那么连接`v_i`和v\_j的路径中所有的边都必须同向。如果图中任意两点都是连通的，那么图被称作连通图。
* 连通分量：无向图G的一个极大连通子图称为G的一个连通分量。连通图只有一个连通分量，即其自身；非连通的无向图有多个连通分量。
* 强连通图：有向图`G = (V, E)`中，若对于V中任意两个不同的顶点`x`和`y`，都存在从`x`到`y`以及从`y`到`x`的路径，则称G是强连通图。相应地有强连通分量的概念。强连通图只有一个强连通分量，即是其自身；非强连通的有向图有多个强连通分量。
* 初级通路：通路中所有的顶点互不相同。初级通路必为简单通路，但反之不真。

#### 有向无环图

* 如果一个有向图从任意顶点出发无法经过若干条边回到该点，则这个图是一个有向无环图
* 有向图中一个点经过两种路线到达另一个点未必形成环，因此有向无环图未必能转化成树，但任何有向树均为有向无环图

#### 完全图

* 每对顶点之间都恰连有一条边的简单图

#### 二分图

* 二分图的顶点可以分成两个互斥的独立集`U`和`V`的图，使得所有边都是连结一个`U`中的点和一个`V`中的点。

#### 正则图

* 每个顶点都有相同数目的邻居的图，即每个顶点的度相同。若每个顶点的度均为`k`，称为k-正则图。
