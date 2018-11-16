---
title: redis 相关学习记录
date: 2018-11-16 23:14:47
tags: redis
categories: 技术
comment: true
---

### 1、一些常规的概念

Redis是一个开源的，遵守BSD协议，基于内存的结构化数据存储媒介，可以作为数据库、缓存服务或消息服务使用。
Redis支持多种数据结构，包括字符串、哈希表、链表、集合、有序集合、位图、Hyperloglogs等。
Redis具备LRU淘汰、事务实现、以及不同级别的硬盘持久化等能力(后面详解)，并且支持副本集和通过Redis Sentinel实现的高可用方案，同时还支持通过Redis Cluster实现的数据自动分片能力。
Redis的主要功能都基于单线程模型实现，也就是说Redis使用一个线程来服务所有的客户端请求，同时Redis采用了非阻塞式IO，并精细地优化各种命令的算法时间复杂度，这些信息意味着：

- Redis是线程安全的（因为只有一个线程），其所有操作都是原子的，不会因并发产生数据异常
- Redis的速度非常快（因为使用非阻塞式IO，且大部分命令的算法时间复杂度都是O(1))
- 使用高耗时的Redis命令是很危险的，会占用唯一的一个线程的大量处理时间，导致所有的请求都被拖慢。（例如时间复杂度为O(N)的KEYS命令，严格禁止在生产环境中使用）

### 2、redis 配置文件的一些简单说明 redis 5.0版本

redis 大版本的迭代 2.4|2.6|2.8|3.0|3.2|4.0|5.0

### 3、数据结构和相关常用的命令(参考<http://redisdoc.com/>)

#### 1、key(键)

Redis采用Key-Value型的基本数据结构，任何二进制序列都可以作为Redis的Key使用

- key 的长度太长查询效率低，太短的话确实可读性，最好有个统一的规范

- Redis允许的最大Key长度是512MB（对Value的长度限制也是512MB）

命令相关：

- **DEL**: 该命令用于在 key 存在是删除 key。 
- **DUMP** : 序列化给定 key ，并返回被序列化的值。 
- **EXISTS** : 检查给定 key 是否存在。 
- **EXPIRE** : seconds 为给定 key 设置过期时间。 
- **EXPIREAT** : timestamp EXPIREAT 的作用和 EXPIRE 类似，都用于为 key 设置过期时间。 不同在于 **EXPIREAT** 命令接受的时间参数是 UNIX 时间戳(unix timestamp)。 
- **PEXPIRE** : 设置 key 的过期时间以毫秒计。 
- **PEXPIREAT** : 设置 key 过期时间的时间戳(unix timestamp) 以毫秒计 
- **KEYS** : 查找所有符合给定模式( pattern)的 key 。 类似于正则匹配一样(线上数据库谨慎使用)
- **MOVE** : 将当前数据库的 key 移动到给定的数据库 db 当中。 
- **PERSIST** : 移除 key 的过期时间，key 将持久保持。 
- **PTTL** : 以毫秒为单位返回 key 的剩余的过期时间。 
- **TTL** : 以秒为单位，返回给定 key 的剩余生存时间(TTL, time to live)。 
- **RANDOMKEY**:  从当前数据库中随机返回一个 key 。 
- **RENAME** : key newkey 修改 key 的名称 (如果 newkey 已经存在，会被 key 覆盖)
- **RENAMENX** : key newkey 仅当 newkey 不存在时，将 key 改名为 newkey 。 
- **TYPE**: 返回 key 所储存的值的类型。

#### 2、String(字符串)

String是Redis的基础数据类型，Redis没有Int、Float、Boolean等数据类型的概念，所有的基本类型在Redis中都以String体现。

与String相关的常用命令：

- **SET**：为一个key设置value，可以配合EX/PX参数指定key的有效期，通过NX/XX参数针对key是否存在的情况进行区别操作，时间复杂度O(1)
- **GET**：获取某个key对应的value，时间复杂度O(1)
- **GETSET**：为一个key设置value，并返回该key的原value(不存在返回 nil)，时间复杂度O(1)
- **SETNX**: 只有在 key 不存在时设置 key 的值。
- **SETEX**: 将值 value 关联到 key ，并将 key 的过期时间设为 seconds (以秒为单位)。(存在的数据会被覆盖)
- **STRLEN**: 返回字符串长度
- **MSET**：为多个key设置value，时间复杂度O(N)
- **MSETNX**：同MSET，如果指定的key中有任意一个已存在，则不进行任何操作，时间复杂度O(N)
- **MGET**：获取多个key对应的value，时间复杂度O(N)

