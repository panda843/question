|类型	|简介	|特性	|场景|
|   ------ |   ------  |   ------  |   ------  |
|String(字符串)|	二进制安全	|可以包含任何数据,比如jpg图片或者序列化的对象,一个键最大能存储512M	|---|
|Hash(字典)|	键值对集合,即编程语言中的Map类型|	适合存储对象,并且可以像数据库中update一个属性一样只修改某一项属性值(Memcached中需要取出整个字符串反序列化成对象修改完再序列化存回去)	|存储、读取、修改用户属性|
|List(列表)|	链表(双向链表)|	增删快,提供了操作某一段元素的API	|1,最新消息排行等功能(比如朋友圈的时间线) 2,消息队列|
|Set(集合)|	哈希表实现,元素不重复|	1、添加、删除,查找的复杂度都是O(1) 2、为集合提供了求交集、并集、差集等操作	|1、共同好友 2、利用唯一性,统计访问网站的所有独立ip 3、好友推荐时,根据tag求交集,大于某个阈值就可以推荐|
|Sorted Set(有序集合)|	将Set中的元素增加一个权重参数score,元素按score有序排列	|数据插入集合时,已经进行天然排序|1、排行榜 2、带权重的消息队列|
|HyperLogLog(HYLL)|优点是，在输入元素的数量或者体积非常非常大时，计算基数所需的空间总是固定 的、并且是很小的。	|每个 HYLL 键只需要花费 12 KB 内存，就可以计算接近 2^64 个不同元素的基 数。这和计算基数时，元素越多耗费内存就越多的集合形成鲜明对比。|用来做基数统计 误差为(0.81%)|
|BitMap(位图)|对于bitmap，我们可以很容易的进行and、or操作，而且效率也是非常高的|对于bitmap的内存占用则是一个比较大的问题，它取决于基数的上限而非元素的个数。比如你的基数是1000W，那么你将需要分配1.2MB左右的内存空间给bitmap而不管你是否仅存了一个元素|可进行数据的快速查找，判重，删除|

#Sset命令
|命令|描述|
|   ------  |   ------  |
|ZADD key score1 member1 [score2 member2] |向有序集合添加一个或多个成员，或者更新已存在成员的分数|
|ZCARD key |获取有序集合的成员数|
|ZCOUNT key min max |计算在有序集合中指定区间分数的成员数|
|ZINCRBY key increment member |有序集合中对指定成员的分数加上增量 increment|
|ZINTERSTORE destination numkeys key [key ...] |计算给定的一个或多个有序集的交集并将结果集存储在新的有序集合 key 中|
|ZLEXCOUNT key min max |在有序集合中计算指定字典区间内成员数量|
|ZRANGE key start stop [WITHSCORES] |通过索引区间返回有序集合成指定区间内的成员|
|ZRANGEBYLEX key min max [LIMIT offset count] |通过字典区间返回有序集合的成员|
|ZRANGEBYSCORE key min max [WITHSCORES] [LIMIT] |通过分数返回有序集合指定区间内的成员|
|ZRANK key member |返回有序集合中指定成员的索引|
|ZREM key member [member ...] |移除有序集合中的一个或多个成员|
|ZREMRANGEBYLEX key min max |移除有序集合中给定的字典区间的所有成员|
|ZREMRANGEBYRANK key start stop |移除有序集合中给定的排名区间的所有成员|
|ZREMRANGEBYSCORE key min max |移除有序集合中给定的分数区间的所有成员|
|ZREVRANGE key start stop [WITHSCORES] |返回有序集中指定区间内的成员，通过索引，分数从高到底|
|ZREVRANGEBYSCORE key max min [WITHSCORES] |返回有序集中指定分数区间内的成员，分数从高到低排序|
|ZREVRANK key member |返回有序集合中指定成员的排名，有序集成员按分数值递减(从大到小)排序|
|ZSCORE key member |返回有序集中，成员的分数值|
|ZUNIONSTORE destination numkeys key [key ...] |计算给定的一个或多个有序集的并集，并存储在新的 key 中|
|ZSCAN key cursor [MATCH pattern] [COUNT count] |迭代有序集合中的元素（包括元素成员和元素分值）|

