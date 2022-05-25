# Redis 简介

> Redis是一个速度非常快的非关系数据库，它可以存储键（key) 与5种不同类型的值（value)之间的映射，可以将存储在内存的键值对数据持久化到硬盘，可以使用复制特性来扩展读性能，还可以使用客户端分片[^分片]来扩展写性能。



# Redis的数据结构

​	Redis共有5种数据结构，分别为字符串（STRING)、列表（LIST）、集合（SET)、散列（HASH）、有序集合（ZSET)。

## 1. 字符串（STRING）

> 通过key-value的方法去存储字符串，字符串可以存储字符串、整数、浮点数这三种类型的值。

**常见的命令操作**

| 命令                        | 描述                                                         |
| --------------------------- | :----------------------------------------------------------- |
| GET key-name                | 获取指定键存储的值，获取失败会返回nil                        |
| SET key-name value          | 存储值到指定键上，成功返回OK                                 |
| DEL key-name                | 删除键值对，成功返回1                                        |
| INCR key-name               | 对键存储的值加1，若该键不存在或值为空，则会当作0做处理；若对应的值不能被解析为整数或浮点数，则会返回一个错误。 |
| DECR key-name               | 对键存储的值减1                                              |
| INCRBY key-name amount      | 对键存储的值加上amount                                       |
| DECRBY key-name amount      | 对键存储的值减去amount                                       |
| INCRBYFLOAT key-name amount | 对键存储的值加上浮点数amount，在Redis2.6版本以上可用         |

​	

**处理子串的命令**

| 命令                           | 描述                              |
| ------------------------------ | --------------------------------- |
| APPEND key-name value          | 将value值追加到key-name原先的值后 |
| GETRANGE key-name start end    | 获取[start,end]的子串，就是subStr |
| SETRANGE key-name offset value | 在偏移量offset开始的字串给定值    |



## 2. 列表（LIST）

> 一个列表结构可以有序地存储多个字符串



**常见的命令操作**

| 命令                          | 描述                                                         |
| ----------------------------- | ------------------------------------------------------------ |
| rpush（lpush） key-name value | 向key-name这个键右边(左边)增加一个value值                    |
| lrange key-name start end     | 获取key-name这个键上从start到end的所有值                     |
| lindex key-name index         | 向key-name这个列表中获取下标为index的值                      |
| lpop（rpop）  key-name        | 从key-name这个列表中最左(右)边弹出一个值                     |
| ltrim key-name start end      | 保留key-name这个列表中从start到end的所有值，相当于subList(start,end); |

## 3. 集合（SET）

> Redis的集合以无序的方法来存储多个各不相同的元素，用户可以快速地对集合执行添加元素操作、移除元素操作以及检查一个元素是否存在于集合里。与列表的不同在于，列表可以存储多个相同的字符串，而集合则通过使用散列表来保证自己存储的每个字符串都是不同的。



**常用的命令**

| 命令                           | 描述                                                         |
| ------------------------------ | ------------------------------------------------------------ |
| sadd key-name value [valu2...] | 添加元素到key-name集合中，后面可接多个                       |
| smembers key-name              | 返回key-name集合中的所有元素（当集合中元素非常多的时候，执行速度会很慢，所以要谨慎使用该命令 --- 大查询了） |
| sismember key-name value       | 检查value元素是否在key-name的集合里                          |
| srem key-name value            | 在key-name集合中删除value这个元素                            |
| scard key-name                 | 返回key-name这个集合中的元素数量                             |
| srandmember key-name [count]   | 从key-name这个集合中随机返回一个或多个元素。count的作用在于命令返回的随机元素会不会重复（正值不会重复） |
| spop key-name                  | 随机删除集合中的一个元素，并返回                             |
| smove source-key dest-key item | 如果source-key中包含Item，则在source-key中删除元素item，并将元素item添加到dest-key集合中；成功移除则返回1，否则返回0 |



**用于组合和处理多个集合的命令**

| 命令                                          | 描述                                                       |
| --------------------------------------------- | ---------------------------------------------------------- |
| sdiff key-name [key-name ...]                 | 差集。返回存在第一个集合、但不存在其他集合中的元素         |
| sdiffstrore dest-key key-name [key-name ...]  | 差集。与前面类似，但该命令会把结果存储到dest-key这个集合中 |
| sinter key-name [key-name ...]                | 交集。返回同时存在所有集合的元素                           |
| sinterstrore dest-key key-name [key-name ...] | 交集结果存储到desy-key这个集合上                           |
| sunion key-name [key-name ...]                | 并集。                                                     |
| sunionstrore dest-key key-name [key-name ...] | 并集结果存储到desy-key这个集合上                           |



## 4. 散列（HASH)

> Redis中的散列可以让用户将多个键值对存储到一个Redis键里面。就是有着一个标识的map，类比关系型数据库中的行。

**常用的命令**

