title: 妈妈再也不用担心别人问我是否真正用过redis了
date: 2015-12-31 14:55:48
tags: [nosql,redis]
categories: [中级]
---
持续更新...

Redis最近几年很火。笔者很羞愧，很长时间内以为和Memcache一样，只能做缓存。事实上Redis功能丰富，十八般武艺样样具全，几乎适用于互联网行业的各个场景，包括存储。但Redis存储相当的贵，而且大文件不适合这种场景。
Redis从设计到运维到使用，都是很大的话题，在面试中被问到的概率越来越大，大部分都是使用问题。简单的了解以下内容，妈妈再也不用担心别人问我是否真正用过redis了。

### Memcache与Redis的区别
现在的硬盘速度很不给力，所以才有了各种各样的缓存，redis最初是作为缓存设计的，相对于另一个缓存大明星Memcache，它们有以下不同。
#### 存储方式不同
Memecache把数据全部存在内存之中，断电后会挂掉，数据不能超过内存大小。
Redis有部份存在硬盘上，这样能保证数据的持久性。

#### 数据支持类型
Memcache对数据类型支持相对简单。
Redis有复杂的数据类型。可以玩很多花样。

#### 使用底层模型不同
它们之间底层实现方式 以及与客户端之间通信的应用协议不一样。
Redis直接自己构建了VM机制 ，因为一般的系统调用系统函数的话，会浪费一定的时间去移动和请求。

### Redis支持的数据类型
Redis支持5种数据类型strings, hashes, lists, sets, sorted sets
- strings 简单字符串，底层使用sds实现
- lists 简单列表，按照需求，底层使用双向链表(linkedlist)或者压缩列表(ziplist)实现
- sets 无序set，底层存储使用intset或者hashtable
- sorted sets 有序set，底层使用压缩列表或者"跳跃表+哈希"实现
- hashes 哈希表，按照需求，底层使用压缩列表或者哈希表实现

### Redis的回收策略
- volatile-lru：从已设置过期时间的数据集（server.db[i].expires）中挑选最近最少使用的数据淘汰
- volatile-ttl：从已设置过期时间的数据集（server.db[i].expires）中挑选将要过期的数据淘汰
- volatile-random：从已设置过期时间的数据集（server.db[i].expires）中任意选择数据淘汰
- allkeys-lru：从数据集（server.db[i].dict）中挑选最近最少使用的数据淘汰
- allkeys-random：从数据集（server.db[i].dict）中任意选择数据淘汰
- no-enviction（驱逐）：禁止驱逐数据