上文提到过，Redis的基本数据类型只有String，但Redis可以把String作为整型或浮点型数字来使用，主要体现在INCR、DECR类的命令上：

- **INCR**：将key对应的value值自增1，并返回自增后的值。只对可以转换为整型的String数据起作用。时间复杂度O(1)
- **INCRBY**：将key对应的value值自增指定的整型数值，并返回自增后的值。只对可以转换为整型的String数据起作用。时间复杂度O(1)
- **DECR/DECRBY**：同INCR/INCRBY，自增改为自减。

INCR/DECR系列命令要求操作的value类型为String，并可以转换为64位带符号的整型数字，否则会返回错误。
也就是说，进行INCR/DECR系列命令的value，必须在[-2^63 ~ 2^63 - 1]范围内。

#### **3、List(列表)**:

Redis的List是链表型的数据结构，可以使用LPUSH/RPUSH/LPOP/RPOP等命令在List的两端执行插入元素和弹出元素的操作。虽然List也支持在特定index上插入和读取元素的功能，但其时间复杂度较高（O(N)），应小心使用。

与List相关的常用命令：

- **LPUSH**：向指定List的左侧（即头部）插入1个或多个元素，返回插入后的List长度。时间复杂度O(N)，N为插入元素的数量
- **RPUSH**：同LPUSH，向指定List的右侧（即尾部）插入1或多个元素
- **LPOP**：从指定List的左侧（即头部）移除一个元素并返回，时间复杂度O(1)
- **RPOP**：同LPOP，从指定List的右侧（即尾部）移除1个元素并返回
- **LPUSHX/RPUSHX**：与LPUSH/RPUSH类似，区别在于，LPUSHX/RPUSHX操作的key如果不存在，则不会进行任何操作
- **LLEN**：返回指定List的长度，时间复杂度O(1)
- **LRANGE**：返回指定List中指定范围的元素（双端包含，即LRANGE key 0 10会返回11个元素），时间复杂度O(N)。应尽可能控制一次获取的元素数量，一次获取过大范围的List元素会导致延迟，同时对长度不可预知的List，避免使用LRANGE key 0 -1这样的完整遍历操作。

应谨慎使用的List相关命令：

- **LINDEX**：返回指定List指定index上的元素，如果index越界，返回nil。index数值是回环的，即-1代表List最后一个位置，-2代表List倒数第二个位置。时间复杂度O(N)
- **LSET**：将指定List指定index上的元素设置为value，如果index越界则返回错误，时间复杂度O(N)。如果操作的是头/尾部的元素，则时间复杂度为O(1)
- **LINSERT**：向指定List中指定元素之前/之后插入一个新元素，并返回操作后的List长度。如果指定的元素不存在，返回-1。如果指定key不存在，不会进行任何操作，时间复杂度O(N)

由于Redis的List是链表结构的，上述的三个命令的算法效率较低，需要对List进行遍历，命令的耗时无法预估，在List长度大的情况下耗时会明显增加，应谨慎使用。

换句话说，Redis的List实际是设计来用于实现队列，而不是用于实现类似ArrayList这样的列表的。如果你不是想要实现一个双端出入的队列，那么请尽量不要使用Redis的List数据结构。

为了更好支持队列的特性，Redis还提供了一系列阻塞式的操作命令，如BLPOP/BRPOP等，能够实现类似于BlockingQueue的能力，即在List为空时，阻塞该连接，直到List中有对象可以出队时再返回。

例： 消息队列，多进程共享消息

#### 4、hash(哈希表)

Hash即哈希表，Redis的Hash和传统的哈希表一样，是一种field-value型的数据结构，可以理解成将HashMap搬入Redis。

Hash非常适合用于表现对象类型的数据，用Hash中的field对应对象的field即可。
Hash的优点包括：

- 可以实现二元查找，如"查找ID为1000的用户的年龄"
- 比起将整个对象序列化后作为String存储的方法，Hash能够有效地减少网络传输的消耗
- 当使用Hash维护一个集合时，提供了比List效率高得多的随机访问命令

与Hash相关的常用命令：

