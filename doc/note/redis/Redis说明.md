下载路径：http://redis.io/download
有2.8和3.0，  两个可以根据情况都用


redis 是一个高效的key-value的内存数据库。支持多种数据类型。  有丰富的操作。   可以做主从提高可用性。可以客户端一致性hash集群，  3.0支持服务器端集群。  支持持久化  

redis命令详解：
http://redisdoc.com/
Key:
    DUMP, RESTORE 序列化与反序列化
    DEL 删除键 多个用空格隔开
    EXISTS  存在返回1， 不存在返回0
    EXPIRE, EXPIREAT, PERSIST, TTL
    PEXPIPE,PEXPIREAT, PTTL毫秒单位
        EXPIRE  设置生存时间，单位s.
                RENAME不会影响生存时间。
                之前的版本时效性1s作用，现在1ms
                再次执行EXPIRE就可以更新生存时间
                模式：导航会话。navigation session
                            用户最近访问页面记录叫导航会话，假设我们需要通过用户的最近浏览来分析推荐物品，为了减少分析链接和时效性。可以在用户每次访问页面的时候   
                            PUSH userid http://sku.
                            EXPIRE userid 60. 
                            这样就记录了用户1分钟内的访问信息
        EXPIREAT  不是设置过多少s过期，设置到哪个时间过期
        TTL 查看剩余生存时间
        PERSIST    移除一个key的生存时间
    KEYS pattern 查询符合模式的key. 效率高，但是还是有可能造成性能问题，  如果需要从一个数据集中查找特定的key,最好还是用Set来作为这个数据结构，来查找Key
    MIGRATE host port key destination-db timeout  从本实例迁移到目的数据库中
            是本地DUMP， 远程RESTORE, 本地DEL这三个命令的合体，原子操作，会阻塞实例。
    MOVE key db 移动到指定的库总，如果已经存在则忽略。  当做锁原语，  (意思是并发的时候都执行这个请求，只有第一次的会返回1?)
    OBJECT  ENCODING <key> 内部表示，比如列表可以是ziplist，linkedlist等
          IDLETIME <key>  key的空闲时间 
    RANDOMKEY 随机key, 不存在为nil
    RENAME key newkey key不存在或者key与newkey相同返回错误， newkey已存在覆盖
    RENAMENX key newkey newkey已经存在返回0
    SORT 有点复杂，用到再说
    TYPE key 返回类型string,list,set,zset,hash
  SCAN 增量迭代SSCAN Set， HSCAN 哈希 ZSCAN有序集合迭代  SCAN 0开头，  返回两个元素的数组，第一个元素是SCAN 迭代的开始至，第二个是数据集， 知道第一个值为0时停止，成为一次完整遍历。  count 可以限制每次迭代的值
  
String
    APPEND
    SET SETEX PSETEX SETNX
        SET key value
        SET key value EX second  == SETEX 
        SET key value PX ms == PSETEX
        SET key value NX 只有键不存在时才执行操作，键存在返回nil
        SET key value XX 键存在才执行设置操作
        模式：锁的实现
        1. set key anystring NX EX 60 返回OK则获得锁，否则被其他资源占用
        2. 进行操作。
        3. DEL命令释放锁
            更健壮的锁，可以防止持有过期锁的客户端误删现有锁的情况出现
            1. set 一个随机字符串值作为口令
            2. 操作
            3. 使用一个Lua脚本来对键进行删除
                if redis.call("get", KEYS[1]) == ARGV[1]
                then
                    return redis.call("del", KEYS[1])
                else 
                    return 0
                end
    SETBIT GETBIT  设置字符串的第几位，之前的位如果不存在会设置未0
    BITCOUNT 字符串中为1位的数量，
    BITOP BITOP AND,OR,NOT,XOR  对一个或多个保存了二进制位的字符串key进行位操作    
    SETRANGE key offset value 从偏移量开始覆盖
    INCR, DECR, DECRBY, INCRBY自增自减，原子操作，类型不对的返回错误
            模式，计数器
                比如点击量，要记录wzj在2015-05-20的点击量， 可以INCR wzj-2015-05-20命令来算
                扩展使用，可以搭配使用INCR，DECR等实现计分器
                配合GETSET命令实现复位计数器
                
            模式，限速器
                限制公开API请求次数，比如限制某个IP地址每秒钟是个请求
                实现办法是get key 如果存在且大于10就返回错误，如果不存在则设置为1，设置超时时间，如果存在，则自增。
                http://redisdoc.com/string/incr.html
    GETRANGE 子串
    GETSET key value ,设置新值返回旧值
        模式： 复位操作的计数器
            如果不用GETSET mycount 0这个命令，则需要使用get mycount;set mycount 0两个操作，还要使用事务
    INCRBYFLOAT 自增浮点数
    MSET,MGET,MSETNX
            批量设置或者获取key. NX要求所有key都不存在才返回1， 
    STRLEN 返回key所存储的字符串的长度
    
Hash 哈希表
    HSET key field value, HSETNX
    HGET key field
    HGETALL 返回所有预和值 返回的是一个列表，  域，值，域，值，这样的结构
    HEXIST field给定的预是否存在
    HDEL key field.. 删除一个或者多个域，不存在被忽略
    HLEN 返回域数量
    HVALS 返回Hash中所有域的值
    HKEYS 返回所有域
    HSCAN 增量迭代 
    HMGET key field.. 返回多个域的值
    HMSET key field value field value 一次设置多个key, value
    HINCRBY key field increment(可正可负)
    HINCRBYFLOAT key field increment 浮点数
    
