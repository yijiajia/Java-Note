## Redis缓存的淘汰策略
Redis的淘汰策略，根据是否会进行数据淘汰可以把它们分成两类：
- 不进行数据淘汰的策略，只有 noeviction 这一种。
- 会进行淘汰的 7 种其他策略。

会进行淘汰的 7 种策略，我们可以再进一步根据淘汰候选数据集的范围把它们分成两类：
- 在设置了过期时间的数据中进行淘汰，包括 volatile-random、volatile-ttl、volatile-lru、volatile-lfu（Redis 4.0 后新增）四种。
- 在所有数据范围内进行淘汰，包括 allkeys-lru、allkeys-random、allkeys-lfu（Redis 4.0 后新增）三种。



<img src="https://static001.geekbang.org/resource/image/04/f6/04bdd13b760016ec3b30f4b02e133df6.jpg"/>

在redis3.0之前，默认是`volatile-lru`；在redis3.0之后（包括3.0），默认淘汰策略则是`noeviction`

### noeviction 策略

noeviction表示不淘汰数据，当缓存数据满了，有新的写请求进来，Redis不再提供服务，而是直接返回错误。

### 根据过期时间的淘汰策略
volatile-random、volatile-ttl、volatile-lru、volatile-lfu 四种策略是针对已经设置了过期时间的键值对。到键值对的到期时间到了或者Redis内存使用量达到了`maxmemory`阈值，Redis会根据这些策略对键值对进行淘汰；

* volatile-ttl 在筛选时，会针对设置了过期时间的键值对，根据过期时间的先后进行删除，越早过期的越先被删除。
* volatile-random 就像它的名称一样，在设置了过期时间的键值对中，进行随机删除。
* volatile-lru 会使用 LRU 算法筛选设置了过期时间的键值对。
* volatile-lfu 会使用 LFU 算法选择设置了过期时间的键值对。


### 所有数据范围内的淘汰策略

allkeys-lru、allkeys-random、allkeys-lfu 这三种策略淘汰的数据范围扩大到所有的键值对，无论这些键值对是否设置了过期时间，筛选数据进行淘汰的规则是：
* allkeys-random 策略，从所有键值对中随机选择并删除数据；
* allkeys-lru 策略，使用 LRU 算法在所有数据中进行筛选。
* allkeys-lfu 策略，使用 LFU 算法在所有数据中进行筛选。


### 关于LRU算法 
LRU算法即是最近最常使用算法，由于LRU会使用一个链表去维护使用的数据列表，当使用的数据越多，其移动元素时就会越耗时，这不可避免地会影响到Redis主线程。为此Redis对lru算法做了些简化。

LRU 策略的核心思想：如果一个数据刚刚被访问，那么这个数据肯定是热数据，还会被再次访问。

按照这个核心思想，Redis 中的 LRU 策略，会在每个数据对应的 RedisObject 结构体中设置一个 lru 字段，用来记录数据的访问时间戳。在进行数据淘汰时，LRU 策略会在候选数据集中淘汰掉 lru 字段值最小的数据（也就是访问时间最久的数据）。

所以，在数据被频繁访问的业务场景中，LRU 策略的确能有效留存访问时间最近的数据。而且，因为留存的这些数据还会被再次访问，所以又可以提升业务应用的访问速度。

具体做法是，在访问键值对时，redis会记录最近一次访问的时间戳。在redis决定淘汰数据时，会随机挑选N个数据，把它们作为一个候选集合，把最小的时间戳给筛选出去。当下一次要淘汰数据时，会挑选比第一次挑选的候选集合时间戳值要小的数据进入新的候选集合。当数据达到maxmemory-samples 时，将最小的值给淘汰掉。

通过该命令可以设置挑选的候选集合数
`CONFIG SET maxmemory-samples N`


### 使用建议

依据策略的特性，可以针对不同场景选择不同的策略去淘汰数据。
* 当缓存数据没有明显的冷热之分，即数据的访问频率差距不大，建议使用`allkeys-random` 随机策略淘汰数据；
* 当数据有明显的冷热之分，建议使用`allkeys-lru` 或者`volatile-lru` 算法，将最近最常访问的数据留在缓存数据中；
* 当业务中存在置顶需求，即不会过期的数据，这类一般不会设置过期时间，可以采用`volatile-lru`策略。这样这类数据就不会被淘汰，而其它数据可以根据lru规则进行淘汰。