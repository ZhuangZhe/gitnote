# 命令

| 命令 | 作用 |
| :--- | :--- |
| `redis-server` | Redis服务器启动命令 |
| `redis-cli` | 客户端，操作入口 |
| `redis-sentinel` | Redis集群适用 |
| `redis-check-aof` | 修复有问题的AOF文件 |
| `redis-check-dump` | 修复有问题的dump.rdb文件 |
| `redis-benchmark` | 性能测试工具，服务器启动后执行 |
|  |  |
|  |  |
|  |  |



## CLI命令

| 命令 | 作用 |
| :--- | :--- |
| `ping` | 验证redis server是否启动 |
| `shotdown` |  |
| `select <dbid>` | 选择使用内部0到15的其中一个数据库 |
| `key <pattern>` | 按pettern查询key |
| `exists <key1 ...>` | 判断key时候存在 |
| `type <key>` | 查询key是否存在 |
| `del <key>` | 删除key所对应的数据 |
| `expire <key> <seconds>` | 为key设置过期时间，单位为秒 |
| `ttl <key>` | 查看还有多少秒过期，-1表示为永不过期，-2表示为已过期 |
| `dbsize` | 查看当前数据库的key的数量 |
| `flushdb` | 清空当前库 |
| `flushall` | 清空全部库 |
| `get <key>` | 查询对应键值 |
| `set <key> <value>` | 添加键值对 |
| `append <key> <value>` | 将给定的value追加到原值的末尾，完成后返回总长度 |
| `strlen <key>` | 获得指的键对应的值长度 |
| `setnx <key> <value>` | 只有在可以不存在时设置key的值 |
| `incr <key>` | 将键对应的值增加一，只能对数值进行此操作 |
| `decr <key>` | 将键对应的值增减一，只能对数值进行此操作 |
| `incrby <key> <步长>` | 将键对应的值增加特定步长，只能对数值进行此操作 |
| `decrby <key> <步长>` | 将键对应的值减少特定步长，只能对数值进行此操作 |
| `mset <key1> <value1> <key2> <value2> ...` | 同时设置一个或多个键值对 |
| `mget <key1> <key2> ...` | 同时获得多个值 |
| `msetnx <key1> <value1> <key2> <value2> ...` | 同时设置多个键值对，仅当所有的键都不存在的时候使用 |
| `getrange <key> <起始位置> <结束位置>` | 获得值得范围，类似于java中的substring |
| `setrange <key> <起始位置> <value>` | 用valye覆盖key所储存的字符串值，从起始位置开始 |
| `setex <key> <过期时间> <value>` | 设置键值对时，同时设置过期时间 |
| `getset <key> <value>` | 以旧换新，设置了新值同时获得旧值 |