List
    LPUSH key value.. 将一个或者多个值插入到列表的表头，注意从左边插入
    LPOP 移除并返回列表key的头元素  
    LPUSHX key value 当且仅当key存在是一个列表的时候插入
    LINDEX key index 返回下标为index的元素，0为底，越界nil
    LINSERT key BEFORE|AFTER pivot value 在pivot的前面或者后面插入value
    LLEN    key 返回列表的长度
    LRANGE   key start stop 返回之前的数0为底，含这两个数
    LREM key count value 从表头或者表尾开始移除跟value相等的值
    RPOP RPUSH RPUSHX
    RPOPLPUSH  source destination 将source的尾弹出给客户端，并且将尾放到des的头
        模式：安全队列
            可以作为一个待处理的安全的任务队列。
            普通的方式是一个客户端在头插入，另外一个客户端从尾获取，实现生产消费这样的模式，但是这样是不安全的，可以一个客户端取出之后没有处理就崩溃了，使用这个命令可以做一个备份，在处理完成后去备份中删除，但是如果出了问题，备份中的就是没有处理的。
            还可以专门写一个客户端程序监控备份表，定期把任务返回消息队列中去。
        模式：循环列表
            source 和des用同一个key，这样很适合做监控。比如服务器的监控的一些网站做心跳，就可以用这种办法。
    BLPOP key... timeout LPOP的阻塞原语， 当没有值可以弹出时阻塞,多个key时会依次监测, BRPOP BRPOPLPUSH
        多客户端阻塞同一个key，有值之后会给第一个阻塞客户端值
        在MULTI/EXEC中的时候，因为事务命令会阻塞整个服务器，所以其他客户端不能PUSH，这样就肯定会超时，因此redis处理的时候会把它降为LPOP来处理
        模式：事件提醒
            普通的做法是轮询，更好的是用这个原语来阻塞。
            但是如果是其他的类型比如Set就做不到，因此可以搭配这个命令，让他在另外的key上帮助SPOP完成阻塞效果，大概的内容：
                消费端
                while(true){
                    if(SPOP(key)){
                        BRPOP helper_key;
                    }
                }
                生产端：
                MULTI
                    SADD key element
                    LPUSH helper_key x
                EXEC
Set集合
    SADD  添加一个或者多个元素到key中，如果key不是Set类型报错
    SCARD 返回集合中元素数量
    SDIFF 差集
    SDIFFSTORE deskey key [key1] 求key,key1的差集，并保存/覆盖deskey.
    SINTER, SINTERSTORE 交集
    SUNION,SUNIONSTORE 并集
    SISMEMBER key member 判断member是否集合key的成员
    SMEMBERS key 返回所有元素， 不存在的key视为空集合
    SMOVE source des member 将member从source移动到des
            原子操作
            如果des中存在，则单存的从source中删除
    SPOP 获取且删除一个随机元素，想不被删除用SRANDMEMBER,想有序用有序集合
    SRANDMEMBER 随机获取一个元素，不删除， 如果跟了数量则会返回一个不重复的结合
    SREM key member 移除一个或者多个member
   
SortedSet 有序集合
    ZADD key score member ... 将一个或者多个score member放到有序集key中 score是用来排序的分值
    ZCARD key 返回基数
    ZOUNT key min max score在mix和max之前的数量
    ZINCRBY key increment member 为key的member的score值增加increment
    ZREVRANGE key start stop [WITHSCORES] 返回下标在指定区间的有序集，按score从大到小
    ZRANGE 从小到大
    ZREVRANGEBYSCORE key max min score在区间内的有续集
    ZRANGEBYSCORE 
    ZREVRANK key member 返回从大到小member的排名 从0开始
    ZRANK
    ZREM key member... 移除一个或者多个成员
    ZREMRANGEBYRANK 移除指定区间内的
    ZRENRANGEBYSCORE 移除score在指定区间内的
    ZSCORE 返回score的值
    ZUNIONSTORE    destination numkeys key... 
            求多个的并集，key的数量为numkeys, 同一个member的score值相加
            WEIGHTS 可以指定每个有序集的权重，即其score在并集的时候会乘以这个权重
            AGGREGATE 默认SUM，还有MIN等，指定并集之后相同member的score怎么处理
     ZINTERSTORE 交集
     ZSCAN
    ZRANGEBYLEX ZLEXCOUNT ZREMRANGEBYLEX
        同样的score是，会按字典排序，上面三个是返回，数量和移除字典序后的在member在一个区间的内容

HyperLogLog
    感觉用处不大，掠过

Pub/Sub 发布订阅
    subscribe   channel 订阅一个频道，如果有其他客户端对这个频道，pub则会收到信息
    unsubscribe 退订上面命令
    publish channel message 向指定频道发消息
    PSUBSCRIBE    pattern 订阅匹配模式的所有频道
    PUBSUBSCRIBE    pattern 退订所给定的模式，或者是退订所有
    PUBSUB 查询命令 +CHANNEL 是查看所有订阅超过一个的频道，还有其他的

Transaction 事务
    
    


    
    
     
            sa