| 命令                                     | 描述                                                         |
| ---------------------------------------- | ------------------------------------------------------------ |
| hset key-name sub-key1 value1            | 添加sub-key1 : value1 这样的键值对存储在key-name这个散列中   |
| hget key-name sub-key1                   | 从key-name这个散列值中获取sub-key1这个键对应的值             |
| hgetall key-name                         | 获取key-name这个散列的所有键值对 （尽量避免使用，量大会堵塞） |
| hdel key-name sub-key1                   | 在key-name这个散列中删除sub-key1这个键值对，若该键存在，会返回1，否则返回0 |
| **批量处理的命令**                       | **描述**                                                     |
| hmget key-name key [key...]              | 从散列中获取一个或多个键对应的值                             |
| hmset key-name key value [key value ...] | 在key-name这个散列中设置一个或多个键值对                     |
| hmdel key-name key [key...]              | 删除key-name这个散列值中的多个键值对，返回成功找到并删除的数量 |
| hlen key-name                            | 返回key-name这个散列中键值对的数量                           |

**其它命令**

| 命令                                | 描述                                   |
| ----------------------------------- | -------------------------------------- |
| hexists key-name key                | 检查key-name这个散列是否存在这个key    |
| hkeys key-name                      | 获取散列中所有的键                     |
| hvals key-name                      | 获取散列中所有的值                     |
| hincrby key-name key increment      | 在散列中的key对应的值加上整数increment |
| hincrbyfloat key-name key increment | 同上，针对浮点数                       |

## 5. 有序集合（ZSET)

> 有序集合与散列一样，存储的都是键值对。有序集合中的键被称为成员，是唯一的；值被成功为分值（score）, 分值必须为浮点数。
>
> 有序集合是Redis里面唯一一个既可以根据成员访问元素（这点和散列是一样的），又可以根据分值以及分值的排列顺序来访问元素的结构。

**常用的命令**

| 命令                                                  | 描述                                                         |
| ----------------------------------------------------- | ------------------------------------------------------------ |
| zadd key-name  score member [score member ...]        | 添加一个分值为score的成员到key-name这个有序集合中            |
| zrange key-name start end [withscores]                | 根据元素在有序排列中所处的位置，从有序集合里面获取多个元素，会并返回对应的分值（如果有withscores的话） |
| zrangebyscore key-name min-score max-score withscores | 获取分值在[min-score,max-score]的成员和分值                  |
| zrem key-name member [member...]                      | 删除有序集合中的成员                                         |
| zcard key-name                                        | 返回有序集合中的数量                                         |
| zincrby key-name increment member                     | 将member成员的分值加上increment                              |
| zcount key-name min max                               | 返回分值介于min和max之间的成员数量                           |
| zrank key-name member                                 | 返回member成员在有序集合中的排名                             |
| zscore key-name member                                | 返回成员member的分值                                         |

此外，有序集合还支持获取指定分值范围内的成员列表，并对它们进行排序。以及 交集、并集等操作。



# 其它命令

## 发布与订阅

> 发布/订阅的特点是订阅者负责订阅频道，发送者负责向频道发送二进制字符串消息。每当有消息被发送到频道时，频道中的所有订阅者都会收到消息。

## 排序

> 可以将sort命令看作是SQL中的order by语句

```shell
sort source-key [by pattern] [limit offset count] [get pattern] [asc|desc] [store dest-key]
```

## 事务

> 有时候需要同时提交几个命令，Redis提供了事务可以让一个客户端在不被其他客户端打断的情况下执行多个命令。使用事务可减少Redis与客户端的通信往返次数。

要想使用事务，则在先执行`multi`命令，后输入想放到这个事务的所有命令，输完后，最后输入`exec`命令提交事务。事务会将所有命令一次性发给Redis。

## 过期命令

> 给键一个过期时间，留内存一条生路。过期后，Redis会自动删除该键。

| 命令                                       | 描述                                                         |
| ------------------------------------------ | ------------------------------------------------------------ |
| persist key-name                           | 移除键的过期时间                                             |
| ttl key-name                               | 查看key-name这个键距离过期还有多少秒                         |
| expire key-name seconds                    | 对key-name这个键设置seconds秒的过期时间                      |
| expireat key-name timestamp                | 对key-name这个键设置给定的UNIX时间戳                         |
| pttl key-name                              | 查看给定键距离过期还有多少毫秒（Redis2.6以上）               |
| pexpire key-name milliseconds              | 对key-name这个键设置milliseconds毫秒的过期时间（Redis2.6以上） |
| pexpireeat key-name timestamp-milliseconds | 对key-name这个键设置给定毫秒精度的UNIX时间戳                 |

​	过期时间的设置对于列表、集合、散列和有序集合这样的容器来说，键过期命令只能为整个键设置过期时间，而没办法为键里面的单个元素设置过期时间。为解决这个问题，可以考虑采用存储时间戳的有序集合来实现对单个元素的过期操作。



[^分片]:  分片是一种将数据划分为多个部分的方法，对数据的划分可以基于键包含的ID、基于键的散列值，或者基于以上两者的某种结合。通过对数据进行分片，用户可以将数据存储到多台机器里面，也可以从多台机器里面获取数据，这种方法在解决某些问题时可以获得线性级别的性能提升。