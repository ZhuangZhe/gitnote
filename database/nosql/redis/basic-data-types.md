# 基本数据类型

**Redis** 有5种基础数据结构，它们分别是：**string\(字符串\)**、**list\(列表\)**、**hash\(字典\)**、**set\(集合\)** 和 **zset\(有序集合\)**。

## 字符串string

Redis中的字符串是一种**动态字符串**，这意味着使用者可以修改，它的底层实现有点类似于Java中的**ArrayList**，有一个字符数组，从源码的**sds.h/sdshdr文件** 中可以看到Redis底层对于字符串的定义**SDS**，即_Simple Dynamic String_ 结构：

```c
COPY/* Note: sdshdr5 is never used, we just access the flags byte directly.
 * However is here to document the layout of type 5 SDS strings. */
struct __attribute__ ((__packed__)) sdshdr5 {
    unsigned char flags; /* 3 lsb of type, and 5 msb of string length */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr8 {
    uint8_t len; /* used */
    uint8_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr16 {
    uint16_t len; /* used */
    uint16_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr32 {
    uint32_t len; /* used */
    uint32_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
struct __attribute__ ((__packed__)) sdshdr64 {
    uint64_t len; /* used */
    uint64_t alloc; /* excluding the header and null terminator */
    unsigned char flags; /* 3 lsb of type, 5 unused bits */
    char buf[];
};
```

你会发现同样一组结构Redis使用泛型定义了好多次，**为什么不直接使用int类型呢？**

因为当字符串比较短的时候，len和alloc可以使用byte和short来表示，**Redis为了对内存做极致的优化，不同长度的字符串使用不同的结构体来表示。**

### SDS与C字符串的区别

为什么不考虑直接使用C语言的字符串呢？因为C语言这种简单的字符串表示方式**不符合 Redis 对字符串在安全性、效率以及功能方面的要求**。我们知道，C语言使用了一个长度为N+1的字符数组来表示长度为N的字符串，并且字符数组最后一个元素总是`'\0'`。_\(下图就展示了C语言中值为 “Redis” 的一个字符数组\)_

