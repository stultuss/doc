Redis
=========================

## 介绍

Redis 是一个**单线程**的 key-value 存储系统，提供了丰富的数据结构，包括了字符串（string），哈希表（hash），列表（list），集合（set），有序集合（sorted set）五种基础类型，还有一种在日常开发中较为有用的是 地理位置（geo），但仅在 3.2.0 版本以上可用。 

Redis 有很多优点：

- 性能高：支持 100K / s 以上的读写频率（不使用 Keys 等关键字的情况下）
- 数据类型丰富：支持二进制数据类型的操作
- 原子性：所有读写操作都是原子性的，同时还支持使用 Lua 脚本合并操作的原子执行
- 特性丰富：支持 publish/subscribe，通知，过期等等。

## 内存分配

Redis 的内存分配是自主分配内存的，在请求到的时候根据内建的算法分配内存，完全自主管理控制内存，Redis 对内存分配函数进行了一层封装，统一使用 zmalloc，zfree 等函数，源码位于 zmalloc.h，zmalloc.c 中。这种做法是为了屏蔽底层平台的差异，方便自己实现相关函数，举例：

- 若是系统中存在 Google 的 tc_malloc 库，则用 tc_malloc 的函数代替原本的 malloc 函数
- 若是系统是 Mac 系统，则使用 malloc.h 中的内存分配函数。
- 其他情况下，在每一段分配好的空间前面，同时多分配一个定长的字端。

自从 2.4 版本后， Redis 使用了一种新的内存分配机制，被称为 “jemalloc”。原因是由于作者认为 linux 的 glibc 的内存分配工具非常的烂，Redis开始受到内存碎片的影响，所以才引入  jemalloc 来解决内存碎片的问题 （仅 linux 平台更新）。

## LRU 机制

Redis 的内存分配机制决定了：当开发人员往 Redis 中写入数据时，如果未对数据设置过期时间，那么 Redis 使用的内存会持续增长下去直至 OOM。**和 Memcache 不同的是，Redis 并不会直接崩溃，而是尝试执行 LRU 机制（除非 maxmemory-policy 设置为 noeviction）。**

- allkeys-lru：在所有key中按照最近最少使用LRU原则剔除key，释放空间
- volatile-lru：仅以设置过期时间key范围内的LRU(如均为设置过期时间，则不会淘汰)
- allkeys-random：随机删除
- volatile-random：仅设置过期时间key范围内的随机
- volatile-ttl：按最小 TTL 的key优先淘汰

**注意：当执行 LRU 机制后，就可能会发生 Redis 主键失效的问题！！！**

## LFU 机制

从 Redis 4.0 开始，新增了 LFU 机制，提供了更好的缓冲命中率。通过记录键使用频率来定位最可能淘汰的键。

对比 LRU 和 LFU 的差别：

- 在 LRU 中，某个键很少被访问，但刚刚被访问后它被淘汰的概率很低。相对其他经常访问的反而会被淘汰。
- 在 LFU 中，按访问频次，淘汰最少被访问的键。

在 4.0 中新增了两种淘汰机制：

- volatile-lfu：设置过期时间的键按LFU淘汰
- allkeys-lfu：所有键按LFU淘汰

## Lua 脚本

Redis 允许使用者编写 Lua 编写脚本，使用 `eval` 命令对 Lua 脚本进行求值，这段脚本会被运行在 Redis 服务器的上下文中，举例：

 ```shell
> eval "return redis.call('set','foo','bar')" 0
OK
 ```

Redis 使用单个 Lua 解释器去运行所有脚本，并且 Redis 也保证脚本以 **原子性** 的方式执行。

## 常见问题

### 时间复杂度

Reids 的大多数命令都是 O(1) 的，但某些命令的时间复杂度是会达到 O(n)， 需要特别注意，例如：

| 类型 | API     |                                                              |      |
| ---- | ------- | ------------------------------------------------------------ | ---- |
| hash | mget    | 批量获取 key-value（n 代表 key 的数量）                      | O(n) |
| mget | mset    | 批量保存 key-value（n 代表 key 的数量）                      | O(n) |
| hash | hgetall | 返回 hash key 对应的所有的 field 个 value（n 代表 hash 表的大小） | O(n) |
| hash | hmget   | 批量获取一批数据（n 代表 key 的数量）                        | O(n) |
| hash | hmset   | 批量保存一批数据（n 代表 key 的数量）                        | O(n) |
| list | lrem    | 根据 count 值，从列表中删除所有的 value 相等的项（n 代表 list 长度） | O(n) |
| list | ltrim   | 按照索引范围修剪列表（n 代表 list 长度）                     | O(n) |
| list | lrange  | 获取列表指定索引范围所有的 item（n 代表 list 长度）          | O(n) |
| list | lindex  | 获取指定列表索引 item（n 代表 list 长度）                    | O(n) |
| list | lset    | 设置列表指定位置指定值（n 代表 list 长度）                   | O(n) |

**由上可知，hash类型保存的 field 一定是有限个，并且需要限制大小的，而 list 类型最适合的操作方式是从最上或最下弹出数据，而不要从中间操作，否则每次操作都是对整个队列的遍历操作。**

### 排行榜

简单的排行榜只需要将数据存入有序集合，Redis 自动就可以将数据进行排序了。**问题：但如果有一个需求是将相同分数按照插入的时间顺序排列，应当如何处理呢**？

我们可以选择一个很远的时间，例如 2030-01-01，然后减去现在的时间 2020-01-01 就会得到一个秒数。然后我们将这个毫秒数除上  `Math.pow(10,12)`，再和 score 相加再存入 Redis 即可，Redis 也可以对这个 Score 进行排序。

```js
const FUTURE_TIMESTAMP = 1893427200000
const NOW_TIMESTAMP = 1577808000000

await redis.zadd('RANKING_KEY', 100 + (FUTURE_TIMESTAMP - NOW_TIMESTAMP / Math.pow(10, 12))
```

### 集群内删除指定数据

```shell
#!/bin/bash

REDIS_HOST='192.168.1.10'
REDIS_PASSWD='XXXXXX'
DEL_KEYS='db_game:*'

oldifs="$IFS"
IFS=$'\n'

for node in `redis-cli -h ${REDIS_HOST} -a ${REDIS_PASSWD} cluster nodes`
do
  echo 'start: ' ${node:0:40}
  echo `redis-cli -h ${REDIS_HOST} -a ${REDIS_PASSWD} keys ${DEL_KEYS} ${node:0:40} | xargs -i redis-cli -h ${REDIS_HOST} -a ${REDIS_PASSWD} del {}`
  echo `redis-cli -h ${REDIS_HOST} -a ${REDIS_PASSWD} keys ${DEL_KEYS} ${node:0:40} | wc -l`
done

IFS="$oldifs"
```