# Set命令
|命令|描述|
|   ------  |   ------  |
|SADD key member1 [member2] |向集合添加一个或多个成员|
|SCARD key |获取集合的成员数|
|SDIFF key1 [key2] |返回给定所有集合的差集|
|SDIFFSTORE destination key1 [key2] |返回给定所有集合的差集并存储在 destination 中|
|SINTER key1 [key2] |返回给定所有集合的交集|
|SINTERSTORE destination key1 [key2] |返回给定所有集合的交集并存储在 destination 中|
|SISMEMBER key member |判断 member 元素是否是集合 key 的成员|
|SMEMBERS key |返回集合中的所有成员|
|SMOVE source destination member |将 member 元素从 source 集合移动到 destination 集合|
|SPOP key |移除并返回集合中的一个随机元素|
|SRANDMEMBER key [count] |返回集合中一个或多个随机数|
|SREM key member1 [member2] |移除集合中一个或多个成员|
|SUNION key1 [key2] |返回所有给定集合的并集|
|SUNIONSTORE destination key1 [key2] |所有给定集合的并集存储在 destination 集合中|
|SSCAN key cursor [MATCH pattern] [COUNT count] |迭代集合中的元素|


# List命令
|命令|描述|
|   ------  |   ------  |
|BLPOP key1 [key2 ] timeout |移出并获取列表的第一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|BRPOP key1 [key2 ] timeout |移出并获取列表的最后一个元素， 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|BRPOPLPUSH source destination timeout |从列表中弹出一个值，将弹出的元素插入到另外一个列表中并返回它； 如果列表没有元素会阻塞列表直到等待超时或发现可弹出元素为止。|
|LINDEX key index |通过索引获取列表中的元素|
|LINSERT key BEFORE 或 AFTER pivot value |在列表的元素前或者后插入元素|
|LLEN key |获取列表长度|
|LPOP key |移出并获取列表的第一个元素|
|LPUSH key value1 [value2] |将一个或多个值插入到列表头部|
|LPUSHX key value |将一个值插入到已存在的列表头部|
|LRANGE key start stop |获取列表指定范围内的元素|
|LREM key count value |移除列表元素|
|LSET key index value |通过索引设置列表元素的值|
|LTRIM key start stop |对一个列表进行修剪(trim)，就是说，让列表只保留指定区间内的元素，不在指定区间之内的元素都将被删除。|
|RPOP key |移除列表的最后一个元素，返回值为移除的元素。|
|RPOPLPUSH source destination |移除列表的最后一个元素，并将该元素添加到另一个列表并返回|
|RPUSH key value1 [value2] |在列表中添加一个或多个值|
|RPUSHX key value |为已存在的列表添加值|

# Hash命令
|命令| 描述|
|   ------  |   ------  |
|HDEL key field1 [field2] |删除一个或多个哈希表字段|
|HEXISTS key field |查看哈希表 key 中，指定的字段是否存在。|
|HGET key field |获取存储在哈希表中指定字段的值。|
|HGETALL key |获取在哈希表中指定 key 的所有字段和值|
|HINCRBY key field increment |为哈希表 key 中的指定字段的整数值加上增量 increment 。|
|HINCRBYFLOAT key field increment |为哈希表 key 中的指定字段的浮点数值加上增量 increment 。|
|HKEYS key |获取所有哈希表中的字段|
|HLEN key |获取哈希表中字段的数量|
|HMGET key field1 [field2] |获取所有给定字段的值|
|HMSET key field1 value1 [field2 value2 ] |同时将多个 field-value (域-值)对设置到哈希表 key 中。|
|HSET key field value |将哈希表 key 中的字段 field 的值设为 value 。|
|HSETNX key field value |只有在字段 field 不存在时，设置哈希表字段的值。|
|HVALS key |获取哈希表中所有值|
|HSCAN key cursor [MATCH pattern] [COUNT count] |迭代哈希表中的键值对。|