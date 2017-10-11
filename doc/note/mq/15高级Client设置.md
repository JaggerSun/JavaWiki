##  Exclusive consumers 独家消费者
    一个消费者接入broker的话永远都是先入先出。
    但是如果有多个消费者的话。最好的解决办法使只有一个消费者，但是还要支持故障转移。
    这个时候最好是AMQ能够有多个看这个queue,但是同时只有一个运行。
### 选择独家的信息消费者

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-15.png)

broker选择一个唯一消费者，只有故障之后才会选择别的人。
如果既有专有消费者也有普通的，那么会先选择专有的，所有专有的都挂了之后会变成普通的模式。
queue = new ActiveMQQueue("TEST.QUEUE?consumer.exclusive=true");
consumer = session.createConsumer(queue);

### Using exclusive consumers to provide a distributed lock 使用专用消费者提供一个分布式锁。
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-16.png)


### message groups
broker保证含有同一个GroupId的发送到同一个Consummer.

### Stream

### Blob

### Surviving network or broker failure with the failover protocol 故障转移
failover:(tcp://host1:61616,tcp://host2:61616,ssl://host2:61616)
connection.addTransportListener(能够监听段了链接这个事件。
都不活了就开始重试。 每次重试都是一个连续重试，如果连着两次都失败，那么将会等待一个倍数关系，比如第一次是2s,第二次将会等待4s。
initialReconnectDelay 两次连续失败的时间，backOffMultiplier指数。
failover:(tcp://master:61616,tcp://slave:61616)?\
backOffMultiplier=1.5,initialReconnectDelay=1000

### Scheduling messages to be delivered by ActiveMQ in the future 定时发送消息