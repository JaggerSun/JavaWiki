提供通讯的接触组件。
Activermq提供了connectors允许client-to-broker以及broker之间进行通讯。
或者定义更复杂的通讯。
从一下两个方面来作介绍配置连接器不同的属性和使用不同的连接器。

##Understanding connector URIs
URI的结构<scheme>://<authority><path><?query>
比如：http://www.nabble.com/forum/NewTopic.jtp?forum=2356
activemq，用他来定义不同的连接器
比如：tcp://localhost:61616  使用tcp协议访问host的哪个接口

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-23.png)

除此之外还有符合的uri的使用，注意配置不能有空格
failover:(tcp://10.24.19.239:61616,tcp://10.24.36.54:61616)  
这个是做故障转移和自动重连的

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-24.png)

##传输连接器
client和borker之前的传递消息使用传输连接器
###配置传输连接器(Transprot connec)
activemq.xml
<transportConnectors>
            <!-- DOS protection, limit concurrent connections to 1000 and frame size to 100MB -->
            <transportConnector name="openwire" uri="tcp://0.0.0.0:61616?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="amqp" uri="amqp://0.0.0.0:5672?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="stomp" uri="stomp://0.0.0.0:61613?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="mqtt" uri="mqtt://0.0.0.0:1883?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
            <transportConnector name="ws" uri="ws://0.0.0.0:61614?maximumConnections=1000&amp;wireFormat.maxFrameSize=104857600"/>
        </transportConnectors>    
这个就是定义传输链接器的
通过不同的协议，监听在不同的端口启动的时候会有如下样式的日志：
INFO TransportConnector - Connector openwire Started
INFO TransportServerThreadSupport - Listening for connections at:
ssl://localhost:61617
在Client来说， uri是用来跟一个broker之间创建链接来发送和接收消息的。比如：
ActiveMQConnectionFactory factory =
new ActiveMQConnectionFactory("tcp://localhost:61616");
Connection connection = factory.createConnection();
connection.start();

##配置各种uri
| protocol        | 描述  |
| ------------- | -----:|
| TCP      | 默认的使用最广 |
| NIO  |   通常为了更好的可扩展性 |
| UDP |  更快   |
| SSL | 安全   |
| HTTP(S) | 为了跨防火墙 |
| VM | broker，comsumer和producer都在同一个Java虚拟机 |

###TCP
例子：
<transportConnectors>
<transportConnector name="tcp"
uri="tcp://localhost:61616?trace=true"/>
</transportConnectors>
特点：
   效率 字节流效率比较高
可靠性 保证不会丢包
可用性  广泛很早开始使用
### NIO
    使用一些现代网络特性。 最显著的作用是： 选择器和非阻塞的IO， 能够使用相同的资源来提高并发。 使用起来跟用TCP没有什么区别，仅仅是在传输的实现上有区别。更适合作如下的事情：
    有大量的Client要连接broker. 因为默认情况下，能够支持的并发跟操作系统线程数相关，而NIO需要的线程更少，支持更高的并发。
 有大量的数据需要传输，比TCP一般会提供更好的性能
对于性能的提升上，还有broker拓扑，设置broker属性等多种方式
nio://192.168.64.19:61616  

### UDP
 与TCP的区别：
        TCP面向流，不存在丢包和重复包， UDP面向包，有可能丢包或者重复
        TCP基于一个活的连接，确保传输可靠性。UDP是无连接协议。

TCP一般使用在需要高可靠性的场景， 而UDP使用高效率的场景
udp://192.168.64.19:61616  
优势：
        broker如果在防火墙的后面,关闭了TCP，那么就只能用UDP了
时间敏感，而不是可用性敏感
注意会丢包

### Secure Sockets Layer Protocol 

###Hypertext Transfer Protocol (HTTP/HTTPS)
就为了绕过防火墙

### Connecting to ActiveMQ inside the virtual machine(VM connector)
JAVA 启动的嵌入式的broker,并且进行连接。
vm://broker1?marshal=false&broker.persistent=false 
嵌入式的启动broker. 然后虚拟机内部使用vm进行通信不开启网络连接。
broker1是broker的名称，是唯一的标示

