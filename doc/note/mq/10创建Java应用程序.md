![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-21.png)

##使用Java嵌入ActiveMq
###使用brokerService嵌入AMQ
    直接看AuthBrokerService的代码
    使用用用户名密码的时候connection = factory.createConnection(username, password);
###使用brokerFactory
System.setProperty("activemq.base", System.getProperty("user.dir"));
String configUri =
"xbean:target/classes/org/apache/activemq/book/ch6/activemq-simple.xml"
URI brokerUri = new URI(configUri);
BrokerService broker = BrokerFactory.createBroker(brokerUri);
broker.start()
## 使用Spring
    上面都是说怎么启动一个Broker的，感觉用到的场景比较少，暂时不管。

##适应JMS实现请求和应答
    之前都是异步的， 并且通常这种都是client-server的结构使用同步接口，但是这样影响了扩展性，现在我们尝试另外的办法。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-22.png)
两边都有生产者和消费者，通过两个队列进行沟通。
流程：
1. client 设置JMSCorrelationID,和JMSReplyTo 为接收的destination. 然后监听着destination来等待结果。
2. woker 收到request之后，继续把JMSCorrelationID赋值进去，然后把计算结果放到JMSReplyTo中去。完成计算
3. Client收到结果后，通过correlation知道是不是自己的请求，然后进行处理。
例子在ch7里面有。

##通过Spring使用JMSclient
跟平常项目里用的差不多，
但是有poolConnectionFactory和CachingConnectionFactory. 具体的区别，等以后细看Spring-jms的时候再说。