- **HSET**：将key对应的Hash中的field设置为value。如果该Hash不存在，会自动创建一个。时间复杂度O(1)
- **HGET**：返回指定Hash中field字段的值，时间复杂度O(1)
- **HMSET/HMGET**：同HSET和HGET，可以批量操作同一个key下的多个field，时间复杂度：O(N)，N为一次操作的field数量
- **HSETNX**：同HSET，但如field已经存在，HSETNX不会进行任何操作，时间复杂度O(1)
- **HEXISTS**：判断指定Hash中field是否存在，存在返回1，不存在返回0，时间复杂度O(1)
- **HDEL**：删除指定Hash中的field（1个或多个），时间复杂度：O(N)，N为操作的field数量
- **HINCRBY**：同INCRBY命令，对指定Hash中的一个field进行INCRBY，时间复杂度O(1)

应谨慎使用的Hash相关命令：

- **HGETALL**：返回指定Hash中所有的field-value对。返回结果为数组，数组中field和value交替出现。时间复杂度O(N)
- **HKEYS/HVALS**：返回指定Hash中所有的field/value，时间复杂度O(N)

上述三个命令都会对Hash进行完整遍历，Hash中的field数量与命令的耗时线性相关，对于尺寸不可预知的Hash，应严格避免使用上面三个命令，而改为使用HSCAN命令进行游标式的遍历

例： 存储、读取、修改用户属性

#### 5、SET(集合)：

Redis Set是无序的，不可重复的String集合。

与Set相关的常用命令：

- **SADD**：向指定Set中添加1个或多个member，如果指定Set不存在，会自动创建一个。时间复杂度O(N)，N为添加的member个数
- **SREM**：从指定Set中移除1个或多个member，时间复杂度O(N)，N为移除的member个数
- **SRANDMEMBER**：从指定Set中随机返回1个或多个member，时间复杂度O(N)，N为返回的member个数
- **SPOP**：从指定Set中随机移除并返回count个member，时间复杂度O(N)，N为移除的member个数
- **SCARD**：返回指定Set中的member个数，时间复杂度O(1)
- **SISMEMBER**：判断指定的value是否存在于指定Set中，时间复杂度O(1)
- **SMOVE**：将指定member从一个Set移至另一个Set

慎用的Set相关命令：

- **SMEMBERS**：返回指定Hash中所有的member，时间复杂度O(N)
- **SUNION/SUNIONSTORE**：计算多个Set的并集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数
- **SINTER/SINTERSTORE**：计算多个Set的交集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数
- **SDIFF/SDIFFSTORE**：计算1个Set与1或多个Set的差集并返回/存储至另一个Set中，时间复杂度O(N)，N为参与计算的所有集合的总member数

上述几个命令涉及的计算量大，应谨慎使用，特别是在参与计算的Set尺寸不可知的情况下，应严格避免使用。可以考虑通过SSCAN命令遍历获取相关Set的全部member，如果需要做并集/交集/差集计算，可以在客户端进行，或在不服务实时查询请求的Slave上进行。

例： 共同好友、二度好友，利用唯一性，可以统计访问网站的所有独立 IP，当天去重(实际使用过)

#### 6、SORTED SET(有序集合)：

Redis Sorted Set是有序的、不可重复的String集合。Sorted Set中的每个元素都需要指派一个分数(score)，Sorted Set会根据score对元素进行升序排序。如果多个member拥有相同的score，则以字典序进行升序排序。

Sorted Set非常适合用于实现排名。

Sorted Set的主要命令：

- **ZADD**：向指定Sorted Set中添加1个或多个member，时间复杂度O(Mlog(N))，M为添加的member数量，N为Sorted Set中的member数量
- **ZREM**：从指定Sorted Set中删除1个或多个member，时间复杂度O(Mlog(N))，M为删除的member数量，N为Sorted Set中的member数量
- **ZCOUNT**：返回指定Sorted Set中指定score范围内的member数量，时间复杂度：O(log(N))
- **ZCARD**：返回指定Sorted Set中的member数量，时间复杂度O(1)
- **ZSCORE**：返回指定Sorted Set中指定member的score，时间复杂度O(1)
- **ZRANK/ZREVRANK**：返回指定member在Sorted Set中的排名，ZRANK返回按升序排序的排名，ZREVRANK则返回按降序排序的排名。时间复杂度O(log(N))
- **ZINCRBY**：同INCRBY，对指定Sorted Set中的指定member的score进行自增，时间复杂度O(log(N))

慎用的Sorted Set相关命令：

