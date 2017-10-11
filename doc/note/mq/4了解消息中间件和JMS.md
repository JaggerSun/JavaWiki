## 说明企业消息
消息中间件是发送者和接收者的信息中介。
提供异步的松散耦合的，可靠的，可扩展的，安全的通信
分布式应用程序或系统之间的方式。

信息通过消息中间件从一个应用传递给另外一个应用。这样就使得发送者和接收者解耦。
因为不限制同时发送和接收，发送者和接收者互相不认识，这样成为异步消息传递。
消息中间件还提供了消息持久性，复杂的消息路由，消息转换。
持久性防丢和排错。
复杂的路由有很广阔的想象。
消息转换，允许两个应用通过同样的格式通讯。

Active MQ提供了上面所有的支持。 

## 什么是JMS
各种消息中间件使用不同的API对外提供服务，知道JMS来了

目的是提供Java程序发送和接收消息的标准话的API。
最大限度的减少Java程序员开发复杂通讯程序所需要的知识，同时提供一定的可扩展性
类似于JDBC是一种高抽象的接口。
1998年开始2002年JMS1.1。提供了如下的概念：
1. JMS client  Java开发的发送和接受消息的程序
2. Non-JMS client    使用JMS provider提供的原生的客户端，而不是Java开发的。
3. JMS producer  创建和发送消息的程序
4. JMS consumer  接收和处理消息的程序
5. JMS provider  JAVA开发的JMS协议的实现
6. JMS message  被JMS Client和producer发送和接收的消息。
7. JMS domains 消息类型。 点对点和发布/订阅模式
8. Administered objects 能够通过JNDI访问的一些预制的配置数据
9. Connection factory  从工厂得到到JMS提供者的链接
10. Destination 消息从那里收，消息发到哪里去。  

##JMS规范
###JMSclient
    客户端通过JMS API跟JMS 提供者进行通讯， 类似于JDBC。
     但是JMS provider一般不是存的JMS的，一般都是看上了他的额外的功能。因此在换中间件的时候有可能带来额外的工作。
     分为了发送消息的生产者，和接收消息的消费者。 有可能兼具有二者的功能
    1. JMS Producer
        向目的地发送消息。用Session.createProducer() 创建，被MessageProducer.send()覆盖
    2. JMS Consumer
         从目的地同步receive的，或者异步的MessageListener.onMessage()接收消息。 
###JMS Provider JMS Api的实现者

###The JMS message 
    是JMS规范中最重要的概念。允许文本，二进制数据作为体，以及头，其消息格式如图：
包含报头和payload。  被设计成灵活和易于理解，唯一复杂的地方在于头信息

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-1.png)

大部分自动设置，  可以是标注的JMS或者是JMS Provider提供的方法

 - JMSDestination 
 - JMSDeliveryMode
传递方式， 支持两种：持久性和非持久性。
持久性：让provider持久化消息。  牺牲性能，换取可靠性，在provider宕机之后不会丢失消息
非持久性，保证非持久化并且传递且只传递一次消息。 性能更好。
设置在provider对所有都生效。  但是会被单独消息的设置覆盖
 - JMSExpiration
消息过期时间。  producer.settimetolive方法设定所有该producer发出的，或者 MessageProducer.send()中设定该次发出的。  JMS使用当前时间+timetolive的设置来得到过期时间，因此如果时间设置的有问题，是有可能发不出去消息的。
默认是永不过期。 
 - JMSMessageID
provider指定的唯一标示， 可以用在存储中。 因为生成这个有一定的开销，可以通过producer.setdisablemessageid来建议provider忽略，但是仅仅是个建议。
 - JMSPriority
producer设置，优先级，10个级别，可以被单个消息的覆盖。  跟java线程的优先级类似
 - JMSTimestamp
JMS producer发出消息的时间，可以建议忽略，且为空值

jmsClient设置的：
 - JMSCorrelationID  
通常用来关联多个Message。例如需要回复一个消息，可以把JMSCorrelationID设置为所收到的消息的JMSMessageID
 - JMSReplyTo
有时消息生产者希望消费者回复一个消息，JMSReplyTo为一个Destination，表示需要回复的目的地。当然消费者可以不理会它
 - JMSType
Headers set optionally by the JMS provider:
 - JMSRedelivered
消息重发，没有收到nok

###JMS 属性
    头中的附件参数。getpropertynames()得到一个枚举  分为了：自定义属性，JMS定义属性，provider定义属性。
    自定义属性：
        使用getbooleanproperty() /
setbooleanproperty()，getstringproperty() / setstringproperty()等方法任意定义。
    JMS定义属性：
        JMSX开头的一些属性。有很多，用到再说
   provider属性：
        JMS_<vendor-name>
        供应商自定义的。

###Message selectors
  有时客户端订阅到一个目的地，但是只想接收指定的部分消息。 这样可以每个消息都带有一个标示，然后客户端可以告诉provider只接收这样标示的消息。
  只能设置消息头和属性


    public void sendStockMessage(Session session,
	MessageProducer producer,
	Destination destination,
	String payload,
	String symbol,
	double price)
	throws JMSException {
	TextMessage textMessage = session.createTextMessage();
	textMessage.setText(payload);
	textMessage.setStringProperty("SYMBOL", symbol);
	textMessage.setDoubleProperty("PRICE", price);
	producer.send(destination, textMessage);
	}
	
	String selector = "SYMBOL = 'AAPL'";
	MessageConsumer consumer =
	session.createConsumer(destination, selector);
	...

###消息体
   六种类型消息体：payload

 - Message  base类，包含了head和属性
 - TextMessae  payload是String
 - MapMessage
 - ByteMessage
 - StreamMessage
 - ObjectMessage

###JMS domains
  支持两种方式：point-to-point and publish/subscribe

####点对点
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-2.png)

 - 消息一次且仅一次传递给一个消费者
 - 同步使用MessageConsumer.receive(),异步使用MessageConsumer setmessagelistener()
 - 队列存储消息直到发出或者到期
 - 多个用户可以注册到同一个队列，会发送给一个消费者，并得到应答，是循环分配的

####发布订阅模式
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-3.png)

 - 发送消息到主题，所有订阅者都会收到
 - 也有异步同步两种
 - 可以持久订阅，当时短线的消费者会产生挤压，等连上之后又会继续消费掉
持久性与持久化 DURABILITY PERSISTENCE
  有点差别，持久性是订阅的一个属性，仅有发布\订阅模式才有，仅仅用于离线后是否仍然记录那些消息没有发给订阅者

####请求应答模式
![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-4.png)
虽然协议里没有，但是确实提供了一些消息头能够处理基本的请求/应答模式

JMSReplyTo 指定 应答需要发送的目的地。
应答的消息的JMSCorrelationID 指定为请求消息的JMSMessageID 这样把1请求和应答关联起来， 有便利的类:QueueRequestor

###Administered objects
管理对象。 本来应该由JMS管理员创建的一些包含特定的provider提供的配置信息。
  用来隐藏特定的供应商的细节，有点像JNDI，或者dubbo.只关心接口不关心实现。
  JMS定义了两种这种对象：ConnecitonFactory和Destination。

 - ConnecitonFactory
   client 通过这个创建到provider的链接。 这个链接可能会是比较大。所以使用一个连接池会是比较好的选择。 一个Connection跟JDBC里的Connection差不多。是一个操作数据库的接口。
   JMS Connection 用来创建JMS session与provider提供交互。
 - Destination
封装了provider特定的消费和发送地址。虽然由session创建但是生命周期跟connection相关。
临时的destination的创建目的是用在请求/应答模式。该连接关闭就没了。