### Redis小命令
其他命令参考[http://redisdoc.com](http://redisdoc.com)

#### 连接
```bash
#连接上redis_001.jr-jr.com，指定host和port
redis-cli -h redis_001.jr-jr.com -p 3306
#注意。可以直接在后面跟上命令
redis-cli -h redis_001.jr-jr.com -p 3306 monitor
```
#### MONITOR
MONITOR是一个调试命令，返回服务器处理的每一个命令，它能帮助我们了解在数据库上发生了什么操作。配合grep可以查看是否有你想要的操作。
由于MONITOR命令返回服务器处理的所有的命令, 所以在性能上会有一些消耗。
```
redis-cli -h redis_001.jr-jr.com -p 3306 monitor | grep "nnn"
```
#### SLOWLOG
通过SLOWLOG可以读取慢查询日志。
使用SLOWLOG LEN就可以获取当前慢日志长度。
使用SLOWLOG GET N就可以获取最近N条慢日志。
使用SLOWLOG RESET命令重置慢日志。一旦执行，将丢失以前的所有慢日志。

#### INFO
详细信息可以看这里
[http://redis.io/commands/info](http://redis.io/commands/info)
[http://redisdoc.com/server/info.html](http://redisdoc.com/server/info.html)

主要关注Memory帧
used_memory : 由 Redis 分配器分配的内存总量，以字节（byte）为单位
used_memory_rss : 从操作系统的角度，返回 Redis 已分配的内存总量（俗称常驻集大小）。这个值和 top 、 ps 等命令的输出一致。

rdb_changes_since_last_save 如果频繁进行save操作，会引起redis卡顿，需要定位
connected_clients 有多少个客户端连接，看详细列表可以使用`CLIENT LIST`

instantaneous_ops_per_sec 另外关注ops，达到2w需要高度关注

keyspace_hits
keyspace_misses 缓存命中，顾名思义，被当作缓存使用时有意义

### 应用场景
进入我们的主旋律，怎么运用redis，具体redis的运维，交给专业DBA吧。不明白的redis指令，希望自己能查查。
啊啊啊啊，妈妈再也不用担心别人问我是否用过redis了。

#### 缓存
对热点数据进行缓存，比如经常访问的用户资料等。内存价位较高，配合设置过期时间，效果更佳。

#### 对用户访问某个API进行频率限制
比起各种分布式协调方案，使用Redis简单多了。

#### 批量获取key
当你的Redis运行在大循环中，使用`MGET`、`MSET`会有意想不到的效果。这是SQL的Batch模式。

#### 用户属性存储
使用大JSON或者`HSET`、`HGET`操作HASH，其中，某个属性也可以使用`HINCRBY`自增哦，牛逼吧。

#### 实现计数器
用来实现 用户： 总点赞数，关注数，粉丝数；帖子： 点赞数，评论数，热度；消息： 已读，未读，红点消息数； 话题： 阅读数，帖子数，收藏数
使用`INCR`即可解决。原子的哦。
但具体的应用场景亦有它的复杂性[[转]微架构设计：微博计数器的设计 http://blog.csdn.net/heiyeshuwu/article/details/7972050](http://blog.csdn.net/heiyeshuwu/article/details/7972050)

#### 分布式锁
通过`SETNX`来实现锁的获取，通过设置`TTL`来设置超时时间，通过`DEL`完成锁的释放。
例如：
加锁：`SETNX foo.lock <current unix time>`
释放锁：`DEL foo.lock`
扩展问题：怎么检测死锁，并解决它呢？自己搜索一下吧。

#### 取最新N个数据的操作
使用`LPUSH`插入
使用`LTRIM`让列表只保存前N条

#### 排行榜
使用`有序的SET`，比如，得到前100名高分用户很简单：`ZREVRANGE leaderboard 0 99`
同理，可以使用`ZREMRANGEBYRANK`让有序集合保存前N条
例子[http://segmentfault.com/a/1190000002694239](http://segmentfault.com/a/1190000002694239)

#### 用户未读消息列表
使用`SADD`、`SCARD`、`DEL`等

#### 黑名单、关注列表、粉丝列表、双向关注列表
使用`ZADD`、`ZRANK`等，将用户的黑名单使用ZADD添加，ZRANK使用返回的sorce值判断是否存在黑名单中

#### Uniq操作，获取某段时间所有数据排重值
这个使用Redis的set数据结构最合适了，只需要不断地将数据往set中扔就行了，set意为集合，所以会自动排重。更何况有`SUNION`等合并两个集合的命令。

#### 用setbit(bitmap)统计每天活跃用户
Bitmap是一串连续的2进制数字（0或1），每一位所在的位置为偏移(offset)，在bitmap上可执行AND,OR,XOR以及其它位操作。
每一位标识一个用户ID。当某个用户访问我们的网页或执行了某个操作，就在bitmap中把标识此用户的位置为1。
使用`SETBIT`、`GETBIT`、`BITCOUNT`等。

#### 队列
`LPUSH`和`RPOP`命令。
如果要实现任务队列，只需要让生产者将任务使用LPUSH命令加入到某个键中，另一边让消费者不断地使用RPOP命令从该键中取出任务即可。
RPOP是非阻塞的，如果想要阻塞队列，使用`BRPOP`，此命令带有超时参数，可得到N个消息。

#### Pub、Sub构建实时消息系统
Redis 的 Pub/Sub 系统可以构建实时的消息系统，比如很多用 Pub/Sub 构建的实时聊天系统的例子

#### LBS应用
Redis3.2将推出GEO功能，将会是除了`PostGIS`以外有力的开源解决方案。更多功能期待ing...
可以实现：
- 两个位置之间的距离
- 查找附近的人
- 摇一摇

Redis通过组合，可以实现N多功能，如果你有更好的方法，请及时告诉我。

### 扩展阅读
- 关于Redis的常识 [http://blog.csdn.net/andy1219111/article/details/18984355](http://blog.csdn.net/andy1219111/article/details/18984355)
- Redis GEO 特性简介[http://blog.jobbole.com/89225/](http://blog.jobbole.com/89225/)