- **ZRANGE/ZREVRANGE**：返回指定Sorted Set中指定排名范围内的所有member，ZRANGE为按score升序排序，ZREVRANGE为按score降序排序，时间复杂度O(log(N)+M)，M为本次返回的member数
- **ZRANGEBYSCORE/ZREVRANGEBYSCORE**：返回指定Sorted Set中指定score范围内的所有member，返回结果以升序/降序排序，min和max可以指定为-inf和+inf，代表返回所有的member。时间复杂度O(log(N)+M)
- **ZREMRANGEBYRANK/ZREMRANGEBYSCORE**：移除Sorted Set中指定排名范围/指定score范围内的所有member。时间复杂度O(log(N)+M)

上述几个命令，应尽量避免传递[0 -1]或[-inf +inf]这样的参数，来对Sorted Set做一次性的完整遍历，特别是在Sorted Set的尺寸不可预知的情况下。可以通过ZSCAN命令来进行游标式的遍历，或通过LIMIT参数来限制返回member的数量（适用于ZRANGEBYSCORE和ZREVRANGEBYSCORE命令），以实现游标式的遍历。

例：带有权重的元素，比如一个游戏的用户得分排行榜，成绩排行

#### 7、Bitmap和HyperLogLog和 GEO地理位置：

Redis的这两种数据结构相较之前的并不常用

Bitmap在Redis中不是一种实际的数据类型，而是一种将String作为Bitmap使用的方法。可以理解为将String转换为bit数组。使用Bitmap来存储true/false类型的简单数据极为节省空间。

HyperLogLogs是一种主要用于数量统计的数据结构，它和Set类似，维护一个不可重复的String集合，但是HyperLogLogs并不维护具体的member内容，只维护member的个数。也就是说，HyperLogLogs只能用于计算一个集合中不重复的元素数量，所以它比Set要节省很多内存空间。

GEO 3.2 版本以后增加的

#### 8、pub/sub,事务:

Redis 服务器收到来自客户端的 MULTI 命令后，为客户端保存一个命令队列结构体，直到收到 EXEC 后才开始执行命令队列中的命令。

需要注意的是，**Redis事务不支持回滚**：
如果一个事务中的命令出现了语法错误，大部分客户端驱动会返回错误，2.6.5版本以上的Redis也会在执行EXEC时检查队列中的命令是否存在语法错误，如果存在，则会自动放弃事务并返回错误。
但如果一个事务中的命令有非语法类的错误（比如对String执行HSET操作），无论客户端驱动还是Redis都无法在真正执行这条命令之前发现，所以事务中的所有命令仍然会被依次执行。在这种情况下，会出现一个事务中部分命令成功部分命令失败的情况，然而与RDBMS不同，Redis不提供事务回滚的功能，所以只能通过其他方法进行数据的回滚。

Redis提供了WATCH命令与事务搭配使用，实现CAS乐观锁的机制。(比如修改商品的一个片段)

### 4、其他的一些概念和原理：

#### 数据持久化

Redis提供了将数据定期自动持久化至硬盘的能力，包括RDB和AOF两种方案，两种方案分别有其长处和短板，可以配合起来同时运行，确保数据的稳定性。

Redis的数据持久化机制是可以关闭的。如果你只把Redis作为缓存服务使用，Redis中存储的所有数据都不是该数据的主体而仅仅是同步过来的备份，那么可以关闭Redis的数据持久化机制。
但通常来说，仍然建议至少开启RDB方式的数据持久化，因为：

- RDB方式的持久化几乎不损耗Redis本身的性能，在进行RDB持久化时，Redis主进程唯一需要做的事情就是fork出一个子进程，所有持久化工作都由子进程完成
- Redis无论因为什么原因crash掉之后，重启时能够自动恢复到上一次RDB快照中记录的数据。这省去了手工从其他数据源（如DB）同步数据的过程，而且要比其他任何的数据恢复方式都要快
- 现在硬盘那么大，真的不缺那一点地方

##### RDB

采用RDB持久方式，Redis会定期保存数据快照至一个rbd文件中，并在启动时自动加载rdb文件，恢复之前保存的数据。可以在配置文件中配置Redis进行快照保存的时机：

```
save [seconds] [changes]
```

意为在[seconds]秒内如果发生了[changes]次数据修改，则进行一次RDB快照保存，例如

```
save 60 100
```

