zab协议： 
类似于proxy协议，但是proxy协议不保证先提交的提案先被接受，可能出现活锁，proxy协议后续的提案不能修改。
为了更好的使用zk使用了zab协议：
所有提案都转发到唯一的Leader， 既所有写请求都赚到leader上。leader保证请求的顺序
如果leader挂了就选择新的leader出来。

相比Paxos协议省略了Prepare阶段
图中的Follower对应Paxos协议中的Acceptor，Observer对应Paxos中的Learner

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/zookeeper/zk-8.png)

zk的角色
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/zookeeper/zk-9.png)

设计特点：
1.最终一致性：client不论连接到哪个Server，展示给它都是同一个视图，这是zookeeper最重要的性能。

2 .可靠性：具有简单、健壮、良好的性能，如果消息m被到一台服务器接受，那么它将被所有的服务器接受。

3 .实时性：Zookeeper保证客户端将在一个时间间隔范围内获得服务器的更新信息，或者服务器失效的信息。但由于网络延时等原因，Zookeeper不能保证两个客户端能同时得到刚更新的数据，如果需要最新数据，应该在读数据之前调用sync()接口。

4 .等待无关（wait-free）：慢的或者失效的client不得干预快速的client的请求，使得每个client都能有效的等待。

5.原子性：更新只能成功或者失败，没有中间状态。

6 .顺序性：包括全局有序和偏序两种：全局有序是指如果在一台服务器上消息a在消息b前发布，则在所有Server上消息a都将在消息b前被发布；偏序是指如果一个消息b在消息a后被同一个发送者发布，a必将排在b前面。


http://cailin.iteye.com/blog/2014486/
