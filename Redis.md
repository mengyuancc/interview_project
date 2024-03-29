1. 微信支付回调，防止重复消费

   - Redis

     记录上次回调的时间点，订单id为key,回调的时间戳为值，
     约定一个合理的时间间隔，比如5秒，如果小于5秒，则返回false

     

2. Redis的适用场景

   - string

     简单的`key-value`  ，可用作记录微博转发数、评论数、点赞数、视频网站播放数

   - list

     有序列表(底层是双向链表)，可做简单队列

   - set

     无序列表(去重)，用作记录观看记录

   - hash

     哈希表，存储结构化数据，可用作缓存系统

   - sortset

     有序集合映射(member-score)，可用作排行榜

     

3. Redis的底层数据结构

   > string：【int, embstr, raw】

   > hash：【ziplist, hashtable】

   > list：【ziplist, linkedlist】

   > set：【intset(整数集合)， hashtable】

   > zSet：【ziplist, skiplist】



4. 过期策略

   - 删除策略

     ```
     定时删除(对内存友好，对CPU不友好)到时间点上就把所有过期的键删除了。
     
     惰性删除(对CPU极度友好，对内存极度不友好)每次从键空间取键的时候，判断一下该键是否过期了，如果过期了就删除。
     
     定期删除(折中)每隔一段时间去删除过期键，限制删除的执行时长和频率。
     ```

   - Redis采用的删除策略

     Redis采用的是惰性删除+定期删除两种策略，所以说，在Redis里边如果过期键到了过期的时间了，未必被立马删除的！

     

5. 内存淘汰策略

   - 内存的最大使用量

     设置内存最大使用量，当内存使用量超出时，会施行数据淘汰策略*数据淘汰策略**

   - 数据淘汰策略

     - volatile-lru 从已设置过期时间的数据集中挑选最近最少使用的数据淘汰
     - volatile-til  从已设置过期时间的数据集中挑选将要过期的数据进行淘汰
     - volatile-random  从已设置过期时间的数据集中任意挑选数据进行淘汰
     - allkeys-lru 从所有数据集中挑选最近最少使用的数据淘汰
     - allkeys-random  从所有数据集中任意挑选数据进行淘汰
     - noeviction 禁止驱逐数据

   - 一般场景

     使用 Redis 缓存数据时，为了提高缓存命中率，需要保证缓存数据都是**热点数据**。可以将内存最大使用量设置为热点数据占用的内存量，然后启用allkeys-lru淘汰策略，将最近最少使用的数据淘汰



6. 缓存雪崩和缓存穿透

    - 缓存雪崩

      - 大量缓存设置相同的过期时间，导致缓存集中失效

        解决办法：设置缓存的过期时间时加上随机值

      - 缓存系统挂掉

        解决办法：

        ​	事发前预防：避免缓存系统挂掉，实现redis的高可用（主从架构+Sentinel 或者集群架构Redis Cluster）

        ​	事发中处理：设置**本地缓存(ehcache)+限流(hystrix)**，尽量避免我们的数据库被干掉

        ​	事发后处理：做好持久化，快速恢复服务

   - 缓存穿透

     - 请求不存在的数据，恶意攻击

       解决办法：缓存空数据

       ​					缓存所有已存在的key，使用布隆过滤器(BloomFilter)或者压缩filter进行提前拦截



7. 高并发场景-解决Redis缓存和Mysql数据一致性问题

   - 延迟双删策略

   - 异步更新缓存 (基于订阅binlog同步机制)

   - 利用mysql的触发器，当发生更新操作时，通过触发器更新相应的缓存

   - 参考

     [高并发问题 - 如何解决Redis缓存和MySQL数据一致性的问题](https://www.jianshu.com/p/61c6f30dc043)

     

8. 秒杀-解决超卖的问题

   1. 使用Redis分布式锁 set(key, value, NX,PX, 100) NX只有在键值不存在的情况下才会设置成功，PX设置缓存的失效时间

      防止死锁的产生

   2. 借用乐观锁的机制，使用版本号检测冲突
   
   3. 借助Redis实现预减库存
   
   

9. 分布式锁的实现