会让Redis每60秒检查一次数据变更情况，如果发生了100次或以上的数据变更，则进行RDB快照保存。
可以配置多条save指令，让Redis执行多级的快照保存策略。
Redis默认开启RDB快照，默认的RDB策略如下：

```
save 900 1
save 300 10
save 60 10000
```

也可以通过**BGSAVE**命令手工触发RDB快照保存。

**RDB的优点：**

- 对性能影响最小。如前文所述，Redis在保存RDB快照时会fork出子进程进行，几乎不影响Redis处理客户端请求的效率。
- 每次快照会生成一个完整的数据快照文件，所以可以辅以其他手段保存多个时间点的快照（例如把每天0点的快照备份至其他存储媒介中），作为非常可靠的灾难恢复手段。
- 使用RDB文件进行数据恢复比使用AOF要快很多。

**RDB的缺点：**

- 快照是定期生成的，所以在Redis crash时或多或少会丢失一部分数据。
- 如果数据集非常大且CPU不够强（比如单核CPU），Redis在fork子进程时可能会消耗相对较长的时间（长至1秒），影响这期间的客户端请求。

##### AOF

采用AOF持久方式时，Redis会把每一个写请求都记录在一个日志文件里。在Redis重启时，会把AOF文件中记录的所有写操作顺序执行一遍，确保数据恢复到最新。

AOF默认是关闭的，如要开启，进行如下配置：

```
appendonly yes
```

AOF提供了三种fsync配置，always/everysec/no，通过配置项[appendfsync]指定：

- appendfsync no：不进行fsync，将flush文件的时机交给OS决定，速度最快
- appendfsync always：每写入一条日志就进行一次fsync操作，数据安全性最高，但速度最慢
- appendfsync everysec：折中的做法，交由后台线程每秒fsync一次

随着AOF不断地记录写操作日志，必定会出现一些无用的日志，例如某个时间点执行了命令**SET key1 "abc"**，在之后某个时间点又执行了**SET key1 "bcd"**，那么第一条命令很显然是没有用的。大量的无用日志会让AOF文件过大，也会让数据恢复的时间过长。
所以Redis提供了AOF rewrite功能，可以重写AOF文件，只保留能够把数据恢复到最新状态的最小写操作集。
AOF rewrite可以通过**BGREWRITEAOF**命令触发，也可以配置Redis定期自动进行：

```
auto-aof-rewrite-percentage 100
auto-aof-rewrite-min-size 64mb
```

上面两行配置的含义是，Redis在每次AOF rewrite时，会记录完成rewrite后的AOF日志大小，当AOF日志大小在该基础上增长了100%后，自动进行AOF rewrite。同时如果增长的大小没有达到64mb，则不会进行rewrite。

**AOF的优点：**

- 最安全，在启用appendfsync always时，任何已写入的数据都不会丢失，使用在启用appendfsync everysec也至多只会丢失1秒的数据。
- AOF文件在发生断电等问题时也不会损坏，即使出现了某条日志只写入了一半的情况，也可以使用redis-check-aof工具轻松修复。
- AOF文件易读，可修改，在进行了某些错误的数据清除操作后，只要AOF文件没有rewrite，就可以把AOF文件备份出来，把错误的命令删除，然后恢复数据。

**AOF的缺点：**

- AOF文件通常比RDB文件更大
- 性能消耗比RDB高
- 数据恢复速度比RDB慢

#### 内存管理与数据淘汰机制

##### 最大内存设置

默认情况下，在32位OS中，Redis最大使用3GB的内存，在64位OS中则没有限制。

在使用Redis时，应该对数据占用的最大空间有一个基本准确的预估，并为Redis设定最大使用的内存。否则在64位OS中Redis会无限制地占用内存（当物理内存被占满后会使用swap空间），容易引发各种各样的问题。

通过如下配置控制Redis使用的最大内存：

```
maxmemory 100mb
```

在内存占用达到了maxmemory后，再向Redis写入数据时，Redis会：

- 根据配置的数据淘汰策略尝试淘汰数据，释放空间
- 如果没有数据可以淘汰，或者没有配置数据淘汰策略，那么Redis会对所有写请求返回错误，但读请求仍然可以正常执行

在为Redis设置maxmemory时，需要注意：

- 如果采用了Redis的主从同步，主节点向从节点同步数据时，会占用掉一部分内存空间，如果maxmemory过于接近主机的可用内存，导致数据同步时内存不足。所以设置的maxmemory不要过于接近主机可用的内存，留出一部分预留用作主从同步。(写时复制)

