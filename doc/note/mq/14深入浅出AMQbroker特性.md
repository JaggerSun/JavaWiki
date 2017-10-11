##通配符和符合Destination
  使用通配符订阅多个目的地，
  使用符合Destination发布消息到多个Destination.
###通配符
     Destination支持层级命名，使用.进行分割
        通配符使用三种特殊字符：. 用于分割层级
                * 匹配任意一个字符
                > 匹配一个或者全部的尾随元素
        比如：Topic allLeeds = session.createTopic("*.*.Leeds");
    ### 发送消息到多个Destinations
        可能需要同时发送一个消息给一个queue和一个topic.
        Queue ordersDestination = session.createQueue("store.orders, topic://
store.orders");

## 资讯信息  Advisory messages
 当接入或者是干啥的都会收到通知。
     这种通知被设置为一种系统的MQ消息。

	例子：  Topic connectionAdvisory = AdvisorySupport.CONNECTION_ADVISORY_TOPIC;
	MessageConsumer consumer = session.createConsumer(connectionAdvisory);
	ActiveMQMessage message = (ActiveMQMessage) consumer.receive();
	DataStructure data = (DataStructure) message.getDataStructure();
	if (data.getDataStructureType() == ConnectionInfo.DATA_STRUCTURE_TYPE) {
	ConnectionInfo connectionInfo = (ConnectionInfo) data;
	System.out.println("Connection started: " + connectionInfo);
	} else if (data.getDataStructureType() == RemoveInfo.DATA_STRUCTURE_TYPE) {
	RemoveInfo removeInfo = (RemoveInfo) data;

## JMS 虚拟主题
  想广播消息给消费者就用Topic. 想发消息给一个池的消费者就用queue.   但是想广播之后发给池来做运算目前没有满意的解决办法。
  topic的模式，一旦一个消费者死了，那么他就接收不到了，不会存在故障转移，并且也不能做负载均衡，但是queue的特性就可以。  我们尝试使用虚拟主题来避免这个问题。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-14.png)

消费者订阅的消息队列，是发送到topic的消息。
    topic的命名必须是这样的：VirtualTopic.<topic name>.
    Consummer queue的命名必须是这样的：Consumer.<consumer name>.VirtualTopic.<virtual topic name>
    具体的看这个类：VirtualTopics

## 追溯消费者
        快速消耗的消息建议关闭持久化。
        但这样有可能错过消息。为了能够追溯消息但又不用持久化AMQ提供了缓存配置，能够缓存一定数目和大小的消息。
        拢共分两步：
            1. 消费者告诉自己对追溯信息感兴趣。
            2. broker配置可以缓存多少信息。
        Topic topic =
session.createTopic("soccer.division1.leeds?consumer.retroactive=true");
MessageConsumer consumer = session.createConsumer(topic);
Message result = consumer.receive();
        broker部分设置属性
        <fixedSizedSubscriptionRecoveryPolicy maximumSize="8mb"/>
</subscriptionRecoveryPolicy>
        这样性能会高一些。

## 返还消息和死信队列  Message redelivery and dead-letter queues
        消息过去或者是没能够再投递的话，会进入死信队列。所以这些消息能够被消费或者是稍后在客户端浏览。
        通常会在如下的场景 向客户端再投递：
    客户端使用了事务， 且调用了rollback()方法
客户端使用了事务，在commit前调用了closes 方法。
客户端设置了CLIENT_ACKNOWLEDGE ，调用了recover（）方法。
怎么设置一个策略：
    RedeliveryPolicy policy = connection.getRedeliveryPolicy();
policy.setInitialRedeliveryDelay(500);
policy.setBackOffMultiplier(2);
policy.setUseExponentialBackOff(true);
policy.setMaximumRedeliveries(2);
默认的死信队列AcitveMQ.DLQ

没看懂这部分，暂时放弃。

## Extending functionality with interceptor plug-ins扩展插件的功能
### 可视化插件Visualization
### Enhanced logging 日志增强
### Central timestamp messages
with the timestamp interceptor plug-in 时间戳拦截插件
### Statistics 统计插件
    
## Routing engine with Apache Camel framework
