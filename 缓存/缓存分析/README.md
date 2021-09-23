set

expire

setex 原子性操作命令

keys 

> 因为Redis使用了IO多路复用模式
>
> keys命令会扫描整个Redis里面的数据进行模糊匹配，会造成线程阻塞

scans

> 不会阻塞线程



del



hset

hget

hdel

hkeys key



s



缓存穿透

> 客户端请求无效的数据，而缓存中没有，直接打到了mysql中
>
> 接口层做校验
>
> 将null值写到redis中（推荐）
>
> 布隆过滤器
>
> IP地址做黑名单



雪崩

> Redis key大面积失效导致请求打到数据库
>
> 动态增减Redis节点时
>
> 一致性Hash算法： 解决数据分布不均匀，数据倾斜等问题，造成一个某个节点压力过大 
>
> TreeMap来实现  环的形式



缓存击穿

> 某个热点key大面积失效，大量请求打到数据库中
>
> 分布式锁setex