##### 数据淘汰机制

Redis提供了5种数据淘汰策略：

- volatile-lru：使用LRU算法(Least Recently Used)进行数据淘汰（淘汰上次使用时间最早的，且使用次数最少的key），只淘汰设定了有效期的key
- allkeys-lru: 使用LRU算法进行数据淘汰，所有的key都可以被淘汰
- volatile-lfu: 使用LFU算法(Least Frequently Used）最近最少使用算法,只淘汰设定了有效期的key
- allkeys-lfu: 使用LFU算法进行数据淘汰，所有的key都可以被淘汰
- volatile-random：随机淘汰数据，只淘汰设定了有效期的key
- allkeys-random：随机淘汰数据，所有的key都可以被淘汰
- volatile-ttl：淘汰剩余有效期最短的key
- noeviction：不淘汰

最好为Redis指定一种有效的数据淘汰策略以配合maxmemory设置，避免在内存使用满后发生写入失败的情况。

一般来说，推荐使用的策略是volatile-lru，并辨识Redis中保存的数据的重要性。对于那些重要的，绝对不能丢弃的数据（如配置类数据等），应不设置有效期，这样Redis就永远不会淘汰这些数据。对于那些相对不是那么重要的，并且能够热加载的数据（比如缓存最近登录的用户信息，当在Redis中找不到时，程序会去DB中读取），可以设置上有效期，这样在内存不够时Redis就会淘汰这部分数据。

配置方法：

```python
maxmemory-policy volatile-lru   #默认是noeviction，即不进行数据淘汰
```



### 5、经常会遇到的一些问题(redis 常联系到的问题)

**缓存穿透**:

缓存穿透是指查询一个一定不存在的数据，由于缓存是不命中时被动写的，并且出于容错考虑，如果从存储层查不到数据则不写入缓存，这将导致这个不存在的数据每次请求都要到存储层去查询，失去了缓存的意义。在流量大时，可能DB就挂掉了，要是有人利用不存在的key频繁攻击我们的应用，这就是漏洞。

解决方案：

方法有很多，最常见的则是采用**布隆过滤器**，将所有可能存在的数据哈希到一个足够大的**bitmap**中，一个一定不存在的数据会被这个bitmap拦截掉，从而避免了对底层存储系统的查询压力。另外也有一个更为简单粗暴的方法，如果一个查询返回的数据为空（不管是数 据不存在，还是系统故障），我们仍然把这个空结果进行缓存。

**缓存雪崩**(缓存失效)：

缓存雪崩是指在我们设置缓存时采用了相同的过期时间，导致缓存在某一时刻同时失效，请求全部转发到DB，DB瞬时压力过重雪崩。

解决方案：

设计者考虑用加锁或者队列的方式保证缓存的单线 程（进程）写，从而避免失效时大量的并发请求落到底层存储系统上。另一种方案，缓存失效时间分散开，比如我们可以在原有的失效时间基础上增加一个随机值，比如1-5分钟随机，这样每一个缓存的过期时间的重复率就会降低，就很难引发集体失效的事件。

**缓存击穿**：

对于一些设置了过期时间的key，如果这些key可能会在某些时间点被超高并发地访问，是一种非常“热点”的数据。这个时候，需要考虑一个问题：缓存被“击穿”的问题，这个和缓存雪崩的区别在于这里针对某一key缓存，前者则是很多key。缓存在某个时间点过期的时候，恰好在这个时间点对这个Key有大量的并发请求过来，这些请求发现缓存过期一般都会从后端DB加载数据并回设到缓存，这个时候大并发的请求可能会瞬间把后端DB压垮

解决方案：

1、使用互斥锁(mutex key)：

简单地来说，就是在缓存失效的时候（判断拿出来的值为空），不是立即去load db，而是先使用缓存工具的某些带成功操作返回值的操作（比如Redis的SETNX或者Memcache的ADD）去set一个mutex key，当操作返回成功时，再进行load db的操作并回设缓存；否则，就重试整个get缓存的方法。

SETNX，是「SET if Not eXists」的缩写，也就是只有不存在的时候才设置，可以利用它来实现锁的效果。

2、"提前"使用互斥锁(mutex key)：
在value内部设置1个超时值(timeout1), timeout1比实际的memcache timeout(timeout2)小。当从cache读取到timeout1发现它已经过期时候，马上延长timeout1并重新设置到cache。然后再从数据库加载数据并设置到cache中。

3、将热点key分散为多个子key，然后存储到缓存集群的不同机器上，这些子key对应的value都和热点key是一样的。当通过热点key去查询数据时，通过某种hash算法随机选择一个子key，然后再去访问缓存机器，将热点分散到了多个子key上。

4、 "永远不过期":

这里的“永远不过期”包含两层意思：

(1) 从redis上看，确实没有设置过期时间，这就保证了，不会出现热点key过期问题，也就是“物理”不过期。

(2) 从功能上看，如果不过期，那不就成静态的了吗？所以我们把过期时间存在key对应的value里，如果发现要过期了，通过一个后台的异步线程进行缓存的构建，也就是“逻辑”过期

从实战看，这种方法对于性能非常友好，唯一不足的就是构建缓存时候，其余线程(非构建缓存的线程)可能访问的是老数据

5、资源隔离组件hystrix(本文) 资源保护等等。。。

### 6、杂谈

- 尽可能使用长连接或连接池，避免频繁创建销毁连接

- 客户端进行的批量数据操作，应使用Pipeline特性在一次交互中完成

- Redis绝大多数读写命令的时间复杂度都在O(1)到O(N)之间，在文本和官方文档中均对每个命令的时间复杂度有说明。 

- 通常来说，O(1)的命令是安全的，O(N)命令在使用时需要注意，如果N的数量级不可预知，则应避免使用  

- Redis提供了SCAN命令，可以对Redis中存储的所有key进行游标式的遍历，避免使用KEYS命令带来的性能问题。同时还有SSCAN/HSCAN/ZSCAN等命令，分别用于对Set/Hash/Sorted Set中的元素进行游标式遍历

- Redis在fork子进程时需要将内存分页表拷贝至子进程，以占用了24GB内存的Redis实例为例，共需要拷贝24GB / 4kB * 8 = 48MB的数据。在使用单Xeon 2.27Ghz的物理机上，这一fork操作耗时216ms。

- 当同一秒内有大量key过期时，也会引发Redis的延迟。在使用时应尽量将key的失效时间错开。

- redis 集群，经常想到的就是主从复制（读写分离)，集群分片(codis)操作。 <http://www.cnblogs.com/276815076/p/7245333.html>

     

