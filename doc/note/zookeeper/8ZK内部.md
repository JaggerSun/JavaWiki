服务器分为了 leader 和 follower
leader处理所有的请求。  追睡着参与投票和接收状态变更。
还有第三者成为观察员Observer,观察者单纯的接收结果，不参加投票

查询请求被负载均衡，变更请求leader来做。

每个znode有zxid,自增的，每次请求都会有这个zxid和值。
zk保证这种所有的变更都是原子的，因为每一次都会进行一次proxy算法。

##Leader选举
选举一个leader出来。多数人同意。
新加入的先去学习，知道谁是leader.