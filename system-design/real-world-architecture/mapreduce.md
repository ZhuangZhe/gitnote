# MapReduce

## 程序设计模型

计算将会以一个键值对集为输入，并且产生一个新键值对集为输出。MapReduce库将计算表示为两个功能：**Map**和**Reduce**。

* **Map：**由用户实现，将输入的键值对集处理成中间量键值对集，MapReduce库将中间量键值对集中具有相同键的键值对都捆绑在一起，将这些捆绑好的数据传给Reduce。
* **Reduce：**由用户实现，从中间量数据中获取键以及每个键对应的值集合。使用Reduce操作将每个键对应这些值转换为一个或零个值。中间量值集合通过迭代器提供数据给reducer，这样让我们可以往内存里塞大量数据。

## 例子









**参考资料：**

* \*\*\*\*[**MapReduce: Simplified Data Processing on Large Clusters**](http://static.googleusercontent.com/media/research.google.com/zh-CN/us/archive/mapreduce-osdi04.pdf)\*\*\*\*

\*\*\*\*