### 7、感兴趣的点

**redis 实现分布式锁**

使用Redis的 SETNX 命令可以实现分布式锁

考虑一种情况，如果进程获得锁后，断开了与 Redis 的连接（可能是进程挂掉，或者网络中断），如果没有有效的释放锁的机制，那么其他进程都会处于一直等待的状态，即出现“死锁”。

SETNX 获得锁时，我们将键 lock.foo 的值设置为锁的有效时间，进程获得锁后，其他进程还会不断的检测锁是否已超时，如果超时，那么等待的进程也将有机会获得锁。

锁超时时，我们不能简单地使用 DEL 命令删除键 lock.foo 以释放锁。考虑以下情况，进程P1已经首先获得了锁 lock.foo，然后进程P1挂掉了。进程P2，P3正在不断地检测锁是否已释放或者已超时，
```python
# -*- coding: utf-8 -*-
import time
import redis


conn = redis.Redis(host='127.0.0.1', port=6379, password="123456", db=1)


class RedisLock(object):
    def __init__(self, key):
        self._lock = 0
        self.lock_key = "%s_dynamic_test" % key

    @staticmethod
    def get_lock(cls, timeout=10):
        while cls._lock != 1:
            now = time.time()
            lock_timeout = now + timeout + 1
            cls._lock = conn.setnx(cls.lock_key, lock_timeout)
            if cls._lock == 1 or (now > conn.get(cls.lock_key) and now > conn.getset(cls.lock_key, lock_timeout)):
                print("get lock")
                break
            else:
                time.sleep(0.3)

    @staticmethod
    def release(cls):
        if time.time() < conn.get(cls.lock_key):
            print("release lock")
            conn.delete(cls.lock_key)


def deco(cls):
    def _deco(func):
        def __deco(*args, **kwargs):
            print("before %s called [%s]." % (func.__name__, cls))
            cls.get_lock(cls)
            try:
                return func(*args, **kwargs)
            finally:
                cls.release(cls)
        return __deco
    return _deco


@deco(RedisLock("112233"))
def myfunc():
    print("myfunc() called.")
    time.sleep(5)


if __name__ == "__main__":
    myfunc()
```