还有另外一种语法配置嵌入式的borker.这种既能够内部自动通信，又能够外部通信。 外部通信使用正常的tcp即可。 内部使用vm来通信
vm:broker:(tcp://localhost:6000)?brokerName=embeddedbroker&persistent=false

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-25.png)

此外，还可以使用配置文件来创建broker:
vm://localhost?brokerConfig=xbean:activemq.xml
### Network connectors
可以是集群等的各种拓扑结构
 负责borker之前通信的。
一种是单向的，负责转发他收到的消息到另外一个borker, 通常叫做转发桥。
此外还支持双向的，一个connection既可以发送也可以接收，这种叫做双工连接器。

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-26.png)

图中既有双工，也有单工。

还有另外一个概念：发现。broders想要发现周边所有的或者的broder.

有两种配置方式： 多播发现式和知道详细网络情况的静态配置式。

###Static Network
需要知道broker的位置

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-27.png)

需要配置的地方：
正常启动一个broker   activemq start
在找一个配置 jetty.xml配置<property name="port" value="8261"/>
activemq.xml  <transportConnector name="openwire" uri="tcp://0.0.0.0:61617
再增加：
<networkConnectors>
			<networkConnector uri="static:(tcp://localhost:61616)" />
		</networkConnectors>
好了把2也启动起来，然后给2发消息。 但是消费1的消息来看效果
测试成功。。。。
INFO | Network connection between vm://localhost#0 and tcp://localhost/127.0.0.
:61616@61418 (localhost) has been established.
这个是启动信息，标示连接到了服务器上
这样的配置，可以把请求平分到多个broker上，提高性能
一个应用场景： 比如一个中心，然后多个地域的这种部署，比如只能家居，如果是所有设备都连接到主服务器，那么会有性能问题。如果每个地区建立一个独立的服务器，然后主服务器先发消息给所有的broker，这样就能平分连接。提高性能。

FAILOVER PROTOCOL故障转移协议
如果不能确定连接到什么地方，或者是自己当前的连接断了，需要实现固定的故障转移的时候，用这个。
有两种方式提供一个供选择的列表， 一种就是上面说的静态的，还有一种是动态的，后面说
通过这种方式进行配置：failover:(uri1,...,uriN)?key=value
通过刚才的静态协议配置，使得会转发给两个其他的broker，然后通过故障转移协议来进行消费。并且尝试挂掉其中的一个，来观看效果
经过尝试，随便来两个也是可以这样的，并且还经过尝试，静态配置的那个从主broker上也是可以收取到消息的(可以确定是消息被复制了两份)。
生产者貌似不行，这个还得再测试

### 动态 network
MULTICAST CONNECTOR
语法：multicast://ipadaddress:port?key=value
需要配置activemq，是的他能够被别人发现。 

利用发现协议则不需要频繁的修改配置
缺点是需要如果不想要被发现需要注意网络配置。

DISCOVERY PROTOCOL
客户端使用，有点类似于故障转移。
discovery:(discoveryAgentURI)?key=value
步骤上是需要先配置networkconnection使其能够被发现，然后client使用发现协议来生产消费信息
成功

PEER PROTOCOL对等协议
为了方便配置
peer://peergroup/brokerName?key=value

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-28.png)

消息会同步给所有加入这个组的broker.
如果生产消费都用这个uri来启动，那么会启动两个对等的嵌入式broker.这两个broker之间通过远程协议通讯。
使用场景：比如想要在本地也运行，在有网络的情况下也运行。
FANOUT CONNECTOR 扇出连接器
客户端同时连接多个broker，且同事复制操作到这多个broker中的情况。
fanout:(fanoutURI)?key=value
fanoutURI可以使用组播或者static的uri.
fanout:(static:(tcp://host1:61616,tcp://host2:61616,tcp://host3:61616))
fanout:(multicast://default)
必须两个以上，只能是publish使用，不能是consumer.

![](http://git.oschina.net/wzj777/princeWiki/raw/master/pic/mq/mq-29.png)

Table 4.2 Summary of protocols used to network brokers

| protocol        | 描述  |
| ------------- | -----:|
| static      | 用来定义已知网络地址的配置 |
| failover  |   当从一个broker断开的时候，重连 |
| multicast |  定义动态的brokers   |
| discovery | client 动态的去找brokers   |
| peer | 更简单的连组播嵌入式brokers |
| fanout | 用来同时向多个brokers发消息。 |