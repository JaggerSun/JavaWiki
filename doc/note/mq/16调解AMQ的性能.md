## General techniques 通用技术。
     两个常用的方式： 非持久化和使用事务来一次性处理大量的消息。
    ### Persistent versus nonpersistent messages
持久化与非持久化消息
broker的消息在被消费之前会被一直持久化防止断电的情况发生。
非持久性的要快一点：

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-17.png)

 生产者异步发送消息，不需要等broker的收据
持久化数据到数据库比网络传输要慢

一般持久性都是为了防止宕机的。但是我们之前说了其实还可以配制成故障转移的方式，所以宕机不是很有问题。因此还是可以用这个方式的。但是默认是持久化的，需要手工设置。

MessageProducer producer = session.createProducer(topic);
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);

### Transactions 事务
    for (int i =0; i < 1000; i++) {
Message message = session.createTextMessage("message " + i);
producer.send(message);
if (i!=0 && i%10==0){
session.commit();
}
}
通过事务边界来批量处理。

### 嵌入式代理

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-18.png)

少了网络传输所以要快很多。
默认会给本地和远程的都发一个，可以通过cf.setCopyMessageOnSend(false);关闭复制。
http://www.mincoder.com/article/5946.shtml

###Tuning the TCP transport
socketBufferSize   用来接收和发送的缓存大小。
tcpNoDelay           

##优化消息生产者
### Asynchronous send 异步发送
我们说了能够通过设置非持久性来增加消费速度。 但是也可以通过设置异步发送来增加效率，既持久化过程和发送过程是异步的。
ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
cf.setUseAsyncSend(true);

### Producer flow control 生产流量控制
发生在消息阻塞的时候，消费者太慢了。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-19.png)

ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
cf.setProducerWindowSize(1024000);

## 优化消费者
### Prefetch limit 预取限制
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-20.png)

预取一定的数据到内部队列。一般值越大性能越好，但是有可能造成有的很忙，有的很闲。
设置成0的时候，则完全是consumer拉取数据，而没有brokerpush的过程。
ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
Properties props = new Properties();
props.setProperty("prefetchPolicy.queuePrefetch", "1000");
props.setProperty("prefetchPolicy.queueBrowserPrefetch", "500");
props.setProperty("prefetchPolicy.durableTopicPrefetch", "100");
props.setProperty("prefetchPolicy.topicPrefetch", "32766");
cf.setProperties(props);
Queue queue = new ActiveMQQueue("TEST.QUEUE?consumer.prefetchSize=10");
MessageConsumer consumer = session.createConsumer(queue);
慎重使用。 
## Delivery and acknowledgment of messages 消息的传递和确认
使用onMessage永远都比MessageConsumer.receive()
快，因为要维护一个队列等待receive方法，并且还会有线程的上下文切换。
为了预取， 将会使用akn的方式告诉broker这个消息由我消费。
ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
cf.setOptimizeAcknowledge(true);
可以一次确认多个，提高性能。
### Asynchronous dispatch
每一个回话会维护一个队列来发送消息给感兴趣的客户端。 这个可以关掉：
ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory();
cf.setAlwaysSessionAsync(false);

## Tuning in action 
演示一个实时的数据流入。
其生产者是一个嵌入式代理。
其消费者是一个远程监听。
BrokerService broker = new BrokerService();
broker.setBrokerName("fast");
broker.getSystemUsage().getMemoryUsage().setLimit(64*1024*1024);
PolicyEntry policy = new PolicyEntry();
policy.setMemoryLimit(4 * 1024 *1024);
policy.setProducerFlowControl(false);
PolicyMap pMap = new PolicyMap();
pMap.setDefaultEntry(policy);
broker.setDestinationPolicy(pMap);
broker.addConnector("tcp://localhost:61616");
broker.start();
设置内存限制， 禁用流量限制。

然后发送端设置非持久性和不保存副本。
ActiveMQConnectionFactory cf = new ActiveMQConnectionFactory("vm://fast");
cf.setCopyMessageOnSend(false);
Connection connection = cf.createConnection();
connection.start();
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Topic topic = session.createTopic("test.topic");
final MessageProducer producer = session.createProducer(topic);
producer.setDeliveryMode(DeliveryMode.NON_PERSISTENT);
for (int i =0; i < 1000000;i++) {
TextMessage message = session.createTextMessage("Test:"+i);
producer.send(message);
}

ActiveMQConnectionFactory cf =
new ActiveMQConnectionFactory("failover://(tcp://localhost:61616)");
cf.setAlwaysSessionAsync(false);
cf.setOptimizeAcknowledge(true);
Connection connection = cf.createConnection();
connection.start();
Session session = connection.createSession(false, Session.AUTO_ACKNOWLEDGE);
Topic topic = session.createTopic("test.topic?consumer.prefetchSize=32766");
MessageConsumer consumer = session.createConsumer(topic);
final AtomicInteger count = new AtomicInteger();
consumer.setMessageListener(new MessageListener() {
public void onMessage(Message message) {
TextMessage textMessage = (TextMessage)message;
try {
if (count.incrementAndGet()%10000==0)
System.err.println("Got = " + textMessage.getText());
} catch (JMSException e) {
e.printStackTrace();
}
}
});
消费端使用listener以及是有优化ack.