![](https://cdn.jsdelivr.net/gh/wmyskxz/img/img/Redis%EF%BC%881%EF%BC%89%E2%80%94%E2%80%945%E7%A7%8D%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7896890-3a7d503c81b6e6e4.png)

这样简单的数据结构可能会造成以下一些问题：

* **获取字符串长度为O\(N\)级别的操作** → 因为C不保存数组的长度，每次都需要遍历一遍整个数组；
* 不能很好的杜绝**缓冲区溢出/内存泄漏**的问题 → 跟上述问题原因一样，如果执行拼接或缩短字符串的操作，如果操作不当就很容易造成上述问题；
* C字符串**只能保存文本数据** → 因为C语言中的字符串必须符合某种编码\(比如 ASCII\)，例如中间出现的`'\0'`可能会被判定为提前结束的字符串而识别不了；

我们以追加字符串的操作举例，Redis源码如下：

```c
COPY/* Append the specified binary-safe string pointed by 't' of 'len' bytes to the
 * end of the specified sds string 's'.
 *
 * After the call, the passed sds string is no longer valid and all the
 * references must be substituted with the new pointer returned by the call. */
sds sdscatlen(sds s, const void *t, size_t len) {
    // 获取原字符串的长度
    size_t curlen = sdslen(s);

    // 按需调整空间，如果容量不够容纳追加的内容，就会重新分配字节数组并复制原字符串的内容到新数组中
    s = sdsMakeRoomFor(s,len); 
    if (s == NULL) return NULL;   // 内存不足
    memcpy(s+curlen, t, len);     // 追加目标字符串到字节数组中
    sdssetlen(s, curlen+len);     // 设置追加后的长度
    s[curlen+len] = '\0';         // 让字符串以 \0 结尾，便于调试打印
    return s;
}
```

* **注：Redis规定了字符串的长度不得超过512MB。**

### 对字符串的基本操作

安装好Redis，我们可以使用`redis-cli`来对Redis进行命令行的操作，当然Redis官方也提供了在线的调试器，你也可以在里面敲入命令进行操作：[http://try.redis.io/\#run](http://try.redis.io/#run)

#### **设置和获取键值对**

```bash
COPY> SET key value
OK
> GET key
"value"
```

正如你看到的，我们通常使用`SET`和`GET`来设置和获取字符串值。

值可以是任何种类的字符串\(包括二进制数据\)，例如你可以在一个键下保存一张`.jpeg`图片，只需要注意不要超过512 MB的最大限度就好了。

当 key 存在时，`SET` 命令会覆盖掉你上一次设置的值：

```bash
COPY> SET key newValue
OK
> GET key
"newValue"
```

另外你还可以使用`EXISTS`和`DEL`关键字来查询是否存在和删除键值对：

```bash
COPY> EXISTS key
(integer) 1
> DEL key
(integer) 1
> GET key
(nil)
```

#### **批量设置键值对**

```bash
COPY> SET key1 value1
OK
> SET key2 value2
OK
> MGET key1 key2 key3    # 返回一个列表
1) "value1"
2) "value2"
3) (nil)
> MSET key1 value1 key2 value2
> MGET key1 key2
1) "value1"
2) "value2"
```

#### **过期和SET命令扩展**

可以对key设置过期时间，到时间会被自动删除，这个功能常用来控制缓存的失效时间。_\(过期可以是任意数据结构\)_

```c
COPY> SET key value1
> GET key
"value1"
> EXPIRE name 5    # 5s 后过期
...                # 等待 5s
> GET key
(nil)
```

等价于`SET`+`EXPIRE`的`SETNX`命令：

```bash
COPY> SETNX key value1
...                # 等待 5s 后获取
> GET key
(nil)

> SETNX key value1  # 如果 key 不存在则 SET 成功
(integer) 1
> SETNX key value1  # 如果 key 存在则 SET 失败
(integer) 0
> GET key
"value"             # 没有改变 
```

#### **计数**

如果value是一个整数，还可以对它使用`INCR`命令进行**原子性**的自增操作，这意味着及时多个客户端对同一个key进行操作，也决不会导致竞争的情况：

```bash
COPY> SET counter 100
> INCR count
(interger) 101
> INCRBY counter 50
(integer) 151
```

#### **返回原值的GETSET命令**

对字符串，还有一个`GETSET`比较让人觉得有意思，它的功能跟它名字一样：为key设置一个值并返回原值：

```c
COPY> SET key value
> GETSET key value1
"value"
```

这可以对于某一些需要隔一段时间就统计的key很方便的设置和查看，例如：系统每当由用户进入的时候你就是用`INCR`命令操作一个key，当需要统计时候你就把这个key使用`GETSET`命令重新赋值为0，这样就达到了统计的目的。

## 列表list

Redis的列表相当于Java语言中的**LinkedList**，注意它是链表而不是数组。这意味着list的插入和删除操作非常快，时间复杂度为O\(1\)，但是索引定位很慢，时间复杂度为O\(n\)。

我们可以从源码的`adlist.h/listNode`来看到对其的定义：

```c
COPY/* Node, List, and Iterator are the only data structures used currently. */

typedef struct listNode {
    struct listNode *prev;
    struct listNode *next;
    void *value;
} listNode;

typedef struct listIter {
    listNode *next;
    int direction;
} listIter;

typedef struct list {
    listNode *head;
    listNode *tail;
    void *(*dup)(void *ptr);
    void (*free)(void *ptr);
    int (*match)(void *ptr, void *key);
    unsigned long len;
} list;
```

可以看到，多个listNode可以通过`prev`和`next`指针组成双向链表：

![](https://cdn.jsdelivr.net/gh/wmyskxz/img/img/Redis%EF%BC%881%EF%BC%89%E2%80%94%E2%80%945%E7%A7%8D%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7896890-8f569f06506845c1.png)

虽然仅仅使用多个listNode结构就可以组成链表，但是使用`adlist.h/list`结构来持有链表的话，操作起来会更加方便：

![](https://cdn.jsdelivr.net/gh/wmyskxz/img/img/Redis%EF%BC%881%EF%BC%89%E2%80%94%E2%80%945%E7%A7%8D%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7896890-c6fb10cdbb32f517.png)

### 链表的基本操作

* `LPUSH`和`RPUSH`分别可以向list的左边\(头部\)和右边\(尾部\)添加一个新元素；
* `LRANGE` 命令可以从list中取出一定范围的元素；
* `LINDEX` 命令可以从list中取出指定下表的元素，相当于Java链表操作中的`get(int index)`操作；

示范：

```bash
COPY> rpush mylist A
(integer) 1
> rpush mylist B
(integer) 2
> lpush mylist first
(integer) 3
> lrange mylist 0 -1    # -1 表示倒数第一个元素, 这里表示从第一个元素到最后一个元素，即所有
1) "first"
2) "A"
3) "B"
```

#### **list实现队列**

队列是先进先出的数据结构，常用于消息排队和异步逻辑处理，它会确保元素的访问顺序：

```c
COPY> RPUSH books python java golang
(integer) 3
> LPOP books
"python"
> LPOP books
"java"
> LPOP books
"golang"
> LPOP books
(nil)
```

#### **list实现栈**

栈是先进后出的数据结构，跟队列正好相反：

```c
COPY> RPUSH books python java golang
> RPOP books
"golang"
> RPOP books
"java"
> RPOP books
"python"
> RPOP books
(nil)
```

## 字典hash

Redis中的字典相当于Java中的**HashMap**，内部实现也差不多类似，都是通过 **“数组 + 链表”** 的链地址法来解决部分**哈希冲突**，同时这样的结构也吸收了两种不同数据结构的优点。源码定义如`dict.h/dictht`定义：

```c
COPYtypedef struct dictht {
    // 哈希表数组
    dictEntry **table;
    // 哈希表大小
    unsigned long size;
    // 哈希表大小掩码，用于计算索引值，总是等于 size - 1
    unsigned long sizemask;
    // 该哈希表已有节点的数量
    unsigned long used;
} dictht;

typedef struct dict {
    dictType *type;
    void *privdata;
    // 内部有两个 dictht 结构
    dictht ht[2];
    long rehashidx; /* rehashing not in progress if rehashidx == -1 */
    unsigned long iterators; /* number of iterators currently running */
} dict;
```

`table`属性是一个数组，数组中的每个元素都是一个指向`dict.h/dictEntry`结构的指针，而每个`dictEntry`结构保存着一个键值对：

```c
COPYtypedef struct dictEntry {
    // 键
    void *key;
    // 值
    union {
        void *val;
        uint64_t u64;
        int64_t s64;
        double d;
    } v;
    // 指向下个哈希表节点，形成链表
    struct dictEntry *next;
} dictEntry;
```

可以从上面的源码中看到，**实际上字典结构的内部包含两个hashtable**，通常情况下只有一个hashtable是有值的，但是在字典扩容缩容时，需要分配新的hashtable，然后进行**渐进式搬迁**。

### 渐进式rehash

大字典的扩容是比较耗时间的，需要重新申请新的数组，然后将旧字典所有链表中的元素重新挂接到新的数组下面，这是一个O\(n\)级别的操作，作为单线程的Redis很难承受这样耗时的过程，所以Redis使用**渐进式rehash**小步搬迁：

![](https://cdn.jsdelivr.net/gh/wmyskxz/img/img/Redis%EF%BC%881%EF%BC%89%E2%80%94%E2%80%945%E7%A7%8D%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7896890-325d968300c47100.png)

渐进式rehash会在rehash的同时，保留新旧两个hash结构，如上图所示，查询时会同时查询两个hash结构，然后在后续的定时任务以及hash操作指令中，循序渐进的把旧字典的内容迁移到新字典中。当搬迁完成了，就会使用新的hash结构取而代之。

### 扩缩容的条件

正常情况下，当hash表中**元素的个数等于第一维数组的长度时**，就会开始扩容，扩容的新数组是**原数组大小的2倍**。不过如果Redis正在做`bgsave(持久化命令)`，为了减少内存也得过多分离，Redis尽量不去扩容，但是如果hash表非常满了，**达到了第一维数组长度的5倍了**，这个时候就会**强制扩容**。

当hash表因为元素逐渐被删除变得越来越稀疏时，Redis会对hash表进行缩容来减少hash表的第一维数组空间占用。所用的条件是**元素个数低于数组长度的10%**，缩容不会考虑Redis是否在做`bgsave`。

### 字典的基本操作

hash也有缺点，hash结构的存储消耗要高于单个字符串，所以到底该使用hash还是字符串，需要根据实际情况再三权衡：

```bash
COPY> HSET books java "think in java"    # 命令行的字符串如果包含空格则需要使用引号包裹
(integer) 1
> HSET books python "python cookbook"
(integer) 1
> HGETALL books    # key 和 value 间隔出现
1) "java"
2) "think in java"
3) "python"
4) "python cookbook"
> HGET books java
"think in java"
> HSET books java "head first java"  
(integer) 0        # 因为是更新操作，所以返回 0
> HMSET books java "effetive  java" python "learning python"    # 批量操作
OK
```

## 集合set

Redis的集合相当于Java语言中的**HashSet**，它内部的键值对是无序、唯一的。它的内部实现相当于一个特殊的字典，字典中所有的value都是一个值NULL。

#### 集合set的基本使用 <a id="&#x96C6;&#x5408;-set-&#x7684;&#x57FA;&#x672C;&#x4F7F;&#x7528;"></a>

由于该结构比较简单，我们直接来看看是如何使用的：

```c
COPY> SADD books java
(integer) 1
> SADD books java    # 重复
(integer) 0
> SADD books python golang
(integer) 2
> SMEMBERS books    # 注意顺序，set 是无序的 
1) "java"
2) "python"
3) "golang"
> SISMEMBER books java    # 查询某个 value 是否存在，相当于 contains
(integer) 1
> SCARD books    # 获取长度
(integer) 3
> SPOP books     # 弹出一个
"java"
```

## 有序列表 zset

这可能使Redis最具特色的一个数据结构了，它类似于Java中**SortedSet**和**HashMap**的结合体，一方面它是一个set，保证了内部value的唯一性，另一方面它可以为每个value赋予一个score值，用来代表排序的权重。

它的内部实现用的是一种叫做**「跳跃表」**的数据结构，由于比较复杂，所以在这里简单提一下原理就好了：

![](https://cdn.jsdelivr.net/gh/wmyskxz/img/img/Redis%EF%BC%881%EF%BC%89%E2%80%94%E2%80%945%E7%A7%8D%E5%9F%BA%E6%9C%AC%E6%95%B0%E6%8D%AE%E7%BB%93%E6%9E%84/7896890-efd5114939a651ed.png)

想象你是一家创业公司的老板，刚开始只有几个人，大家都平起平坐。后来随着公司的发展，人数越来越多，团队沟通成本逐渐增加，渐渐地引入了组长制，对团队进行划分，于是有一些人**又是员工又有组长的身份**。

再后来，公司规模进一步扩大，公司需要再进入一个层级：部门。于是每个部门又会从组长中推举一位选出部长。

跳跃表就类似于这样的机制，最下面一层所有的元素都会串起来，都是员工，然后每隔几个元素就会挑选出一个代表，再把这几个代表使用另外一级指针串起来。然后再在这些代表里面挑出二级代表，再串起来。**最终形成了一个金字塔的结构。**

想一下你目前所在的地理位置：亚洲 &gt; 中国 &gt; 某省 &gt; 某市 &gt; ….，**就是这样一个结构！**

### 有序列表zset基础操作

```c
COPY> ZADD books 9.0 "think in java"
> ZADD books 8.9 "java concurrency"
> ZADD books 8.6 "java cookbook"

> ZRANGE books 0 -1     # 按 score 排序列出，参数区间为排名范围
1) "java cookbook"
2) "java concurrency"
3) "think in java"

> ZREVRANGE books 0 -1  # 按 score 逆序列出，参数区间为排名范围
1) "think in java"
2) "java concurrency"
3) "java cookbook"

> ZCARD books           # 相当于 count()
(integer) 3

> ZSCORE books "java concurrency"   # 获取指定 value 的 score
"8.9000000000000004"                # 内部 score 使用 double 类型进行存储，所以存在小数点精度问题

> ZRANK books "java concurrency"    # 排名
(integer) 1

> ZRANGEBYSCORE books 0 8.91        # 根据分值区间遍历 zset
1) "java cookbook"
2) "java concurrency"

> ZRANGEBYSCORE books -inf 8.91 withscores  # 根据分值区间 (-∞, 8.91] 遍历 zset，同时返回分值。inf 代表 infinite，无穷大的意思。
1) "java cookbook"
2) "8.5999999999999996"
3) "java concurrency"
4) "8.9000000000000004"

> ZREM books "java concurrency"             # 删除 value
(integer) 1
> ZRANGE books 0 -1
1) "java cookbook"
2) "think in java"
```

\*\*\*\*

**扩展/相关阅读**

1. \*\*\*\*[**阿里云 Redis 开发规范**](https://www.infoq.cn/article/K7dB5AFKI9mr5Ugbs_px)\*\*\*\*
2. \*\*\*\*[**为什么要防止bigkey？** ](%20https://mp.weixin.qq.com/s?__biz=Mzg2NTEyNzE0OA==&mid=2247483677&idx=1&sn=5c320b46f0e06ce9369a29909d62b401&chksm=ce5f9e9ef928178834021b6f9b939550ac400abae5c31e1933bafca2f16b23d028cc51813aec&scene=21#wechat_redirect)\*\*\*\*
3. \*\*\*\*[**Redis【入门】就这一篇！**](https://www.wmyskxz.com/2018/05/31/redis-ru-men-jiu-zhe-yi-pian/) ****



**参考资料:**

1. \*\*\*\*[**Redis\(1\)——5种基本数据结构**](https://www.wmyskxz.com/2020/02/28/redis-1-5-chong-ji-ben-shu-ju-jie-gou/)\*\*\*\*
2. \*\*\*\*[**《Redis 设计与实现》**](http://redisbook.com/)\*\*\*\*
3. \*\*\*\*[**【官方文档】Redis 数据类型介绍**](%20http://www.redis.cn/topics/data-types-intro.html)\*\*\*\*
4. \*\*\*\*[**《Redis 深度历险》**](%20https://book.douban.com/subject/30386804/)\*\*\*\*
5. \*\*\*\*[**阿里云Redis开发规范**](https://www.infoq.cn/article/K7dB5AFKI9mr5Ugbs_px)\*\*\*\*
6. \*\*\*\*[**Redis快速入门 - 易百教程** ](https://www.yiibai.com/redis/redis_quick_guide.html)\*\*\*\*



